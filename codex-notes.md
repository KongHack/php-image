# Codex Notes

- Last updated: 2026-09-15
- Purpose: public-safe working memory for establishing the KongHack PHP container images.

## Goal

Publish reusable PHP-FPM base images through GitHub Container Registry. Private consumers can derive thin, locally hosted images from these public bases and add their own configuration or proprietary components without duplicating the public build.

## Confirmed Decisions

1. This repository contains only the PHP image sources. Nginx, Docker Compose, private CI configuration, infrastructure notes, and deployment-specific networking are out of scope.
2. The source directories are named `php-8.4/` and `php-8.5/`.
3. The public package name will be `ghcr.io/konghack/php`.
4. PHP versions will be expressed as tags:
   - `ghcr.io/konghack/php:8.4`
   - `ghcr.io/konghack/php:8.5`
5. Version-specific tags such as `8.4` and `8.5` should be preferred over an ambiguous `latest` tag.
6. The public images should install sanitized, generic `php.ini` and PHP-FPM pool configuration rather than carrying unused configuration files.
7. The project uses the MIT License with `GameCharmer` as the public copyright-holder name. It permits closed-source commercial and noncommercial downstream use without visible website attribution.
8. Both runtime lines use the supported Debian Bookworm variant. PHP 8.4 was moved from Bullseye because Debian 11 reached end-of-life on 2026-08-31.

## Tag Model

- `php` is the image/package name.
- `8.4` and `8.5` are mutable compatibility tags that identify the PHP runtime line.
- Reproducible release tags should also be published alongside the compatibility tags, for example `8.4-1.0.0` and `8.5-1.0.0`.
- Downstream production builds may pin a release tag or digest when immutable inputs are required.

Example downstream usage:

```dockerfile
FROM ghcr.io/konghack/php:8.4
```

## Current Repository State

- The repository is public at `https://github.com/KongHack/php-image`.
- The PHP 8.4 and 8.5 image sources have been prepared for public use.
- Both source directories currently contain:
  - `Dockerfile`
  - `php.ini`
  - `www.conf`
- The Dockerfiles install the PHP and FPM configuration, pin PECL releases, and run build-time smoke checks.
- `README.md`, `SECURITY.md`, the MIT `LICENSE`, public-safe ignore files, Dependabot configuration, and a GHCR publishing workflow are present.
- The workflow builds `linux/amd64` and `linux/arm64`, publishes compatibility tags from `main`, publishes release tags from bare semantic-version Git tags, and intentionally does not publish `latest`.
- Release Git tags use bare semantic versions such as `1.0.0`, matching the repository's `VERSION` file and README version section.
- The compatibility images have been published publicly at `ghcr.io/konghack/php`.
- Dependabot monitors GitHub Actions only; Docker base tags are refreshed by scheduled image builds instead of cross-version update pull requests.
- A sensitive-string scan found no private organization names, internal infrastructure, credentials, tokens, or private filesystem paths.

## Important Technical Findings

1. Local AMD64 builds succeeded for PHP 8.4.25 and PHP 8.5.10 on Bookworm.
2. Every documented extension loaded, the effective session handler was `files`, FPM configuration validation passed, and Composer, Git, MariaDB client, and Unzip were available.
3. PECL dependencies are pinned. Upstream PHP and Composer tags plus Debian repositories remain moving inputs; GHCR digests are the strict reproducibility boundary.
4. The MariaDB remote setup-script pipe was removed. Both images use Debian's packaged MariaDB 10.11 client.
5. The image remains intentionally broad and is approximately 249–251 MB locally. A future multi-stage or squashed-image hardening pass could further reduce transferred build-tool layers inherited from the official PHP base.
6. Actionlint, Hadolint, YAML parsing, whitespace checks, runtime checks, and the public-safety scan passed locally.

## Next Steps

1. Commit and push the bare-semantic-version workflow and documentation changes.
2. Run `gittag '1.0.0'` to update the version files, create the first release tag, and push it.
3. Confirm that the release workflow publishes `8.4-1.0.0` and `8.5-1.0.0`.
