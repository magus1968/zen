---
icon: lucide/cloud-upload
tags:
  - Get started
  - Publishing
---

# Publish your site

The great thing about hosting project documentation in a `git` repository is
the ability to deploy it automatically when new changes are pushed. Zensical
makes this ridiculously simple.

## GitHub Pages

If you're already hosting your code on GitHub, [GitHub Pages] is certainly
the most convenient way to publish your project documentation. It's free of
charge and pretty easy to set up.

  [GitHub Pages]: https://pages.github.com/

### with GitHub Actions

Using [GitHub Actions] you can automate the deployment of your project
documentation. At the root of your repository, create a new GitHub Actions
workflow, e.g. `.github/workflows/docs.yml`, and copy and paste the following
contents:

``` yaml
name: Documentation
on:
  push:
    branches:
      - master
      - main
permissions:
  contents: read
  pages: write
  id-token: write
jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/configure-pages@v5
      - uses: actions/checkout@v5
      - uses: actions/setup-python@v5
        with:
          python-version: 3.x
      - run: pip install zensical
      - run: zensical build --clean # (1)!
      - uses: actions/upload-pages-artifact@v4
        with:
          path: site
      - uses: actions/deploy-pages@v4
        id: deployment
```

1.  At the moment, we do not recommend using caches on CI systems as the caching
    functionality will undergo revisions as we optimize the performance of
    Zensical.

When a new commit is pushed to the branch you are using for deployment
(e.g. `master` or `main`), the static site is automatically built and
deployed. Push your changes to see the workflow in action.

Your documentation should shortly appear at `<username>.github.io/<repository>`.

  [GitHub Actions]: https://github.com/features/actions

## GitLab Pages

If you're hosting your code on GitLab, deploying to [GitLab Pages] can be done
by using the [GitLab CI] task runner. At the root of your repository, create a
task definition named `.gitlab-ci.yml` and copy and paste the following
contents:

``` yaml
pages:
  stage: deploy
  image: python:latest
  script:
    - pip install zensical
    - zensical build --clean # (1)!
  pages:
    publish: public
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
```

1.  At the moment, we do not recommend using caches on CI systems as the caching
    functionality will undergo revisions as we optimize the performance of
    Zensical.

!!! warning "Prerequisites"
    Make sure to change [`site_dir`][site_dir] to `public` in your
    configuration as GitLab requires this.

When a new commit is pushed to the [default branch] (e.g. `master` or `main`),
the static site is automatically built and deployed. Push your changes to see
the workflow in action.

Your documentation is now published under `<username>.gitlab.io/<repository>`.

  [GitLab Pages]: https://gitlab.com/pages
  [GitLab CI]: https://docs.gitlab.com/ee/ci/
  [site_dir]: setup/basics.md#site_dir
  [default branch]: https://docs.gitlab.com/ee/user/project/repository/branches/default.html

## Azure Static Web App

You can deploy your Zensical site to [Azure Static Web Apps (SWA)](https://azure.microsoft.com/products/app-service/static) using GitHub Actions (or other methods). Azure SWA supports advanced features such as custom domains and authentication (including Microsoft Entra ID) which are not covered here.

These are the steps to create an Azure SWA and configure a GitHub hosted site to automatically build and deploy a basic, public site:

1. In your site repository, disable the default, automatic GitHub Pages workflow in `.github/workflows/docs.yml` - you can do this by deleting the file or changing the trigger to workflow_despatch.
   ```yaml
    on:
      workflow_dispatch:
   ```
2. In the [Azure Portal](https://portal.azure.com/), create a new **Static Web App** resource:
    - **Subscription:** Select your Azure subscription
    - **Resource Group:** Create or select an existing group
    - **Name:** Choose a unique name for your SWA
    - **Hosting Plan:** Free or Standard (free is fine for a basic, public site)
    - **Source:** GitHub
    - Click **Sign in with GitHub**
        - Click **Authorize AzureAppService**
        - Authenticate with your GitHub account
    - **Organization/Repository/Branch:** Select your organization, repository and branch (e.g. `main`)
3. Click **Review + create** and wait for validation
4. Click **Create** to deploy your application, wait for this to complete
5. Click **Go to resource** to open the application
6. Make a note of the default URL `https://UNIQUE_NAME.azurestaticapps.net`
7. Ignore the initial GitHub Action deployment error which is caused by missing settings in the workflow
8. Open your GitHub repository (remember to pull the changes made by the Azure setup) and find the new Azure workflow YAML in `.github/workflows/` - it will be named something like `azure-static-web-apps-UNIQUE_NAME.yml`
9. You need to update this workflow file in two places to correctly build and deploy your Zensical site. Firstly, find the `actions/checkout` step and paste the following steps between it and the following `Azure/static-web-apps-deploy` step:

   ```yaml
   - uses: actions/setup-python@v6
     with:
       python-version: 3.x
   - run: pip install zensical
   - run: zensical build --clean
   ```
10. Secondly, in the `Azure/static-web-apps-deploy` step, look for the `app_location` setting and modify it and the surrounding block to look like this:

   ```yaml
          app_location: "site" # App source code path
          api_location: "" # Api source code path - optional          
          skip_app_build: true # not required since zensical is already building the app
          skip_api_build: true # not required since there is no API
          output_location: "" # Built app content directory - optional
          ###### End of Repository/Build Configurations ######
   ```
11. Commit and push your changes, monitor the GitHub Action to make sure the site deploys successfully

You can now validate your site has been published at `https://UNIQUE_NAME.azurestaticapps.net`. On each subsequent push to the configured branch, GitHub Actions will rebuild your site and deploy it to Azure SWA.

For more details, see [Azure Static Web Apps documentation](https://learn.microsoft.com/azure/static-web-apps/overview).