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
| `zip-file` | File(s) to upload to GitHub Release. Supports glob patterns (e.g., `*.zip`, `my-plugin.zip;README.md`, `build/*.zip`). Paths are relative to repository root. | — |

### Optional Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `node-version` | Node.js version to set up (leave empty to skip Node.js) | `''` |
| `use-zipit` | Use `bin/zipit` to build the plugin package | `'true'` |
| `zipit-config` | Path to ZipIt configuration file | `.zipit-conf.php` |
| `deploy-to-wporg` | Deploy the plugin to WordPress.org SVN repository | `'false'` |
| `dry-run` | Test WordPress.org deployment without committing changes | `'false'` |
| `assets-dir` | Directory containing WordPress.org assets (banners, icons, screenshots) | `'.wordpress-org'` |

### WordPress.org Deployment Inputs

Required only when `deploy-to-wporg` is `'true'`:

| Input | Description |
|-------|-------------|
| `plugin-slug` | WordPress.org plugin slug (unique identifier) |
| `svn-username` | WordPress.org SVN username |
| `svn-password` | WordPress.org SVN password |

---

## Outputs

The action exposes the following outputs for use in downstream workflow steps:

| Output | Description |
|--------|-------------|
| `release-created` | `'true'` if a release was created by release-please, `'false'` otherwise |
| `tag-name` | The release tag name (e.g., `v1.2.3`), only set if a release was created |

### Using Outputs

You can reference these outputs in subsequent jobs or steps:

```yaml
jobs:
  release:
    runs-on: ubuntu-latest
    outputs:
      released: ${{ steps.deploy.outputs.release-created }}
      version: ${{ steps.deploy.outputs.tag-name }}
    steps:
      - name: Deploy Release
        id: deploy
        uses: devuri/plugin-release-deploy-action@main
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          # ... other inputs

  notify:
    needs: release
    if: needs.release.outputs.released == 'true'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Released version ${{ needs.release.outputs.version }}"
```

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

### With Custom Assets Directory

If your WordPress.org assets (banners, icons, screenshots) are in a non-standard location:

```yaml
- name: Deploy Release
  uses: devuri/plugin-release-deploy-action@main
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    php-version: '8.1'
    plugin-slug: 'my-plugin'
    build-dir: './build/my-plugin/'
    zip-file: 'my-plugin.zip'
    assets-dir: './assets/wporg/'  # Custom location for banners/icons
    svn-username: ${{ secrets.WP_SVN_USERNAME }}
    svn-password: ${{ secrets.WP_SVN_PASSWORD }}
    deploy-to-wporg: 'true'
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

The `zip-file` input specifies which file(s) to upload to your GitHub Release. It uses the [AButler/upload-release-assets](https://github.com/AButler/upload-release-assets) action under the hood, which supports **glob patterns** for flexible file selection.

**Glob Pattern Support:**

You can specify files using various patterns:

```yaml
# Single file by name
zip-file: 'my-plugin.zip'

# Single file with path
zip-file: 'build/my-plugin.zip'

# Wildcard pattern
zip-file: '*.zip'

# Multiple files (semicolon-separated)
zip-file: 'my-plugin.zip;README.md;CHANGELOG.md'

# Multiple glob patterns
zip-file: 'build/*.zip;dist/*.tar.gz'

# Files in subdirectory
zip-file: 'build/trunk/*.zip'
```

**Path Resolution:**

All paths are **relative to your repository root**. The glob patterns are evaluated using the `fast-glob` library, so standard glob syntax applies:
- `*` matches any characters except `/`
- `**` matches any characters including `/`
- `{a,b}` matches either `a` or `b`

**Common Patterns:**

```yaml
# Upload main plugin zip only
zip-file: 'my-plugin.zip'

# Upload zip from build directory
zip-file: 'build/my-plugin.zip'

# Upload all zips in repository
zip-file: '*.zip'

# Upload zip plus documentation
zip-file: 'my-plugin.zip;README.md;LICENSE.txt'
```

**Best Practice:** Configure ZipIt's `outputFile` to create the zip in a predictable location (like repository root or a `dist/` folder), then reference it clearly in your workflow.

### Key Differences

| Aspect | `build-dir` | `zip-file` |
|--------|-------------|------------|
| **Format** | Directory (unpacked files) | File(s) or glob pattern |
| **Used By** | 10up WordPress.org deploy action | GitHub Release upload |
| **Required When** | Always (required input) | Always (required input) |
| **Used When** | `deploy-to-wporg: 'true'` | Every release |
| **Purpose** | SVN deployment source | Distribution download(s) |
| **Supports Multiple** | No (single directory) | Yes (via glob patterns) |

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

## WordPress.org Assets

The `assets-dir` input specifies where your WordPress.org plugin assets are located. These assets include:

- **Plugin banners** (`banner-772x250.png`, `banner-1544x500.png`)
- **Plugin icons** (`icon-128x128.png`, `icon-256x256.png`, `icon.svg`)
- **Screenshots** (`screenshot-1.png`, `screenshot-2.png`, etc.)

By default, the action looks for these in `.wordpress-org/` at your repository root. The 10up action will automatically copy these to the SVN `assets/` directory (which is separate from `trunk/`).

**Example structure:**
```
.wordpress-org/
├── banner-772x250.png
├── banner-1544x500.png
├── icon-128x128.png
├── icon-256x256.png
└── screenshot-1.png
```

---

## File Exclusion with `.distignore`

When deploying to WordPress.org, the [10up action](https://github.com/10up/action-wordpress-plugin-deploy) supports file exclusion via `.distignore` or `.gitattributes`.

**Note:** File exclusion only applies when you are **not** using `build-dir`. When `build-dir` is set, the action deploys all files in that directory and ignores `.distignore`/`.gitattributes`.

If you're building your plugin files directly in the repository (without a separate build directory), you can create a `.distignore` file to exclude development files:

```
# .distignore
/.wordpress-org
/.git
/.github
/node_modules
/tests

.distignore
.gitignore
composer.json
composer.lock
package.json
package-lock.json
phpunit.xml
```

For most modern plugin setups with a build step, using `build-dir` is recommended as it gives you explicit control over what gets deployed.

---

## Requirements

### Repository Configuration

**Release Please**  
Your repository must be configured for [`release-please`](https://github.com/google-github-actions/release-please-action) with:
- `release-please-config.json`
- `.release-please-manifest.json`

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
- The `outputFile` in `.zipit-conf.php` should match your `zip-file` input
- PHP 8.1+ is required (default `php-version` is `8.1`)

**ZipIt Output Location:**

ZipIt creates the zip file at the location specified by `outputFile` in your `.zipit-conf.php`:

```php
<?php
return [
    'baseDir' => __DIR__,
    'files' => ['src/', 'vendor/', 'plugin.php'],
    'exclude' => ['tests/', 'node_modules/'],
    'outputFile' => __DIR__ . '/my-plugin.zip',  // Creates zip in repo root
];
```

**Tip:** For simplicity, configure ZipIt to output the zip in your repository root, then use just the filename for `zip-file` input.

### Build Output

**Build Directory**  
The `build-dir` contains your unpacked plugin files ready for WordPress.org SVN deployment. When `deploy-to-wporg` is `'true'`, the 10up action syncs this directory to the WordPress.org repository's `trunk/` (and creates tags from it).

**Zip File(s)**  
The `zip-file` input specifies file(s) to upload to GitHub Releases using glob patterns. Files are automatically created by ZipIt when `use-zipit` is `'true'`, or by your custom build process when `use-zipit` is `'false'`. 

**Important:** 
- All paths in `zip-file` are relative to your repository root
- Supports glob patterns for flexibility (e.g., `*.zip`, `build/*.zip`, `plugin.zip;README.md`)
- Configure ZipIt's `outputFile` in `.zipit-conf.php` to control where files are created
- The uploaded file(s) and build directory serve different distribution channels (GitHub vs WordPress.org)

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
- **[ZipIt](https://github.com/devuri/zipit)** – PHP-based plugin packaging tool (optional)

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
