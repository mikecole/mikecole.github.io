---
layout: post
title: Adjusting Docker To Handle Recent mssql Container Changes
date: 2024-08-01
tags: docker mssql
meta-description: Microsoft made recent changes to their mssql/server:2022-latest and mssql/server:2019-latest Docker images, and it broke a lot of builds
---

Our team uses `docker-compose` to create temporary ephemeral database that we can use for integration tests and dispose. We were using the `mssql/server:2019-latest` image, but today the builds all randomly stopped working without a lot of info. The only detail I could find was a single line in the build log: `dependency failed to start: container <name> is unhealthy`. Everything worked fine locally. I found [this post](https://www.reddit.com/r/docker/comments/15m3eg5/docker_compose_nocache/) on Reddit which helped me realize my local system was pulling the old cached versions of the images. I performed the following two commands to recreate the error on my machine: `docker compose pull` then `docker compose up -d --force-recreate`.

When I visited [Microsoft's Docker Repository](https://mcr.microsoft.com/en-us/product/mssql/server/tags) I saw that the image was modified today, so I figured something must have changed with the image. After more digging, I found the following [GitHub Issue](https://github.com/microsoft/mssql-docker/issues/892) that detailed the same issue for the `mssql/server:2022-latest` image. Here I saw that the latest packages are now including the new `mssql-tools18` package, which changes the directory of the ODBC tools we were using for a health check on our container. It seems like a lot of people copied and pasted the same health check from somewhere. The old broken health check: `test: /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P "$$SA_PASSWORD" -Q "SELECT 1" || exit 1` becomes the new working health check: `test: /opt/mssql-tools18/bin/sqlcmd -C -S localhost -U sa -P "$${SA_PASSWORD}" -Q "SELECT 1" -b -o /dev/null` and now everything works again.

Another alternative would be to pin your container version to `mssql/server:2022-CU13-ubuntu-20.04` instead of using the unpinned version.

They probably should have used a symlink for backwards compatiblity. It appears they haven't yet fixed their documentation to reflect this change, so beware!