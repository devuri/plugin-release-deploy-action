# Plugin Release Deploy Action

This GitHub Action facilitates the release and deployment of WordPress plugins using PHP and Node. It combines the necessary steps to automate the process from release creation to deployment on WordPress.org.

## Description

This composite action performs the following tasks:
1. Runs `release-please` to manage release tagging and changelog generation.
2. Checks out the repository code.
3. Sets up the specified PHP and Node.js environments.
4. Installs PHP and Node.js dependencies.
5. Builds the project artifact.
6. Uploads the built artifact as a zip file to the GitHub release.
7. Deploys the WordPress plugin to the WordPress.org SVN repository.

## Inputs

| Input Name     | Description                                                   | Required | Default |
|----------------|---------------------------------------------------------------|----------|---------|
| `php-version`  | PHP version to setup                                          | true     | `7.4`   |
| `node-version` | Node version to setup                                         | true     | `16`    |
| `plugin-slug`  | Slug for the WordPress plugin on WordPress.org                | true     |         |
| `build-dir`    | Build directory for the WordPress plugin                      | true     |         |
| `zip-file`     | Zip file to upload to release                                 | true     |         |
| `svn-username` | SVN username for WordPress.org plugin deployment              | true     |         |
| `svn-password` | SVN password for WordPress.org plugin deployment              | true     |         |
| `dry-run`      | Flag to perform a dry run of the deployment                   | false    | `false` |

## Usage

To use this action in your repository, reference it in your workflow file as shown below:

```yaml
name: 🚀 Deploy Release
on:
  pull_request:
    types:
      - closed
  workflow_dispatch:

jobs:
  if_merged:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    steps:
      - run: echo The PR was merged
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy Release
        uses: devuri/plugin-release-deploy-action@main
        with:
          php-version: '7.4'
          node-version: '16'
          plugin-slug: 'your-plugin-slug' # WordPress.org plugin slug
          build-dir: './build/trunk/'
          zip-file: 'your-plugin.zip'
          svn-username: ${{ secrets.WP_SVN_USERNAME }}
          svn-password: ${{ secrets.WP_SVN_PASSWORD }}
          dry-run: 'false' # This is optional and defaults to 'false'
```


## Example

Here’s an example configuration for a plugin named `my-wordpress-plugin`:

```yaml
name: 🚀 Deploy Release
on:
  pull_request:
    types:
      - closed
  workflow_dispatch:

jobs:
  if_merged:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    steps:
      - run: echo The PR was merged
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy Release
        uses: devuri/plugin-release-deploy-action@main
        with:
          php-version: '7.4'
          node-version: '16'
          plugin-slug: 'my-wordpress-plugin' # WordPress.org plugin slug
          build-dir: './build/trunk/'
          zip-file: 'my-wordpress-plugin.zip'
          svn-username: ${{ secrets.WP_SVN_USERNAME }}
          svn-password: ${{ secrets.WP_SVN_PASSWORD }}
          dry-run: 'false'
```

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
