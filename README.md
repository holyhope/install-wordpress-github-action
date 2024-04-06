<!-- How to generate this file
```shell
npx --yes github-action-readme-generator@v1.7.2
```
-->
<div align="center" width="100%">
<!-- start title -->

# <img src="doc/assets/branding.svg" width="60px" align="center" alt="branding<icon:download-cloud color:blue>" /> GitHub Action: Install WordPress

<!-- end title -->
<!-- start badges -->

<a href="https://github.com/holyhope/install-wordpress-github-action/releases/latest"><img src="https://img.shields.io/github/v/release/holyhope/install-wordpress-github-action?display_name=tag&sort=semver&logo=github&style=flat-square" alt="Release" /></a><a href="https://github.com/holyhope/install-wordpress-github-action/releases/latest"><img src="https://img.shields.io/github/release-date/holyhope/install-wordpress-github-action?display_name=tag&sort=semver&logo=github&style=flat-square" alt="Release" /></a><img src="https://img.shields.io/github/last-commit/holyhope/install-wordpress-github-action?logo=github&style=flat-square" alt="Commit" /><a href="https://github.com/holyhope/install-wordpress-github-action/issues"><img src="https://img.shields.io/github/issues/holyhope/install-wordpress-github-action?logo=github&style=flat-square" alt="Open Issues" /></a><img src="https://img.shields.io/github/downloads/holyhope/install-wordpress-github-action/total?logo=github&style=flat-square" alt="Downloads" />

<!-- end badges -->
</div>

<!-- start description -->

Install WordPress locally using WP-CLI.

<!-- end description -->

<!-- start contents -->

This action installs a [WordPress](https://developer.wordpress.org) instance thanks to [wp-cli](https://wp-cli.org) in your GitHub workflow.

<!-- end contents -->

## Usage

<!-- start usage -->

```yaml
- uses: holyhope/install-wordpress-github-action@main
  with:
    # Description: Select which version you want to download. Accepts a version
    # number, 'latest' or 'nightly'.
    #
    # Default: latest
    wordpress_version: ""

    # Description: The title of the new site.
    #
    # Default: WordPress
    wordpress_title: ""

    # Description: The locale/language for the installation (e.g. `de_DE`).
    #
    # Default: en_US
    wordpress_locale: ""

    # Description: PHP version
    #
    # Default: 8.2
    php_version: ""

    # Description: PHP extensions to install. Each extension should be on a new line.
    #
    # Default: imagick
    php_extensions: ""

    # Description: WordPress Installation path
    #
    # Default: ${{ runner.temp }}/wordpress
    installation_path: ""

    # Description: GitHub token
    #
    # Default: ${{ github.token }}
    github_token: ""

    # Description: Path to the WP-CLI binary. To use a custom version of WP-CLI, see
    # the
    # [`install-wp-cli` GitHub action](https://github.com/marketplace/actions/install-wp-cli)
    # Default: Downloaded from the official WP-CLI website.
    #
    wp_cli_path: ""
```

<!-- end usage -->

## Inputs

<!-- start inputs -->

| **<b>Input</b>**                      | **<b>Description</b>**                                                                                                                                                                                                        | **<b>Default</b>**                        | **<b>Required</b>** |
| ------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------- | ------------------- |
| <b><code>wordpress_version</code></b> | Select which version you want to download. Accepts a version number, 'latest' or 'nightly'.                                                                                                                                   | <code>latest</code>                       | **false**           |
| <b><code>wordpress_title</code></b>   | The title of the new site.                                                                                                                                                                                                    | <code>WordPress</code>                    | **false**           |
| <b><code>wordpress_locale</code></b>  | The locale/language for the installation (e.g. `de_DE`).                                                                                                                                                                      | <code>en_US</code>                        | **false**           |
| <b><code>php_version</code></b>       | PHP version                                                                                                                                                                                                                   | <code>8.2</code>                          | **false**           |
| <b><code>php_extensions</code></b>    | PHP extensions to install.<br />Each extension should be on a new line.                                                                                                                                                       | <code>imagick</code>                      | **false**           |
| <b><code>installation_path</code></b> | WordPress Installation path                                                                                                                                                                                                   | <code>${{ runner.temp }}/wordpress</code> | **false**           |
| <b><code>github_token</code></b>      | GitHub token                                                                                                                                                                                                                  | <code>${{ github.token }}</code>          | **true**            |
| <b><code>wp_cli_path</code></b>       | Path to the WP-CLI binary.<br />To use a custom version of WP-CLI, see the [`install-wp-cli` GitHub action](https://github.com/marketplace/actions/install-wp-cli)<br />Default: Downloaded from the official WP-CLI website. |                                           | **false**           |

<!-- end inputs -->
<!-- start outputs -->

| **<b>Output</b>**                  | **<b>Description</b>**         |
| ---------------------------------- | ------------------------------ |
| <b><code>version</code></b>        | WordPress version              |
| <b><code>path</code></b>           | Path to WordPress installation |
| <b><code>plugins_path</code></b>   | Path to WordPress plugins      |
| <b><code>themes_path</code></b>    | Path to WordPress themes       |
| <b><code>admin_username</code></b> | WordPress admin username       |
| <b><code>admin_password</code></b> | WordPress admin password       |
| <b><code>db_name</code></b>        | WordPress database name        |
| <b><code>db_user</code></b>        | WordPress database user        |
| <b><code>db_pass</code></b>        | WordPress database password    |
| <b><code>db_host</code></b>        | WordPress database host        |
| <b><code>db_port</code></b>        | WordPress database port        |
| <b><code>wp_cli</code></b>         | Path to WP-CLI binary          |

<!-- end outputs -->
