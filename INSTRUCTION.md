# Running Todolist with Docker Compose

## Prerequisites

- Install Docker Engine with the Compose plugin, or Docker Desktop (Windows/macOS).
- Start Docker. On Windows, use Linux containers.
- Make sure host ports `8080` (application) and `3306` (MySQL) are available.
- Open a terminal in the project directory containing `docker-compose.yml`.

Check Docker and Compose:

```sh
docker --version
docker compose version
```

The commands below use `docker compose`. If your environment uses the legacy standalone command, replace it with `docker-compose`, for example `docker-compose up --build -d`.

## Build and start

Validate the Compose configuration:

```sh
docker compose config --quiet
```

Build both images and start the services in the background:

```sh
docker compose up --build -d
```

The services are:

- `mysql`: MySQL database `app_db`, accessible to the application at `mysql:3306`.
- `todolistapp`: Django application, accessible at http://localhost:8080/.

The application entrypoint runs database migrations before starting the server. The first build and MySQL initialization can take some time.

Check container status and follow startup logs:

```sh
docker compose ps
docker compose logs -f
```

Press `Ctrl+C` to stop following logs; the background containers keep running.

Once the logs show that the Django server has started, open http://localhost:8080/ in a browser.

To build and run with logs attached instead:

```sh
docker compose up --build
```

In this mode, `Ctrl+C` stops the containers.

## Stop and start existing containers

Stop both services while keeping their containers and database data:

```sh
docker compose stop
```

Start those existing containers again:

```sh
docker compose start
```

After changing application code, dependencies, or Dockerfiles, rebuild and recreate the services:

```sh
docker compose up --build -d
```

## Remove containers and preserve database data

Stop and remove the project containers and network:

```sh
docker compose down
```

MySQL stores its data in the named volume `db-data`, mounted at `/var/lib/mysql`. This volume survives `stop` and ordinary `down`, so saved todos remain available on the next startup:

```sh
docker compose up -d
```

## Optional: reset the database

**This permanently deletes all database data, including accounts and todos.** Only run this when you intentionally want a fresh database:

```sh
docker compose down --volumes
docker compose up --build -d
```

## Troubleshooting

Inspect recent logs for each service:

```sh
docker compose logs --tail=100 mysql
docker compose logs --tail=100 todolistapp
```

The current `depends_on` configuration controls startup order but does not wait for MySQL to accept connections. If migrations fail because MySQL is still initializing, wait until the MySQL logs report that it is ready for connections, then restart the application:

```sh
docker compose restart todolistapp
```

For automatic readiness handling, the Compose configuration would need a MySQL healthcheck and a `service_healthy` dependency condition.

If a host port is already in use, stop the conflicting service or change the host side of the corresponding port mapping in `docker-compose.yml`.

## References

- [Docker Compose up](https://docs.docker.com/reference/cli/docker/compose/up/)
- [Docker Compose down](https://docs.docker.com/reference/cli/docker/compose/down/)
- [Startup order and database readiness](https://docs.docker.com/compose/how-tos/startup-order/)
