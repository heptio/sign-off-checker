# sign-off-checker

This is a simple Go server that listens for web hooks from GitHub for PRs. It then looks at each commit in that PR and sets a status.  If all of the commits have a "Signed-off-by" line on them then it marks all of those commits as "success".  If any one of them is missing the "Signed-off-by" line then all are marked as "failed".

The status check points to a "CONTRIBUTING.md" file in the repo in question.

## Building

You can just `go get github.com/heptiolabs/sign-off-checker/cmd/sign-off-checker` to get the binary installed locally.  To build a docker container do `make push REGISTRY=<my-gcr-registry>` from this repo.

## Running
There are two environment variables that need to be set when running:

* `SHARED_SECRET`: Set this to a random value that you supply as the "secret" when configuring the webhook.
* `GITHUB_TOKEN`: Set this to a personal access token for a github user that has access to the repo in question.  The webhook doesn't include details of the commits so we have to fetch them.  Unfortunately this requires full read/write `repo` access scope (even if we're not using automatic registration and are just reading).  Create one of these at https://github.com/settings/tokens.

Run the server someplace.  It'll listen at `http://<example.com>/webhook`.  Now head on over to the settings tab of your repo and add a webhook.  The Payload URL should be set to the URL. The content type should be `application/json` and the secret should be the secret above.  Select "individual events" and check "Pull request".  If things are working you can check the status of the webhook from GitHub's point of view on that page.

## Automatic Registration
The server also supports automatically registering itself for all DCO repositories in a list of GitHub organizations.
It will find all repositories that use Developer Certificate of Origin (in "CONTRIBUTING.md") and configure:

 1. A webhook that receives all `pull_request` events.
 1. Branch protection for the default branch.

To enable this mode, pass two extra environment variables:

* `AUTOREGISTER_ORGANIZATIONS`: Set this to a comma-separated list of organizations you would like to automatically register.
* `PUBLIC_WEBHOOK_URL`: Set this to the public URL of this server (i.e., your internet-facing HTTPS URL)

You can optionally pass `DRY_RUN=1` to enable dry run mode (no configuration will be changed).

### Build stuff
Taken from https://github.com/thockin/go-build-template