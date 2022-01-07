# Home Assistant - Setup

## Governing principle

- Install HA with [Home Assistance install](https://github.com/slittorin/home-assistant-install/).
- If not otherwise stated, the user `pi` performs all actions.

## Preparation

1. Under `/srv`:
   - Create the file `.env`.
   - Create the file `/srv/docker-compose.yml` with the following content:
     ```
     version: '3'
     
     services:
     
     volumes:
     ```

## Installation for MariaDB

1. Check versions of available docker images for MariaDB at [Docker - MariaDB](https://hub.docker.com/_/mariadb).
   - If you do not want the 'latest' version, , use version number.
   - At time of writing (20220207) the 'latest' version is 10.6.5 (isolated with `sudo docker image inspect mariadb` and looking for 'MARIADB_VERSION').
2. Create the directory `/srv/ha-db`, and the following sub-directories:
   - At present no specific directories are used.
3. For the following file `/srv/.env` add the following content:
