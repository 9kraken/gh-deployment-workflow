# gh-deployment-workflow

This project demonstrates how to automatically deploy a static website to GitHub Pages using GitHub Actions.

The workflow runs on every push to the `main` branch, builds the site, and publishes it to the `gh-pages` branch.

https://roadmap.sh/projects/github-actions-deployment-workflow

## Project Structure

```text
/
|-- index.html
|-- README.md
+-- .github
    +-- workflows
        +-- deploy.yml
```

## How It Works

1. Changes of index.html are pushed to the `main` branch.
2. GitHub Actions triggers the workflow.
3. The workflow prepares the build directory.
4. The `peaceiris/actions-gh-pages` action deploys the files to the `gh-pages` branch using SSH authentication.
5. GitHub Pages serves the site from the `gh-pages` branch.

## Why SSH Authentication Is Needed

GitHub Actions runs in a temporary virtual machine.
When the workflow deploys the site, it needs permission to push to the repository.

In our workflow the deployment action performs something similar to:

`git push origin gh-pages`

However, the workflow environment is not logged into GitHub, so it *must authenticate.*

There are several ways to authenticate:

| Method | Scope | Security Benefit | Best For |
| :--- | :--- | :--- | :--- |
| **GITHUB_TOKEN** | Single repository <br>(where workflow runs) | Auto-generated & <br>revoked per job | Standart tasks: CI/CD, <br>releases and PR comments <br>within the same repo |
| **GitHub App** | Fine-grained <br>(selected repos) | Short-lived; <br>Not tied to user | Advanced automation. <br>Cross-repo auth |
| **PAT** | User-level <br>(all repos that <br>user can see) | Simple setup for <br>personal scripts or <br>third party uses| Quick fixes or scripts <br>that need access to your <br>entire account's data. |
| **SSH Key** | Single repository | No web-tokens; <br>Git only | Simple repo cloning <br>or pushing to external <br>repository |

In this project we use *SSH deploy keys.*

## What an SSH Deploy Key Is

An SSH deploy key is a **cryptographic key pair** used for authentication.

It consists of two files:

Private key (secret):

    deploy_key

Public key:

    deploy_key.pub

The idea is simple:

- the **public key** is added to GitHub
- the **private key** is stored securely and used by the workflow

When the workflow connects to GitHub via SSH, GitHub verifies that the private key matches the stored public key.

## Generating the SSH Keys

Keys were generated locally using:

    ssh-keygen -t ed25519 -C "deploy@github" -f .github/deploy_key -N ""

Explanation:

- `-t ed25519` - modern and secure SSH key algorithm
- `-C` - comment added to the key
- `-f` - output file location
- `-N ""` - empty passphrase (required for automation)

This command creates:

    .github/deploy_key
    .github/deploy_key.pub

## Adding the Key to GitHub

The **public key** was added to the repository:

    Settings → Deploy keys → Add deploy key

Options:

- Title: `actions-deploy-key`
- Key: contents of `deploy_key.pub`
- Allow write access: enabled

Write access is required because the workflow pushes to the `gh-pages` branch.

## Storing the Private Key Securely

The **private key must never be committed to the repository**.

Instead, it is stored in GitHub Secrets:

    Settings → Secrets and variables → Actions → New repository secret

Secret name:

    ACTIONS_DEPLOY_KEY

Value:

contents of the `deploy_key` file.

During workflow execution the key is accessed via:

    ${{ secrets.ACTIONS_DEPLOY_KEY }}

## Deployment Action

The deployment is performed using:

    peaceiris/actions-gh-pages

This action:

1. receives the private SSH key
2. authenticates with GitHub
3. pushes the build directory to the `gh-pages` branch

Example configuration:

    - name: Deploy to GitHub Pages
      uses: peaceiris/actions-gh-pages@v4
      with:
        publish_dir: ./build
        publish_branch: gh-pages
        ssh_private_key: ${{ secrets.ACTIONS_DEPLOY_KEY }}

## Result

Every time code is pushed to `main`:

1. GitHub Actions runs the workflow
2. The site is built
3. The `gh-pages` branch is updated
4. GitHub Pages publishes the website automatically
