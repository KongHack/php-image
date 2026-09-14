# KongHack PHP images

Reusable PHP-FPM base images for KongHack projects and other PHP applications.
The images are published to the GitHub Container Registry under the MIT License.

## Supported images

| PHP | Base image | Compatibility tag |
| --- | --- | --- |
| 8.4 | `php:8.4-fpm-bookworm` | `ghcr.io/konghack/php:8.4` |
| 8.5 | `php:8.5-fpm-bookworm` | `ghcr.io/konghack/php:8.5` |

There is intentionally no `latest` tag. Choose the PHP runtime line explicitly:

```dockerfile
FROM ghcr.io/konghack/php:8.5
```

The compatibility tags are rebuilt when their source changes and on the scheduled
maintenance build. Release tags combine the PHP line and project release, such as
`8.5-1.0.0`. Production consumers that need immutable inputs should pin a release
tag or, for strict reproducibility, the image digest shown by GHCR.

## Included software

Both images include Composer 2, Git, Unzip, the Debian-provided MariaDB client,
and the following PHP extensions:

- APCu
- BCMath
- cURL
- Exif
- Fileinfo
- GD with FreeType, JPEG, and WebP support
- GnuPG
- IMAP
- Intl
- Mailparse
- Mbstring
- MongoDB
- Msgpack
- PCNTL
- PDO and PDO MySQL
- POSIX
- Redis
- Sockets
- SSH2
- Swoole
- XML
- Zip

PECL extension versions are pinned as Docker build arguments in each Dockerfile.
The upstream PHP and Composer image tags and Debian package repositories remain
moving inputs. Scheduled builds pick up their security and patch updates; the
resulting GHCR digest is the authoritative immutable identifier.

## Configuration

The image activates the official PHP production configuration and then loads the
small set of generic overrides in `php-8.4/php.ini` or `php-8.5/php.ini`. The FPM
pool overrides listen on port `9000` and use a small dynamic process pool.

Application-specific settings belong in a downstream image. Later `.ini` and FPM
`.conf` files override these defaults:

```dockerfile
FROM ghcr.io/konghack/php:8.5

COPY app.ini /usr/local/etc/php/conf.d/zzz-app.ini
COPY app-fpm.conf /usr/local/etc/php-fpm.d/zzz-app.conf
```

The default session handler is PHP's filesystem handler. Applications that use
Redis, databases, or another custom session handler should configure it downstream.

PHP-FPM speaks FastCGI, not HTTP. Keep port `9000` on a private container network
behind a trusted reverse proxy; do not expose it directly to the internet.

## Build locally

Run builds from the repository root so the Dockerfiles can copy their matching
configuration files:

```console
docker build -f php-8.4/Dockerfile -t konghack/php:8.4 .
docker build -f php-8.5/Dockerfile -t konghack/php:8.5 .
```

Each Dockerfile verifies the required extensions, session handler, FPM
configuration, and Composer installation during the build.

## Publishing

The GitHub Actions workflow validates pull requests and publishes multi-platform
`linux/amd64` and `linux/arm64` images for the `main` branch, scheduled maintenance
builds, and releases.

- A build of the default branch updates `8.4` and `8.5`.
- A Git tag such as `v1.0.0` publishes `8.4-1.0.0` and `8.5-1.0.0`.
- Release tags must never be moved or reused.

After the first workflow publish, an organization owner must set the
`ghcr.io/konghack/php` package visibility to **Public** in GitHub's package
settings. The image's OCI source label links the package to this repository.

## License

[MIT](LICENSE) © 2026 GameCharmer
