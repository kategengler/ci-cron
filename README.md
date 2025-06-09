# CI Cron GitHub Action

This action allows running a particular workflow on a branch of choice.

It accomplishes this by creating a new branch from the specified branch, making
a commit, and then pushing it to the remote repository. It then waits for the 
completion of all check suites from 'github-actions' on that commit. 

The branches created are deleted after the workflow completes. To rerun, trigger
the Cron workflow manually.

For example, Ember.js uses this action to run the main CI workflow on the
`main`, `beta` and `release` branches nightly. This allows us to ensure that the
code on these branches is always in a good state, even if no one has pushed to them recently.

## Usage

For the workflow(s) you wish to run the cron on, you will need to add `- cron*`
to the list of branches that trigger the workflow. This prefix is configurable if you wish
to have more than one cron workflow. 

An example workflow is below: 

```yaml
name: Cron

on:
  schedule:
    - cron:  '0 7 * * *' # daily, 7am -- set this to how often you want to run your cron
  workflow_dispatch:     # This allows triggering the workflow manually

permissions:
  contents: read

jobs:
  trigger-ci:
    permissions:
      contents: read # the push uses a personal-access token so it will trigger workflows, so this permission is read-only
      checks: read

    name: Trigger cron build
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        branch: [main, beta, release] # List of branches to run the cron on
    steps:
      - uses: kategengler/ci-cron@v1
        with:
          branch: ${{ matrix.branch }}
          # This must use a personal access token because of a Github Actions
          # limitation where it will not trigger workflows from pushes from
          # other workflows with the token it provides.
          # The PERSONAL_ACCESS secret must be a token with `repo` scope.
          # See https://help.github.com/en/actions/reference/events-that-trigger-workflows#triggering-new-workflows-using-a-personal-access-token
          personal_token: ${{ secrets.PERSONAL_TOKEN }}
          committer_email: 'cron@example.com'
          committer_name: 'Ember.js Cron CI'
```
