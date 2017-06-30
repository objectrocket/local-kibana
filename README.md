# local-kibana
A Dockerized local install of Kibana that can be used to connect to an ObjectRocket cluster remotely. It leverages the official [Kibana Docker](https://github.com/elastic/kibana-docker) image and wraps it up with some configuration info.

## Requirements

docker
docker-compose

## Setup

First clone this repo to your local machine

### kibana.yml

In the newly cloned repo, the Kibana configuration file is ``local-kibana/kibana/config/kibana.yml``. You'll need to update the included version with:

- One of the https connection strings (NOT the provided Kibana string) in the ObjectRocket UI
- A Username and password as configured in the ObjectRocket UI
- Any other kibana settings you'd like to tweak (like a different kibana.index)

> Kibana can only accept a single host for the elasticsearch url. It is not able to round-robin, like other elasticsearch clients, so you'll need to take the list of hosts provided in the ObjectRocket UI and select only one host to use for the local Kibana instance.

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

## Shutting down Kibana

To shut down the Kibana, use a terminal to navigate to the local-kibana directory and run the docker-compose down command:

```
  $ cd /some/directory/local-kibana
  $ docker-compose down
```
