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
| `build-dir` | Directory containing unpacked plugin files for WordPress.org SVN deployment (e.g., `./build/trunk/` or `./build/my-plugin/`) | — |
| `zip-file` | Name/path of the zip file to upload to GitHub Release | — |

### Optional Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `node-version` | Node.js version to set up (leave empty to skip Node.js) | `''` |
| `use-zipit` | Use `bin/zipit` to build the plugin package | `'true'` |
| `zipit-config` | Path to ZipIt configuration file | `.zipit-conf.php` |
| `deploy-to-wporg` | Deploy the plugin to WordPress.org SVN repository | `'false'` |
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
          build-dir: './build/your-plugin-slug/'  # Can be any directory name
          zip-file: 'your-plugin.zip'
          svn-username: ${{ secrets.WP_SVN_USERNAME }}
          svn-password: ${{ secrets.WP_SVN_PASSWORD }}
          deploy-to-wporg: 'true'  # Enable WordPress.org deployment
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
          deploy-to-wporg: 'true'  # Enable WordPress.org deployment
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
          # deploy-to-wporg defaults to 'false' - no WordPress.org deployment
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
          deploy-to-wporg: 'true'  # Enable WordPress.org deployment
```

---

## Understanding `build-dir` vs `zip-file`

These two inputs serve different purposes in the release workflow:

### `build-dir` – Source for WordPress.org SVN Deployment

The `build-dir` specifies where your **unpacked plugin files** live on disk. This can be any directory path (e.g., `./build/trunk/`, `./build/my-plugin/`, or `./dist/`). This directory is passed to the [10up WordPress Plugin Deploy action](https://github.com/10up/action-wordpress-plugin-deploy), which:

1. Takes the contents of `build-dir`
2. Copies them to the WordPress.org SVN working copy's `trunk/` directory
3. Commits and tags the release on WordPress.org

**Example structure:**
```
./build/my-plugin/          # Your build-dir can be named anything
├── my-plugin.php
├── readme.txt
├── includes/
└── assets/
```

When you set `build-dir: './build/my-plugin/'`, the action deploys **the contents of this directory** to WordPress.org's SVN `trunk/`.

### `zip-file` – Package for GitHub Release

The `zip-file` is the **distribution package** uploaded to your GitHub Release. This is typically created by ZipIt (or your custom build process) and represents the downloadable plugin archive.

**Example:** `my-plugin.zip`

### Key Differences

| Aspect | `build-dir` | `zip-file` |
|--------|-------------|------------|
| **Format** | Directory (unpacked files) | Zip archive (packaged file) |
| **Used By** | 10up WordPress.org deploy action | GitHub Release upload |
| **Required When** | Always (required input) | Always (required input) |
| **Used When** | `deploy-to-wporg: 'true'` | Every release |
| **Purpose** | SVN deployment source | Distribution download |

### Typical Workflow

1. Your build process (Composer + optionally Node.js) creates final plugin files in a directory like `./build/my-plugin/` or `./build/trunk/`
2. ZipIt packages those files into `my-plugin.zip` (based on `.zipit-conf.php`)
3. The action:
   - Uploads `my-plugin.zip` to GitHub Release (using `zip-file`)
   - Deploys contents of your build directory to WordPress.org (using `build-dir`)

**Note:** The `build-dir` name doesn't have to be "trunk"—you can use any directory structure that works for your project (e.g., `./build/my-plugin/`, `./dist/`, `./release/`).

### When Not Deploying to WordPress.org

By default, `deploy-to-wporg` is `'false'`, which means WordPress.org deployment is disabled unless you explicitly enable it. When WordPress.org deployment is disabled, the `build-dir` input becomes unused (the 10up action never runs). However, it remains a required input for consistency across different deployment scenarios.

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
The `build-dir` contains your unpacked plugin files ready for WordPress.org SVN deployment. When `deploy-to-wporg` is `'true'`, the 10up action syncs this directory to the WordPress.org repository's `trunk/` (and creates tags from it).

**Zip File**  
The `zip-file` is your packaged plugin archive uploaded to GitHub Releases. This file is automatically created by ZipIt when `use-zipit` is `'true'`, or by your custom build process when `use-zipit` is `'false'`. The zip file and build directory can (but don't have to) contain the same files—they serve different distribution channels.

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
