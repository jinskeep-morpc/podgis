# Setting up Postgresql, Postgis, and PGadmin in Podman

## Introduction

Working with large amounts of spatial data can be computationally expensive or impossible if we rely on spatial files and typical data workflows. 
In order to create a modern and efficient data system, it makes sense to deploy a spatial databases which allows us to store and access data more quickly
and transparently. The state-of-the-art in spatial databases in PostgreSQL with the PostGIS extension. 

Postgres is a good choice because it is scalable, and is supported by many of the software and systems we currently use (ArcGIS, frictionless, etc.). 
Though adopting and implementing this software requires that we host the database somehow, somewhere. MORPC's current technology stack does not support the Data Teams
needs in this area. 

One technology that would help the data team develop agile, secure, and useful databases is containers. A container is similar to a virtual machine, but 
is host agnostic and more transferable. A popular container software is Docker, which could be useful, but I believe to due to potential security concerns (root daemon)
and limitation to networking capabilities, I will suggest that we use Podman, a docker alternative. 

I will be following [this tutorial](https://www.riannek.de/2025/how-to-use-postgis-in-podman/) to set up a postgres instance in Podman, in order to hopefully facilitate the development of a more modern and efficient data storage
solution for MORPC Data and Mapping team. 

## Setup Repo

1. Create a repository and clone it to local machine named podgis.
2. Create this README
3. Create folders for the separate data, one `postgres`, one `pgadmin`

## Install Podman and podman compose

### For linux

1. Run `sudo apt -y install podman podman-compose`

### For Windows

Follow [this tutorial](https://github.com/containers/podman/blob/main/docs/tutorials/podman-for-windows.md).

## Create a Pod

Using a compose.yaml file we will designate the containers that will be included in the pod. 

1. Create a .env file with usernames and passwords. Use actual passwords. Obviously. 

```
# .env

POSTGRES_USER=postgres # Don't change this, this is determined by Postgresql
POSTGRES_PASSWORD=SUPERSECRETPASSWORD
PGADMIN_DEFAULT_EMAIL=jinskeep@morpc.org
PGADMIN_DEFAULT_PASSWORD=ANOTHERSECRETPASSWORD
```

2. Add the .env to your .gitignore

```
# .gitignore

.env
```

3. Create the compose.yaml

> **NOTE** Podman assigns an ip range for the pod and specific ips for containers. 
> To assign static ips for containers, we must set up a network manually and assign
> the static ips to the containers. 

```
# compose.yaml

services:
  postgres:
    image: docker.io/postgis/postgis
    environment:
      - POSTGRES_USER=${POSTGRES_USERNAME}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - ./postgres:/var/lib/postgresql/data
    ports:
      - 5432:5432
    networks:
      podgis_network:
        ipv4_address: 10.89.0.5
        aliases: 
          - postgres

  pgadmin:
    image: docker.io/dpage/pgadmin4
    environment:
      - PGADMIN_DEFAULT_EMAIL=${PGADMIN_DEFAULT_EMAIL}
      - PGADMIN_DEFAULT_PASSWORD=${PGADMIN_DEFAULT_PASSWORD}
    volumes:
      - ./pgadmin:/var/lib/pgadmin:Z
    ports:
      - 8080:80
    networks:
      podgis_network:
        ipv4_address: 10.89.0.6
        aliases: 
          - pgadmin

networks:
  podgis_network:
    driver: bridge
    ipam:
      config:
        - subnet: 10.89.0.1/16
          ip_range: 10.89.0.0/24
          gateway: 10.89.0.1

```

4. Run the pod. In a terminal run `podman-compose up -d`


## Link containers and set up server

1. Log into pgadmin

- Visit 127.0.0.1:8080 *(This is the port assigned to pgadmin in the compose.yaml file.)*
- Login using the email and password defined in the .env file. 

2. Connect to postgres server

- Click "Add New Server" or right click on the "Servers" drop-down on the left panel and click "Register > Server".
- In the "General" tab, name the server. In this case, I name it "podgis".
- In the "Connection" tab,
    - In "Host name/address", add the ip address defined in the compose file for the postgres container.
    - Leave "Port" as 5432, unless change in the compose "ports" field.
    - In "Username", add "postgres". This is the default system manager account for all postgres databases.
    - In "Password", add the password form POSTGRES_PASSWORD in the .env file.
    - Mark to save the password.
    - Click "Save"

3. You should see in the left hand drop-down panel a server named "podgis" or the name you just designated and you should have been taken to the dashboard panel showing the activity for the server. 


## Create a database and activate postgis extension

1. Right click on the server name in the left drop-down panel. 
2. Click "Create > Database"
3. Name the datebase in the General tab.
4. Under the Definition tab, select:
    - Template as "template_postgis"
    - Tablespace as "pg_default"
    - Collation as "en_US.utf8"
    - Character Type as "en_US.utf8

## Connect to database and add tables using python.