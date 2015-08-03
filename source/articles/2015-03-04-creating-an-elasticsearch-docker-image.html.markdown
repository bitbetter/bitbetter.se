---
date: 2015-03-04
title: Deploying ELK with Docker - Part 1
subtitle: How I created Docker images for Elasticsearch, Logstash and Kibana.
cover: containers-bg.jpg
layout: post
tags: Docker
published: false
---

## Creating an Elasticsearch Docker image

...


### Ubuntu and Java

I chose to base my Docker image on Ubuntu. And I also want to use Oracle Java 8.
Instead of doing all that myself, I will instead reuse images that already exist
on [Docker Hub](https://registry.hub.docker.com/). I chose [dockerfile/java:oracle-java8](https://registry.hub.docker.com/u/dockerfile/java/).

```conf
FROM dockerfile/java:oracle-java8
```

### Elasticsearch

Elasticsearch can be installed as a deb package or downloaded as a tarball. I
chose deb package since it is easier to install, and also easy to run (but more
on that topic later).

```conf
RUN wget -qO- https://packages.elasticsearch.org/GPG-KEY-elasticsearch | sudo apt-key add -
RUN add-apt-repository "deb http://packages.elasticsearch.org/elasticsearch/1.4/debian stable main"
RUN apt-get -q update
RUN apt-get -qy install elasticsearch=1.4.4
```

### Remove old Elasticsearch indexes

I want to avoid the disk filling up with Elasticsearch's database.
[Elasticsearch Curator]() is the best tool I found to help with removing indexes,
and what I want command ´curator delete --disk-space 4´ does what I want.

```
RUN apt-get -y install python python-dev python-distribute python-pip
RUN pip install elasticsearch-curator
```

Then set up a job running daily, via cron, that limites the size of the
Elasticsearch indexes to 4GB, by removing the oldest indexes.

```
@daily
```

```
ADD elasticsearch.crontab /tmp/elasticsearch.crontab
RUN crontab /tmp/elasticsearch.crontab
```

### Monitoring
```
# Install monit
RUN apt-get -y install monit
ADD elasticsearch.monit.conf /etc/monit/conf.d/elasticsearch.monit.conf
```

### Ports
```
# Expose ports
EXPOSE 9200 9300
```

### External data
```
# Make elasticsearch data folder mountable
VOLUME ["/var/lib/elasticsearch"]
```

### Go!
```
# Run supervisor
WORKDIR /tmp
CMD ["/usr/bin/monit", "-d 10", "-I"]
```

## Running containers locally

## Additional resources
