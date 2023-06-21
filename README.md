# Traefik compose configuration
[![License: MIT](https://img.shields.io/github/license/DurhamARC/traefik-compose)](https://github.com/DurhamARC/traefik-compose/blob/main/LICENSE)

This repository contains a set of configuration files for Traefik which allow to spin up a front-end load balancer in Docker very quickly.

## Usage
Clone this repository into its own directory and run `docker compose up -d`. This will bring up Traefik and the docker proxy it uses to communicate with running containers. Use separate directories for services using the loadbalancer.

Traefik will automatically request SSL certificates for services configured to use it. Perform configuraiton using `docker` labels. An example of usage with `docker compose` is as follows:

```Dockerfile
version: '3.6'

services:
  nginx:
    image: nginx:alpine
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=traefik_traefik"
      - "traefik.http.routers.web-http.entrypoints=http"
      - "traefik.http.routers.web-http.rule=Host(`${DOMAIN}`)"
      - "traefik.http.routers.web-http.middlewares=httpsRedirect@file"
      - "traefik.http.routers.web.entrypoints=https"
      - "traefik.http.routers.web.rule=Host(`${DOMAIN}`)"
      - "traefik.http.routers.web.tls=true"
      - "traefik.http.routers.web.tls.certresolver=letsencrypt"
      - "traefik.http.routers.web.middlewares=secured@file"
      - "traefik.http.services.web.loadbalancer.server.port=80"
      - "com.centurylink.watchtower.enable=true"
    volumes:
       - "./conf/nginx.conf:/etc/nginx/conf.d/mysite.conf:ro"
       - "./volumes/${DOMAIN}/:/var/www/html"
    networks:
      - traefik

networks:
  traefik:
    name: traefik_traefik
    external: true

```
This will bring up a webserver running nginx, linked to the Traefik load balancer.

Note that services are required to expose a plain HTTP port. It is possible to configure Traefik to communicate internally via https but this is beyond the scope of this repository.

## Used by

  * [Metagenome](https://meta-genome.org)
  * [ManyFEWS](https://floodwarnings.durham.ac.uk)

## Durham University Project Team

| Project Member                                       | Contact address                                                   | Role
|------------------------------------------------------|-------------------------------------------------------------------|--------------------------------
| Dr. [Samantha Finnigan](github.com/sjmf)             | [samantha.finnigan@dur.ac.uk](mailto:samantha.finnigan@dur.ac.uk) | Research Software Engineer (RSE


