---
date: 2015-03-11
title: Deploying ELK with Docker
subtitle: How I deployed Elasticsearch, Logstash and Kibana Docker containers with Docker Compose.
cover: containers-bg.jpg
layout: post
tags: Docker
published: false
---

This is how I deployed Docker containers with Elasticsearch, Logstash and
Kibana using Docker Compose. I have created the images myself, and they can be found on [Docker Hub](https://registry.hub.docker.com/u/bitbetter/) and on
[Github](http://github.com/bitbetter).

(I will not cover what ELK is, but there is [plenty of information about ELK](http://www.elasticsearch.org/overview/) if you want to know more.)

## Introduction to Docker Compose

[Docker compose](https://docs.docker.com/compose/) (or **compose** for short) is a tool
for defining and running complex applications with Docker. With Compose, you define a
multi-container application in a single file, then spin your application up in a
single command which does everything that needs to be done to get it running.

The compose configuration file, `docker-compose.yml`, contains the same information that you put into the `docker run` command.

As an example, the docker command to run a container with Logstash, linking to Elasticsearch and publishing up port 4560 on the host:

```sh
$ docker run -d --name logstash -p 4560:4560 -l elasticsearch:elasticsearch bitbetter/logstash
```

The same configuration using compose:

```conf
logstash:
  image: bitbetter/logstash
  links:
    - elasticsearch
  ports:
    - "4560:4560"
```

Nice! Compose really makes the configuration easy to write and clear to read.

### Linking containers

Logstash and Kibana needs to connect to Elasticsearch. One way is to use container linking, where the hostname of linked containers are associated with an IP address in the local /etc/hosts file.

The following configuration example will let the Logstash container use the hostname `elasticsearch` as part of a url when connecting, e.g. `http://elasticsearch:9300`.

```conf
elasticsearch:
  image: bitbetter/elasticsearch

logstash:
  image: bitbetter/logstash
  links:
    - elasticsearch
```

### Publishing ports

The Logstash service needs to receive connections from outside the host to a port on the container, and that port needs to be declared as published.

The following configuration example publishes the internal port 4560 as port 4560 on the host. (Note the quotationmarks, as they should not really be needed, but due to a parsing rule in YAML it is good practice to use them.)

```
logstash:
  image: bitbetter/logstash
  ports:
    - "4560:4560"
```

### Mounting data volume containers

When a container is updated, it also needs to be replaced. In the case of the Elasticsearch container, all indexed data would be lost if we upgraded the code. Not good. A solution for that is to create a separate container that only stores data - a data volume container - and mount that container onto the Elasticsearch container when starting it.

The following configuration example will create an Elasticsearch container, and a data volume container for `/var/lib/elasticsearch`, and will mount the data volume container on the Elasticsearch container.

```conf
elasticsearchdata:
  image: busybox
  volumes:
    - /var/lib/elasticsearch

elasticsearch:
  image: bitbetter/elasticsearch
  volumes_from:
    - elasticsearchdata
```

## ELK stack configuration

Let's put it all toghether. The following is the actual configuration I used to deploy Elasticsearch, Logstash and Kibana.

```conf
elasticsearchdata:
  image: busybox
  volumes:
    - /var/lib/elasticsearch

elasticsearch:
  image: bitbetter/elasticsearch
  volumes_from:
    - elasticsearchdata

logstash:
  image: bitbetter/logstash
  links:
    - elasticsearch
  ports:
    - "4560:4560"

kibana:
  image: bitbetter/kibana
  links:
    - elasticsearch
  ports:
    - "80:5601"
```

## Docker host

One gotcha with docker-compose is that `DOCKER_HOST` needs to be set. If you are on OSX and using boot2docker, then it is already set for you.

If you want to run on a remote instance, one of the best alternatives are [DigitalOcean](https://www.digitalocean.com/?refcode=138e6c803d00), and you can use [this link](https://www.digitalocean.com/?refcode=138e6c803d00) to get $10 free hosting!

Once you have created your instance and [installed Docker](https://docs.docker.com/installation/ubuntulinux/#ubuntu-trusty-1404-lts-64-bit), set `DOCKER_HOST="tcp://<instance_ip>:2375"` and you should be ready to deploy!

## Deploy!

Starting our containers is easy, just run `docker-compose up -d`. Without the `-d` you will stop the containers when you exit the command. I also usually specify the project name with `-p` for a nicer grouping, otherwise the group name will be the parent folder name.

```shell
$ docker-compose -p elk up -d
Creating elk_elasticsearchdata_1...
Creating elk_elasticsearch_1...
Creating elk_logstash_1...
Creating elk_kibana_1...

# List containers
$ docker-compose ps
Name                      Command                          State
-------------------------------------------------------------------
elk_elasticsearch_1       /usr/bin/monit -d 10 -I          Up
elk_elasticsearchdata_1   /bin/sh                          Exit 0
elk_kibana_1              /usr/bin/supervisord --con ...   Up
elk_logstash_1            /usr/bin/monit -d 10 -I          Up
```

Great success!

## Further reading

* [ELK Docker images on GitHub](http://github.com/bitbetter)
* [Elasticsearch](http://www.elasticsearch.org/overview)
* [Docker Compose](https://docs.docker.com/compose)
* [DigitalOcean](https://www.digitalocean.com/?refcode=138e6c803d00) - Easy to use, great prices and offer [$10 free hosting](https://www.digitalocean.com/?refcode=138e6c803d00)!

---

If you like this you should [follow me on Twitter](https://twitter.com/martinkretz).
