# Gitea Documentation

This documentation details the deployment and configuration of a self-hosted Gitea instance utilizing Docker Compose and a dedicated database container.

## Deployment Architecture

The environment consists of two isolated containers communicating over an internal Docker network:

- **Gitea**: The front end web application and Git routing service.
- **MYSQL**: The backend database. (Used for user data, metadata, ...)

## Configuration

Below you will find the used `docker-compose.yml` file I used:


```yaml
version: "3"

networks:
  gitea:
    external: false

services:
  server:
    image: docker.gitea.com/gitea:1.25.4
    container_name: gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - GITEA__database__DB_TYPE=mysql
      - GITEA__database__HOST=db:3306
      - GITEA__database__NAME=gitea
      - GITEA__database__USER=gitea
      - GITEA__database__PASSWD=gitea
    restart: always
    networks:
      - gitea
    volumes:
      - ./gitea:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "8080:3000"
      - "2221:22"
    depends_on:
      - db
  db:
    image: docker.io/library/mysql:8
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=gitea
      - MYSQL_USER=gitea
      - MYSQL_PASSWORD=gitea
      - MYSQL_DATABASE=gitea
    networks:
      - gitea
    volumes:
      - ./mysql:/var/lib/mysql
```

## Initialization

Start the environment in detached mode:

```bash
docker compose up -d
```

Navigate to `http://<server-ip>:8080` to finalize the installation. The environment variables in the Compose file pre-fill the database parameters. The following must be configured manually during the initial web setup:

Server Domain: Set to the server's IP address or fully qualified domain name.

- **Server Domain**: Set to the server's IP address or fully qualified domain name.
- **Gitea base URL**: `http://<server-ip>:8080`
- Administrator account: Expand the bottom section to set the primary admin credentials to prevent automatic admin assignment to the first registered user.
