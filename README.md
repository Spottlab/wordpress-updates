# Spottlab WordPress updates

A reusable GitHub Actions workflow that turns [Spottlab](https://spottlab.com)'s update alerts into pull requests. When Spottlab finds a WordPress or plugin update, or a vulnerability with a fixed version out, it starts your repository's `spottlab.yml` workflow. This workflow then updates WordPress and its plugins on `spottlab/updates/<branch>` and opens a pull request into `<branch>`. If that pull request is already open, it refreshes it. Your checks and your usual deploy take it from there.

## Setup

1. Add `.github/workflows/spottlab.yml` to your repository's default branch:

   ```yaml
   name: Spottlab updates
   on: workflow_dispatch
   jobs:
     updates:
       uses: spottlab/wordpress-updates/.github/workflows/updates.yml@1.0.0
       permissions:
         contents: write
         pull-requests: write
   ```

2. In the repository, open Settings → Actions → General, and under Workflow permissions tick **Allow GitHub Actions to create and approve pull requests**.
3. Make a fine-grained personal access token with access to only that repository and a single permission, **Actions: Read and write**. Paste it into the GitHub action on Spottlab's Workflows page, with the repository and the branch to update. The settings there link to GitHub's token page with everything filled in except the repository.
4. Run a test from there. It also checks that the token cannot change your code.

## What it updates

It works out how the site keeps WordPress:

- **With Composer** (Bedrock, `roots/wordpress`, WPackagist or WP Packages): it updates `composer.json` and `composer.lock` to the newest versions. A pinned version stays pinned, and a caret stays a caret. Other constraints are updated as far as they allow. For private repositories, add a `COMPOSER_AUTH` secret, and `secrets: inherit` under `uses`.
- **Committed as it is:** it updates core, the plugins from WordPress.org and their translations with wp-cli. This happens in a WordPress and a database of the job's own, so your site's database is never touched. Each updated plugin is committed exactly as released.

Plugins from anywhere else, such as ACF Pro, are left as they are. A new WordPress version can change the database, so have your deploy run `wp core update-db` after it goes out.

## Inputs

The job runs on PHP 8.4, and adds Composer and what wp-cli needs. For another version:

```yaml
    uses: spottlab/wordpress-updates/.github/workflows/updates.yml@1.0.0
    with:
      php: '8.3'
```

GitHub does not run other workflows for a pull request that a workflow opened, so your checks start when you push to it or reopen it.

## If something goes wrong

Spottlab passes nothing to the workflow, and its token can start workflows and nothing more. Wherever the token ends up, it cannot read or change your code or see your secrets. It can start, rerun or cancel workflows. That includes any other workflow of yours that runs on `workflow_dispatch`, with inputs of its choosing, so keep inputs in `env` and out of `run` lines. This workflow is pinned to the version in your `uses`, and it gets updates only from WordPress.org and Composer. Nothing reaches your site until someone merges the pull request. To stop it at once, delete the token on GitHub.
