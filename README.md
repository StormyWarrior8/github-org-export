# github-org-export
A script to organize repositories in an Org by size and request exports from the migration API. This script is NOT a replacement for Professional Services engagement.

## Usage
This script requires two environment variables:
 - `GITHUB_TOKEN`
   
   A token generated by an administrator account for the organization
 - `GITHUB_ORG`
   
   The name of the Github Org. e.g. somecorp

Additionally, there are adjustable variables at the beginning of the script. Of note, there are:
 - `EXPORT_SIZE`
   
   This needs to be kept 'under 5G'. Keeping this at or under 2G was good for my runs. Repo sizes vary greatly and this was a lazy (best?) effort to keep the export size in check.
 - `LOCK_REPOSITORIES`
   
   This is false out of the box because you're doing dry runs first right? RIGHT?

With these variables reviewed, running the script is as easy as `python export_github_org.py`. Here's a description of what happens when the script runs:

 1. Request a list of all repositories. This is paginated as indicated by the response headers.
 2. Request each page of repositories, and write each page to the directory `repo_out` as page #.json
 3. Request a list of all users. This is also paginated, so we do the same thing as the repo list, and write those pages to `user_out`
 4. We extract the individual user names and drop them in a file alongside the script called `user_names.txt`. This isn't specifically needed to export, but may be good to have later. Particularly because you might be creating a map file to connect Github.com users to AD users or some other authentication source.
 5. Groupings of repositories are created by going through a list of repo names and sizes. The max number of repos to be requested is 100. The maximum size of an export to be requested is roughly 2GB. This isn't fool proof. If you have very large repositories, consider commenting out `create_migration_bundles()` at the end of the script, running it once, then remove those repositories from repo_names.txt. You will need to request exports for those on their own, but you can go back and remove the comment to process larger quantities of repositories. We had over 1,000 repositories to migrate.
 6. Each migration export request gets a reply with a GUID, a migration URL, and an archive URL. In `mig_out`, there are directories named after the GUID from each export bundle. There's also a guid file with the same string in that directory along with a list of the repos that were requested and the migration url. The migration url will be needed in the event that you need to unlock repositories later on Github.com.
 7. At this point, we poll the migration url to check the stats every 30 seconds. Once the status comes back `exported`, we download the migration archive to the directory for that export (named after guid).

When the script completes, you should have everything you need to begin importing. When you're comfortable and satisfied with your results, schedule your organization's outage, flip `LOCK_REPOSITORIES` to true, and run for production. With `LOCK_REPOSITORIES` set to true, all repos in the organization will be unavailable to pushes, pulls, clones, and viewing on Github.com.

## License
 See [LICENSE.txt](LICENSE.txt) for details.