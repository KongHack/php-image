# Codex Notes

- Last updated: 2026-09-14 16:09 EDT
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

- The repository has no commits yet.
- The PHP 8.4 and 8.5 image sources have been prepared for public use.
- Both source directories currently contain:
  - `Dockerfile`
  - `php.ini`
  - `www.conf`
- The Dockerfiles install the PHP and FPM configuration, pin PECL releases, and run build-time smoke checks.
- `README.md`, `SECURITY.md`, the MIT `LICENSE`, public-safe ignore files, Dependabot configuration, and a GHCR publishing workflow are present.
- The workflow builds `linux/amd64` and `linux/arm64`, publishes compatibility tags from `main`, publishes release tags from `v*` Git tags, and intentionally does not publish `latest`.
- A sensitive-string scan found no private organization names, internal infrastructure, credentials, tokens, or private filesystem paths.
- The user-staged `.gitignore` remains staged; all files created or changed during the publication pass remain unstaged/untracked for review.

## Important Technical Findings

1. Local AMD64 builds succeeded for PHP 8.4.25 and PHP 8.5.10 on Bookworm.
2. Every documented extension loaded, the effective session handler was `files`, FPM configuration validation passed, and Composer, Git, MariaDB client, and Unzip were available.
3. PECL dependencies are pinned. Upstream PHP and Composer tags plus Debian repositories remain moving inputs; GHCR digests are the strict reproducibility boundary.
4. The MariaDB remote setup-script pipe was removed. Both images use Debian's packaged MariaDB 10.11 client.
5. The image remains intentionally broad and is approximately 249–251 MB locally. A future multi-stage or squashed-image hardening pass could further reduce transferred build-tool layers inherited from the official PHP base.
6. Actionlint, Hadolint, YAML parsing, whitespace checks, runtime checks, and the public-safety scan passed locally.

## Next Steps

1. Review the working tree and create the repository's first commit when approved.
2. Push `main` and confirm the multi-platform GitHub Actions build publishes both compatibility tags.
3. After the first publish, set `ghcr.io/konghack/php` package visibility to Public in the GitHub organization package settings.
4. Verify anonymous pulls for `8.4` and `8.5`, then create the first `vX.Y.Z` release tag when ready.
5. Repeat the sensitive-information audit immediately before making the repository public.
