# Most common installation of PHP for many projects

Images are published as `krizius/php:<tag>` on Docker Hub. Build from the repo root:

```bash
docker build -f 7.4/apache/Dockerfile -t krizius/php:7.4-apache .
```

| Tag | Base | PHP (2026-10-26) | Notes |
|---|---|---|---|
| `7.4-apache` | `debian:bookworm-slim` + packages.sury.org | 7.4.33 | EOL upstream, security fixes only from Sury |
| `8.0-apache` | `debian:bookworm-slim` + packages.sury.org | 8.0.30 | EOL upstream, security fixes only from Sury |
| `8.1-apache` | `debian:bookworm-slim` + packages.sury.org | 8.1.34 | EOL upstream, security fixes only from Sury |
| `8.2-apache` | `php:8.2-apache-bookworm` | 8.2.35 | supported upstream |
| `8.3-apache` | `php:8.3-apache-bookworm` | 8.3.36 | supported upstream |
| `8.3-cli` | `php:8.3-cli-bookworm` + openswoole | 8.3.36 | supported upstream |
| `8.1-swoole` | `php:8.1-cli-bookworm` + openswoole 4.11 | 8.1.34 | CLI only |

## PHP2Shell (CVE-2026-109618 / CVE-2026-109619)

Upstream fixed 8.2.35, 8.3.36, 8.4.27 and 8.5.12. Everything older (7.4, 8.0, 8.1) is EOL and gets
the fix only as a backport in packages.sury.org, therefore those images are built from Debian + Sury
instead of the official `php:*-apache` images. They keep the layout of the official image
(`docker-php-entrypoint`, `apache2-foreground`, `/usr/local/bin/php`, `/usr/local/etc/php/conf.d`,
`.htaccess` honoured under `/var/www`). `session.upload_progress.enabled=0` is set as defence in depth.
The `-pre-php2shell` tags on Docker Hub hold the images from before this change.

Rebuild regularly (`docker build --no-cache --pull`) to pick up new security fixes.

## Host requirements

Debian 12 images (glibc >= 2.34) need Docker >= 20.10 on the host. Docker 19.03's seccomp profile
blocks the `clone3` syscall, which makes `exec()`, `popen()` and `passthru()` fail inside the container.
