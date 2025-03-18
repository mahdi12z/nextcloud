# nextcloud
Document: Setting Up the Nextcloud Project Environment and Installation
# 1-Set Up the Nextcloud Project Environment

Before installing Nextcloud, we need to prepare a suitable project environment where the Nextcloud server will be launched. Follow the steps below:
# A) Create a Project Directory

```bash
mkdir nextcloud
```

# B) Create Configuration Files
In this step, we will create two configuration files: docker-compose.yaml and .env, which will be used to configure Docker containers and Docker networks.

docker-compose.yaml: Contains instructions to set up multi-container Docker applications in YAML format.
.env: Stores environment variables used inside the docker-compose.yaml configuration.
Run the following commands to create these two files:
```bash
touch /nextcloud/docker-compose.yaml
touch /nextcloud/.env

```
# C) Create a Docker Network
We will set up a Docker network for communication among the containers that will host Nextcloud.

(Replace nodeshift_network with your preferred name)

```bash
docker network create nodeshift_network
```

# 2- Configure Nextcloud

The next steps will guide you in configuring your Nextcloud server settings, primarily the docker-compose.yaml file and later the .env file.

# A) Configure Reverse Proxy
A reverse proxy is used in Nextcloud configuration to improve security, performance, and scalability. It can handle SSL encryption for your server and allows you to use a custom domain name instead of an IP address by routing traffic. We will use Nginx for this purpose.

# B) Open the docker-compose.yaml File
Use the following command to open the file in the Nano editor:
```bash
version: '3'
services:
  proxy:
    image: jwilder/nginx-proxy:alpine
    labels:
      - "com.github.jrcs.letsencrypt_nginx_proxy_companion.nginx_proxy=true"
    container_name: nextcloud-proxy
    networks:
      - nodeshift_network
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./proxy/conf.d:/etc/nginx/conf.d:rw
      - ./proxy/vhost.d:/etc/nginx/vhost.d:rw
      - ./proxy/html:/usr/share/nginx/html:rw
      - ./proxy/certs:/etc/nginx/certs:ro
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/tmp/docker.sock:ro
    restart: unless-stopped

  db:
    image: mariadb
    container_name: nextcloud-mariadb
    networks:
      - nodeshift_network
    volumes:
      - db:/var/lib/mysql
      - /etc/localtime:/etc/localtime:ro
    environment:
      - MYSQL_ROOT_PASSWORD
      - MYSQL_PASSWORD
      - MYSQL_DATABASE
      - MYSQL_USER
    restart: unless-stopped

  app:
    image: nextcloud:latest
    container_name: nextcloud-app
    networks:
      - nodeshift_network
    depends_on:
      - proxy
      - db
    volumes:
      - nextcloud:/var/www/html
      - ./app/config:/var/www/html/config
      - ./app/custom_apps:/var/www/html/custom_apps
      - ./app/data:/var/www/html/data
      - ./app/themes:/var/www/html/themes
      - /etc/localtime:/etc/localtime:ro
    environment:
      - VIRTUAL_HOST=<YOUR_SERVER_IP>
    restart: unless-stopped

volumes:
  nextcloud:
  db:

networks:
  nodeshift_network:

```
# 3- 




