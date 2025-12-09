# Plugin Release Deploy Action

A GitHub Action that automates WordPress plugin releases and deployments using PHP and optionally Node.js. It streamlines the entire process from creating releases to building distribution packages and deploying to WordPress.org.

## Overview

This action orchestrates your complete plugin release workflow:

1. **Release Management** – Uses [`release-please`](https://github.com/google-github-actions/release-please-action) to create GitHub releases with automated changelogs
2. **Environment Setup** – Configures PHP and optionally Node.js environments
3. **Dependency Management** – Installs and builds PHP dependencies via Composer
4. **Asset Building** – Optionally builds frontend assets with Node.js
5. **Package Creation** – Builds plugin zip files using [ZipIt](https://github.com/devuri/zipit) or custom methods
6. **Release Distribution** – Uploads the plugin package to GitHub releases
7. **WordPress.org Deployment** – Optionally deploys to the WordPress.org plugin repository via SVN

---

## Inputs

### Required Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `github-token` | GitHub authentication token (typically `${{ secrets.GITHUB_TOKEN }}`) | — |
| `php-version` | PHP version for your environment | `8.1` |
| `build-dir` | Directory containing built plugin files (e.g., `./build/trunk/`) | — |
| `zip-file` | Name/path of the zip file to create and upload | — |

### Optional Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `node-version` | Node.js version to set up (leave empty to skip Node.js) | `''` |
| `use-zipit` | Use `bin/zipit` to build the plugin package | `'true'` |
| `zipit-config` | Path to ZipIt configuration file | `.zipit-conf.php` |
| `deploy-to-wporg` | Deploy the plugin to WordPress.org SVN repository | `'true'` |
| `dry-run` | Test WordPress.org deployment without committing changes | `'false'` |

### WordPress.org Deployment Inputs

Required only when `deploy-to-wporg` is `'true'`:

| Input | Description |
|-------|-------------|
| `plugin-slug` | WordPress.org plugin slug (unique identifier) |
| `svn-username` | WordPress.org SVN username |
| `svn-password` | WordPress.org SVN password |

---

## Usage Examples

### Basic PHP Plugin with ZipIt

Standard WordPress.org plugin using PHP only:

```yaml
name: 🚀 Deploy Release

on:
  pull_request:
    types: [closed]
  workflow_dispatch:

jobs:
  release:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    
    steps:
      - name: Deploy Release
        uses: devuri/plugin-release-deploy-action@main
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          php-version: '8.1'
          plugin-slug: 'your-plugin-slug'
          build-dir: './build/trunk/'
          zip-file: 'your-plugin.zip'
          svn-username: ${{ secrets.WP_SVN_USERNAME }}
          svn-password: ${{ secrets.WP_SVN_PASSWORD }}
```

### PHP + Node.js Plugin

Plugin with frontend assets built via Node.js:

```yaml
name: 🚀 Deploy Release

on:
  pull_request:
    types: [closed]
  workflow_dispatch:

jobs:
  release:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    
    steps:
      - name: Deploy Release
        uses: devuri/plugin-release-deploy-action@main
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          php-version: '8.1'
          node-version: '20'
          plugin-slug: 'my-wordpress-plugin'
          build-dir: './build/trunk/'
          zip-file: 'my-wordpress-plugin.zip'
          svn-username: ${{ secrets.WP_SVN_USERNAME }}
          svn-password: ${{ secrets.WP_SVN_PASSWORD }}
```

### GitHub-Only Release (No WordPress.org)

Create releases on GitHub without deploying to WordPress.org:

```yaml
name: 🚀 Deploy Release

on:
  pull_request:
    types: [closed]
  workflow_dispatch:

jobs:
  release:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    
    steps:
      - name: Deploy Release
        uses: devuri/plugin-release-deploy-action@main
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          php-version: '8.1'
          build-dir: './build/'
          zip-file: 'my-plugin.zip'
          deploy-to-wporg: 'false'
```

### Custom Zip Building (Without ZipIt)

Use your own build process instead of ZipIt:

```yaml
name: 🚀 Deploy Release

on:
  pull_request:
    types: [closed]
  workflow_dispatch:

jobs:
  release:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4
      
      - name: Build Plugin Zip
        run: ./bin/build-zip.sh
      
      - name: Deploy Release
        uses: devuri/plugin-release-deploy-action@main
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          php-version: '8.1'
          plugin-slug: 'legacy-plugin'
          build-dir: './build/trunk/'
          zip-file: 'legacy-plugin.zip'
          svn-username: ${{ secrets.WP_SVN_USERNAME }}
          svn-password: ${{ secrets.WP_SVN_PASSWORD }}
          use-zipit: 'false'
```

---

## Requirements

### Repository Configuration

**Release Please**  
Your repository must be configured for [`release-please`](https://github.com/google-github-actions/release-please-action) with:
- `release-please-config.json`
- `release-please-manifest.json`

This action runs the release-please `manifest` command but does not configure it.

**Composer**  
Your plugin must use Composer with a `build` script defined in `composer.json`:

```json
{
  "scripts": {
    "build": "php ./bin/build-plugin.php"
  }
}
```

**Node.js** (Optional)  
If using Node.js, ensure you have:
- A `package.json` file
- Build scripts (e.g., `npm run build`)

### ZipIt Configuration

When `use-zipit` is `'true'`:

- The `bin/zipit` executable must exist and be executable
- A `.zipit-conf.php` file must be present (or at the path specified by `zipit-config`)
- The `outputFile` in `.zipit-conf.php` must match the `zip-file` input
- PHP 8.1+ is required (default `php-version` is `8.1`)

### Build Output

**Build Directory**  
The `build-dir` must contain your final plugin files ready for deployment. This typically corresponds to the `trunk/` directory in WordPress.org's SVN structure.

**Zip File**  
The `zip-file` must exist before the upload step. This is automatically created by ZipIt when `use-zipit` is `'true'`, or by your custom build process when `use-zipit` is `'false'`.

### WordPress.org Deployment

When deploying to WordPress.org:

- Store SVN credentials in repository secrets (`WP_SVN_USERNAME`, `WP_SVN_PASSWORD`)
- Ensure version tags from release-please match your plugin's `Version` header
- Verify your plugin slug matches the WordPress.org repository name

---

## How It Works

This action combines several tools into a streamlined workflow:

- **[release-please](https://github.com/google-github-actions/release-please-action)** – Automated release creation and changelog generation
- **[10up WordPress Plugin Deploy](https://github.com/10up/action-wordpress-plugin-deploy)** – WordPress.org SVN deployment
- **[ZipIt](https://github.com/devuri/zipit)** – PHP-based plugin packaging tool

While opinionated, this approach provides a complete solution for most WordPress plugin release workflows. For more flexibility, consider using these tools individually.

---

## Additional Resources

**Example Plugin**  
See the [Sample Plugin](https://github.com/devuri/shortcode-options) for a complete working example of plugin structure and configuration.

**Secrets Setup**  
Add these secrets to your repository (Settings → Secrets and variables → Actions):
- `WP_SVN_USERNAME` – Your WordPress.org username
- `WP_SVN_PASSWORD` – Your WordPress.org password
- `GITHUB_TOKEN` – Automatically provided by GitHub Actions

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
