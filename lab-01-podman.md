# Lab 1 (optional): Podman Introduction

⚠️ WARNING: if you are unable to install podman locally, you may skip this lab.

## Prerequisites

This set of instructions requires that podman is already installed and podman commands can be run from a bash shell. You can get more information at the [Podman website](https://podman.io/). The following section gives you a starting point to use podman on top of MacOS ( see also https://abvijaykumar.medium.com/avengers-of-container-world-episode-1-podman-hands-on-f81d8ee93b57 ). We prefer though to leverage the production stream of podman. 

## Setting Up Podman on MacOS with Vagrant and Virtual Box

For MacOS these instructions help you setup a running podman environment.

- Install VirtualBox, e.g. from [VirtualBox Downloads](https://www.virtualbox.org/wiki/Downloads) 
- Install Vagrant, e.g. from [Vagrant Downloads](https://www.vagrantup.com/downloads)
- Initialize Vagrant in a new directory
```bash
$ vagrant init centos/8
```
- Provision the CentOs environment in VirtualBox
```bash
$ vagrant up
```
- Login to CentOS
```bash
$ vagrant ssh
```
- Install Podman
```bash
$ sudo yum -y install podman
```

## Working with podman - "Hello World" using tomcat

Launch a shell and confirm that podman is installed. The version number isn't particular important.

```bash
$ podman -v
podman version 2.2.1
```

Search for tomcat
```bash
$ podman search tomcat
```

Login with podman to docker.io
```bash
$ podman login docker.io
```

Validate the pull of the tomcat image
```bash
$ podman images

REPOSITORY                TAG     IMAGE ID      CREATED     SIZE
docker.io/library/tomcat  latest  08efef7ca980  2 days ago  679 MB
```

Run the image
```bash
$ podman run -d --name tomcat-1 -p 8080:8080 tomcat
```
Find the container instance
```bash
$ podman ps
```

Stop the container instance and delete the stopped instance.
```bash
$ podman stop tomcat-1
$ podman rm tomcat-1
```

## Working with podman - Leveraging CouchDB

We use the following parameters in docker run:

- -e allows to set environment variables
- -d runs the container in detached mode returning only the container ID
- -p <host port>:<container port> binds the container to the host port

```bash
$ podman run -d -p:5984:5984 -e COUCHDB_USER=admin -e COUCHDB_PASSWORD=password apache/couchdb:latest
Unable to find image 'apache/couchdb:latest' locally
latest: Pulling from apache/couchdb
d121f8d1c412: Pull complete
92e1697550ec: Pull complete
2f41f51f40fc: Pull complete
0eb42e9db372: Pull complete
f233415680da: Pull complete
2fcfeb745daa: Pull complete
499b885fbd81: Pull complete
02b6d34405ec: Pull complete
b4976aa20614: Pull complete
77bb2d366040: Pull complete
93df8fed85d6: Pull complete
Digest: sha256:95bb69dd700df71578ac1b4011f40ca66369e6d132b80f1520a3d21e7bff084f
Status: Downloaded newer image for apache/couchdb:latest
4de83b227b03bf17a42a1915d87928e310c44764ef9776083963276eb114e908
```

The output above was captured while the image was still downloading from docker-hub. When the download is done you don't see anything from the container, like with hello-world. Instead you see a long hex id like 4de83b227b03bf17a42a1915d87928e310c44764ef9776083963276eb114e908. This is the id of the container.

Here is how you would see the running container. Notice only the first part of that long hex id is displayed. Typically this is more than enough to uniquely identify that container. docker ps provides information about when the container was created, how long it has been running, the name of the image, as well as the name of the container. Note that each container must have a unique name. You can specify a name for each container as long it is unique.

```bash
$ docker ps
CONTAINER ID   IMAGE                 COMMAND   CREATED        STATUS       PORTS           NAMES
4de83b227b03   apache/couchdb:latest "tini.."  4 minutes ago  Up 4 minutes 4369/tcp [...]  eloquent_ardinghelli
```

To validate CouchDB is up an running execute a curl command against the exposed port. You should receive a JSON-based welcome message.

```bash
$ curl localhost:5984
{"couchdb":"Welcome","version":"3.1.1","git_sha":"ce596c65d","uuid":"509813e7f91cb260fb30039f7421abab","features":["access-ready","partitioned","pluggable-storage-engines","reshard","scheduler"],"vendor":{"name":"The Apache Software Foundation"}}
```

Create a new customer database with the following curl command.

```bash
$ curl -X PUT http://admin:password@localhost:5984/customer
{"ok":true}
```

Insert a customer entry to the database leveraging the following HTTP PUT request.

```bash
$ curl -X PUT http://admin:password@localhost:5984/customer/1 -d '{"firstName": "Hugo", "lastName": "Boss"}'
{"ok":true,"id":"1","rev":"1-b8745a7b645d3f29f0b19274bf63707b"}
```

To finalize our tests we will retrieve the created document from the customer database.

```bash
$ curl -X GET http://admin:password@localhost:5984/customer/1
{"_id":"1","_rev":"1-b8745a7b645d3f29f0b19274bf63707b","firstName":"Hugo","lastName":"Boss"}
```

Now let's clean up the environment. Start with stopping the container with the proper container id.

```bash
$ docker stop 4de83b227b03
4de83b227b03
```

Notice the image still exists. Go ahead and try to delete the CouchDB image with `docker rmi apache/couchdb`

```bash
$ docker rmi apache/couchdb
Error response from daemon: conflict: unable to remove repository reference "apache/couchdb" (must force) - container 4de83b227b03 is using its referenced image 1916ea74a583
```

Oops, we can't delete the image until we remove the couchdb container. Note that `docker ps -a` will show us all the containers, not just the ones that are running.

```bash
$ docker ps -a
4de83b227b03   apache/couchdb:latest   "tini -- /docker-ent…"   17 minutes ago   Exited (0) 4 minutes ago   eloquent_ardinghelli
```

With `docker rm <container id>` you can remove a container. Delete the couchdb container, delete the couchdb image and make sure it's gone.

```bash
$ docker rm 4de83b227b03
4de83b227b03

$ docker rmi apache/couchdb
Untagged: apache/couchdb:latest
Untagged: apache/couchdb@sha256:95bb69dd700df71578ac1b4011f40ca66369e6d132b80f1520a3d21e7bff084f
Deleted: sha256:1916ea74a583b0e0df17b2d14418095ce7a614027cb363d014f2b8af584ebcf2
Deleted: sha256:9894abfedce3d1ad1832f96f48c62b1e2c5e7f0e7e79d5b35e3d766df588221c
Deleted: sha256:79de819838b0479a03a484cff2f8df9e64207de6f1f0303c508b3627c64754bc
Deleted: sha256:618ec9e04da8fe87c14a90caacf957f9308124e219604fc3d5b60a6e4ba52df8
Deleted: sha256:e8c96b7c719eb50273cd44b25aaf5b108e5334088912e749f88ad1356c32cb04
Deleted: sha256:19760c4688fcb49f88e75d80de15dc559cad3eb014bbc3d6f42140ae7beac721
Deleted: sha256:e6fa3b97bbfb030a9f33b6c8a96034ef1e58a5559490fbe87e7b4124ddaf15e0

$ docker images
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
hello-world                  latest              bf756fb1ae65        20 mins ago         13.3kB
```

**Congratulations, you have done your first steps with Docker.**


Optional Lab 1

If you want to build your own image you can checkout the **/node_docker** folder in this repository and build + test your own based on a simple Node.JS sample (look into the README for some hints).
