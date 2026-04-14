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

