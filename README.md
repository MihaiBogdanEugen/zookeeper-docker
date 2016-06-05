This image is proposed to [Docker official images](https://hub.docker.com/explore/). Please send [this PR](https://github.com/docker-library/official-images/pull/1765) some love to make this happen!

# Supported tags and respective `Dockerfile` links

* `3.3.6` [(3.3.6/Dockerfile)](https://github.com/31z4/zookeeper-docker/blob/master/3.3.6/Dockerfile)
* `3.4.8`, `latest` [(3.4.8/Dockerfile)](https://github.com/31z4/zookeeper-docker/blob/master/3.4.8/Dockerfile)

[![](https://badge.imagelayers.io/31z4/storm:latest.svg)](https://imagelayers.io/?images=31z4%2Fzookeeper:3.4.8,31z4%2Fzookeeper:3.3.6)

# What is Apache Zookeeper?

Apache ZooKeeper is a software project of the Apache Software Foundation, providing an open source distributed configuration service, synchronization service, and naming registry for large distributed systems. ZooKeeper was a sub-project of Hadoop but is now a top-level project in its own right.

> [wikipedia.org/wiki/Apache_ZooKeeper](https://en.wikipedia.org/wiki/Apache_ZooKeeper)

# How to use this image

## Start a Zookeeper server instance

	$ docker run --name some-zookeeper --restart always -d 31z4/zookeeper

This image includes `EXPOSE 2181` (the zookeeper port), so standard container linking will make it automatically available to the linked containers. Since the Zookeeper "fails fast" it's better to always restart it.

## Connect to Zookeeper from an application in another Docker container

	$ docker run --name some-app --link some-zookeeper:zookeeper -d application-that-uses-zookeeper

## Connect to Zookeeper from the Zookeeper command line client

	$ docker run -it --rm --link some-zookeeper:zookeeper 31z4/zookeeper zkCli.sh -server zookeeper

## ... via [`docker-compose`](https://github.com/docker/compose)

Example `docker-compose.yml` for `zookeeper`:

```yaml
version: '2'
services:
    zoo1:
        image: 31z4/zookeeper
        restart: always
        ports:
            - 2181
        environment:
            ZOO_MY_ID: 1
            ZOO_SERVERS: server.1=zoo0:2888:3888 server.2=zoo1:2888:3888 server.3=zoo2:2888:3888

    zoo2:
        image: 31z4/zookeeper
        restart: always
        ports:
            - 2181
        environment:
            ZOO_MY_ID: 2
            ZOO_SERVERS: server.1=zoo0:2888:3888 server.2=zoo1:2888:3888 server.3=zoo2:2888:3888

    zoo3:
        image: 31z4/zookeeper
        restart: always
        ports:
            - 2181
        environment:
            ZOO_MY_ID: 3
            ZOO_SERVERS: server.1=zoo0:2888:3888 server.2=zoo1:2888:3888 server.3=zoo2:2888:3888
```

This will start Zookeeper in [replicated mode](http://zookeeper.apache.org/doc/current/zookeeperStarted.html#sc_RunningReplicatedZooKeeper). Run `docker-compose up` and wait for it to initialize completely. Run `docker-compose ps` to figure out exposed ports.

> Please be aware that setting up multiple servers on a single machine will not create any redundancy. If something were to happen which caused the machine to die, all of the zookeeper servers would be offline. Full redundancy requires that each server have its own machine. It must be a completely separate physical server. Multiple virtual machines on the same physical host are still vulnerable to the complete failure of that host.

Consider using [Docker Swarm](https://www.docker.com/products/docker-swarm) when running Zookeeper in replicated mode.

## Configuration

Zookeeper configuration is located in `/conf`. One way to change it is mounting your config file as a volume:

	$ docker run --name some-zookeeper --restart always -d -v $(pwd)/zoo.cfg:/conf/zoo.cfg 31z4/zookeeper

## Environment variables

Variables below are mandatory if you want to run Zookeeper in replicated mode.

### `ZOO_MY_ID`

The id must be unique within the ensemble and should have a value between 1 and 255. Do note that this variable will not have any effect if you start the container with a `/data` directory that already contains the `myid` file.

### `ZOO_SERVERS`

This variable allows you to specify a list of machines of the Zookeeper ensemble. Each entry has the form of `server.id=host:port:port`. Entries are separated with space. Do note that this variable will not have any effect if you start the container with a `/conf` directory that already contains the `zoo.cfg` file.

## Where to store data

This image is configured with volumes at `/data` and `/datalog` to hold the Zookeeper in-memory database snapshots and the transaction log of updates to the database, respectively.

> Be careful where you put the transaction log. A dedicated transaction log device is key to consistent good performance. Putting the log on a busy device will adversely effect performance.

# License

View [license information](https://github.com/apache/zookeeper/blob/release-3.4.8/LICENSE.txt) for the software contained in this image.
