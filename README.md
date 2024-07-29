# Plugin Release Deploy Action

This GitHub Action helps you release and deploy WordPress plugins using PHP and Node. It automates the entire process, from creating a release to deploying the plugin on WordPress.org.

## Description

This action does the following:
1. Creates a new release and generates a changelog.
2. Checks out your repository code.
3. Sets up the specified PHP and Node.js environments.
4. Installs the required PHP and Node.js dependencies.
5. Builds the plugin.
6. Uploads the built plugin as a zip file to the GitHub release.
7. Deploys the plugin to the WordPress.org SVN repository.

## Inputs

These are the settings you can configure for this action:

| Input Name     | Description                                                   | Required | Default |
|----------------|---------------------------------------------------------------|----------|---------|
| `php-version`  | The PHP version to set up                                     | Yes      | `7.4`   |
| `node-version` | The Node.js version to set up                                 | Yes      | `16`    |
| `plugin-slug`  | The slug (unique identifier) for your plugin on WordPress.org | Yes      |         |
| `build-dir`    | The directory where your plugin is built                      | Yes      |         |
| `zip-file`     | The name of the zip file to upload to the release             | Yes      |         |
| `svn-username` | Your WordPress.org SVN username                               | Yes      |         |
| `svn-password` | Your WordPress.org SVN password                               | Yes      |         |
| `dry-run`      | If set to `true`, performs a dry run (test) of the deployment | No       | `false` |

## Usage

To use this action, you need to add it to your GitHub Actions workflow file. Here’s a sample workflow file:

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
      - run: echo "The PR was merged"
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

In this example:
- Replace `'your-plugin-slug'` with the slug of your WordPress plugin.
- Replace `'./build/trunk/'` with the directory where your plugin is built.
- Replace `'your-plugin.zip'` with the name of the zip file to upload.
- Add your WordPress.org SVN username and password to your repository secrets as `WP_SVN_USERNAME` and `WP_SVN_PASSWORD`.

## Example

Here’s a complete example for a plugin named `my-wordpress-plugin`:

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
      - run: echo "The PR was merged"
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
