# Passbolt CE on Docker with a NGINX Reverse-Proxy

This is a Docker Compose configuration file for running two instances of Passbolt Pro with separate databases using a single Nginx front-end as a reverse proxy.

Prerequisites
- Docker
- Docker Compose

## Usage

- Clone this repository.
- Open a terminal in the directory containing the docker-compose.yml file.
- Run `docker-compose -f docker-compose-ce.yaml up -d` to start the containers in the background.
- Access the Passbolt instances at:
  - https://passbolt.local/docker 
  - https://passbolt.local:4443/k8s

**NOTE:** The path /docker and /k8s are placeholders and can be changed for any other names. With this repository as a template, it will be easier for you to create as many sub folders as you need following the conventions. You will need to update the `.mysql/init.sql` and `.docker-compose-ce.yaml` file to create new database credentials and containers.
Also, do not forget to adapt each `.nginx/` files.

## Configuration

### Nginx

The Nginx container listens on ports 8080 and 4433 on the host and proxies requests to the Passbolt containers on ports 80 and 443 respectively. The configuration for Nginx is located in ./nginx/core.conf.d/proxy.conf and ./nginx/nginx.conf.

### Passbolt

There are two Passbolt containers, passbolt and passbolt2, each with their own database defined in the db container. The containers are configured with the following environment variables:

- `APP_FULL_BASE_URL`: The full URL for the Passbolt instance, including the protocol and port number.
- `APP_BASE`: The base path for the Passbolt instance.
- `DATASOURCES_DEFAULT_HOST`: The hostname of the database server.
- `DATASOURCES_DEFAULT_USERNAME`: The username for connecting to the database.
- `DATASOURCES_DEFAULT_PASSWORD`: The password for connecting to the database.
- `DATASOURCES_DEFAULT_DATABASE`: The name of the database.

The command section of each container defines a startup script that waits for the db container to be available before running the Passbolt entry point script.

#### Create an admin

You'll need to create as many admin as passbolt containers, `passbolt` and `passbolt2` are the provided containers. Please, change the container name following this example.

```
docker-compose -f docker-compose-ce.yaml exec passbolt su -m -c "/usr/share/php/passbolt/bin/cake \
  passbolt register_user \
  -u your@email \
  -f your_name \
  -l your_surname \
  -r admin" -s /bin/sh www-data
```

It should output something like: https://passbolt.local:4443/k8s/setup/install/user_id/token_id

### Volumes

The following volumes are defined:

- `database_volume`: The volume used by the db container to store its data.
- `gpg_volume and gpg_volume2`: The volumes used by the Passbolt containers to store GPG keys.
- `jwt_volume and jwt_volume2`: The volumes used by the Passbolt containers to store JWT authentication keys.
- `init_sql`: The volume used by the db container to execute an init.sql script on startup. The init.sql script creates the passbolt and passbolt2 databases and users, and grants them the necessary privileges.

## License

Please note that this docker-compose configuration is provided as a reference for using Passbolt CE and is subject to the Passbolt CE license. The software and all materials contained in this repository are reserved to Passbolt SA.

For more information about Passbolt CE licensing, please refer to the Passbolt CE License Agreement at https://www.passbolt.com/terms.