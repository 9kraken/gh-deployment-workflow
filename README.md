# gh-deployment-workflow

This project demonstrates how to automatically deploy a static website to GitHub Pages using GitHub Actions.

The workflow runs on every push to the `main` branch, builds the site, and publishes it to the `gh-pages` branch.

https://roadmap.sh/projects/github-actions-deployment-workflow

## How It Works

1. Changes of index.html are pushed to the `main` branch.
2. GitHub Actions triggers the workflow.
3. The workflow prepares the build directory.
4. The `peaceiris/actions-gh-pages` action deploys the files to the `gh-pages` branch using SSH authentication.
5. GitHub Pages serves the site from the `gh-pages` branch.
