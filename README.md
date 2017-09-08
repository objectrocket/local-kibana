# local-kibana
A Dockerized local install of Kibana that can be used to connect to an ObjectRocket cluster remotely. It leverages the official [Kibana Docker](https://github.com/elastic/kibana-docker) image and wraps it up with some configuration info.

## Requirements

* docker version **17.05.0+**
* docker-compose version **1.12.0+**

> Earlier versions of docker/docker-compose will work, but you may need to change the Dockerfile to replace the `${KB_VER}` with the specific version you want at the end of the `FROM docker.elastic.co/kibana/kibana:${KB_VER}` line.

## Setup

First clone this repo to your local machine

### Configuration Settings

The repo now makes all required configuration via the `docker-compose.yml` file, so the minimum required settings can all be made in one place. The default version of this file is below:

```yaml
version: '2'

services:
  kibana:
    build:
      context: kibana/
      args:
        KB_VER: '5.5.1'
    volumes:
      - ./kibana/config/:/usr/share/kibana/config
    environment:
      SERVER_NAME: "kibana"
      SERVER_HOST: '"0"'
      ELASTICSEARCH_URL: 'https://dc-port-id.es.objectrocket.com:port'
      ELASTICSEARCH_USERNAME: 'myuser'
      ELASTICSEARCH_PASSWORD: 'mypassword'
    ports:
      - "5601:5601"
```

The settings you will need to change are:

- **KB_VER** : This is the version of KIbana you'd like. You can see which versions of Kibana are compatible with your version of Elasticsearch in the official Elastic [support matrix](https://www.elastic.co/support/matrix#show_compatibility) .
- **ELASTICSEARCH_URL** : One of the https connection strings (NOT the provided Kibana string) from the ObjectRocket UI
- **ELASTICSEARCH_USERNAME** : A Username and password as configured in the ObjectRocket UI
- **ELASTICSEARCH_PASSWORD** : Any other kibana settings you'd like to tweak (like a different kibana.index)

> Kibana can only accept a single host for the elasticsearch url. It is not able to round-robin, like other elasticsearch clients, so you'll need to take the list of hosts provided in the ObjectRocket UI and select only one host to use for the local Kibana instance.

### kibana.yml

The Kibana configuration file is ``local-kibana/kibana/config/kibana.yml``. In the latest version of this repo, the required settings should all be configured in the `environment` section of the `docker-compose.yml` file. However, the file remains in the repo to use if you want to make further customizations. Just note that the environment variables will override the settings in the `kibana.yml` file.

### ObjectRocket ACLs

Make sure that you have an elasticsearch ACL set up to allow you to connect from your local machine. From the ObjectRocket UI:

1. Click on the instance that you'd like to connect to
2. Click the "+ Add ACL" button
3. Select "Elasticsearch" for ACL Role, then select the "MyIP" button and then "Add ACL Entry"


## Launching Kibana

From a terminal in the newly cloned 'local-kibana' directory:

```
  $ docker-compose build --no-cache
  $ docker-compose up
```

This will launch the Kibana docker container and you will start to see status information from Kibana printed to your terminal. Once Kibana has finished loading up, you should see something like:

```
  kibana_1  | {"type":"log","@timestamp":"2017-06-29T21:34:36Z","tags":["status","ui settings","info"],"pid":1,"state":"green","message":"Status changed from yellow to green - Ready","prevState":"yellow","prevMsg":"Elasticsearch plugin is yellow"}
```

## Connecting to Kibana

In a browser, you can now navigate to ``localhost:5601``.

Kibana will start loading and you will need to enter your ObjectRocket Elasticsearch username and password one more time.

## Using the non-default kibana index

By default, and on the ObjectRocket service, Kibana stores all of its data in an Elasticsearch index called ``.kibana``. If you'd like to create your own set of Kibana items that won't show up in the hosted ObjectRocket Kibana, you must add the following setting to your kibana.yml file:

```
  kibana.index: "yourkibanaindexname"
```

If creating a different kibana index you will need to create an index pattern. This can be done in the UI or with the command line using the following:
```bash
$ curl -XPUT -D- 'http://localhost:9200/.yourkibanaindexname/index-pattern/logstash-*' \
    -H 'Content-Type: application/json' \
    -d '{"title" : "logstash-*", "timeFieldName": "@timestamp", "notExpandable": true}'
```
```bash
$ curl -XPUT -D- 'http://localhost:9200/.yourkibanaindexname/config/5.4.1' \
    -H 'Content-Type: application/json' \
    -d '{"defaultIndex": "logstash-*"}'
```

## Shutting down Kibana

To shut down the Kibana, use a terminal to navigate to the local-kibana directory and run the docker-compose down command:

```
  $ cd /some/directory/local-kibana
  $ docker-compose down
```
