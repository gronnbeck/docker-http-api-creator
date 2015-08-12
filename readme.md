# Experiment: Build and run docker containers using HTTP API

#### 12.08.15
So now that I got it to work. I want to play with the API a bit. Let us
start it up again, this time in background mode. Because now I can start
creating a more complex application while I am able to play with the API.

```sh
docker run --restart always -d --name docker-proxy \
-e DOCKER_USER=user -e DOCKER_PASSWORD=pass -p 2375:2375 \
srault95/docker-proxy-api
```

since ``curl`` does not pretty print JSON for me. I will pipe data from curl
into ``python -mjson.tool`` like this:

```sh
curl -k -u user:pass https://<ip>:2375/<rpc> | python -mjson.tool
```

As I see it there are two ways of creating images through the [Docker API](https://docs.docker.com/reference/api/docker_remote_api_v1.20/)

 1. Tar a folder with a Dockerfile and ship to til /build
 2. Create image by pulling it from the Docker registry

None of these suits my current purpose. I want to easily create an image
by pointing it to a Dockerfile specified in gist. However, it does seem
like (1) with the ``remote`` query parameter might solve my problems.

> remote – A Git repository URI or HTTP/HTTPS URI build source.
If the URI specifies a filename, the file’s contents are placed into
a file called ``Dockerfile``.

So I will try this under the assumption that the ``{{ TAR STREAM}}``
input is allowed to empty.

So I created the following [Gist](https://gist.githubusercontent.com/gronnbeck/541ece530754413929f8/raw/8f9c40fbaf5794200ec9a5873378ad107dff5cbf/Docker):

```Dockerfile
FROM node:0.10

ENV SHELL /bin/bash

CMD []
```

Running the following command

```sh
$ curl -k -u user:pass -XPOST \
https://<host>:2375/build\?remote\=https://gist.githubusercontent.com/gronnbeck/541ece530754413929f8/raw/8f9c40fbaf5794200ec9a5873378ad107dff5cbf/Docker

# <A stream of data impossible for the human to process>

{"status":"Status: Downloaded newer image for node:0.10"}
{"stream":" ---\u003e f7e97f7762a3\n"}
{"stream":"Step 1 : ENV SHELL /bin/bash\n"}
{"stream":" ---\u003e Running in b87adcf105cc\n"}
{"stream":" ---\u003e 5bb715101fde\n"}
{"stream":"Removing intermediate container b87adcf105cc\n"}
{"stream":"Step 2 : CMD \n"}
{"stream":" ---\u003e Running in 0d49627651cf\n"}
{"stream":" ---\u003e ee65e194451b\n"}
{"stream":"Removing intermediate container 0d49627651cf\n"}
{"stream":"Successfully built ee65e194451b\n"}
```

and it seems to have worked unexpectedly well. I will not be
entirely happy until I get the more complex things right.
But it is a start. I SSH into my server and verified that the
image was created, YEAH!

```sh

$ docker images

REPOSITORY                         TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
<none>                             <none>              ee65e194451b        17 seconds ago      632.7 MB

```

So what is the next step? I want to do this incrementally, but
my end goal is to create a simple way of getting my code
to run on a server. That also includes an elegant way of starting
databases and other dependencies easily.

So the next step I guess is to create a simple app for creating
customizable Dockerfile for my projects and some utility for
telling docker to create the image.

#### 10.08.15 - 2
Tried docker-proxy-api on my DigitalOcean ubuntu and it worked after configuring the docker deamon. Just followed guide on the
[github repo](https://github.com/srault95/docker-proxy-api).
Also, copied below for my own references

```
172.17.42.1 is default ip address for docker0 interface in ubuntu trusty

$ vi /etc/default/docker

DOCKER_OPTS="-H tcp://172.17.42.1:4444 -H unix:///var/run/docker.sock"

$ service docker reload
```

Do not know how secure this is. But I will not worry about right now.
I verified that the proxy was set up correctly by curling the api.

```sh
curl -k -u docker:docker https://<ip>:2375/containers/json
```

Also, as a to be able to debug what went wrong I ran the proxy
in debug mode, with the following command:

```sh
docker run -it --rm -p 2375:2375 srault95/docker-proxy-api
```

#### 10.08.15

Tried download the docker-proxy-api as described below. But without any luck.
Could not get it to work on OS X using boot2docker. Maybe it will work on
ubuntu with native Docker. Maybe look into that another time.


```sh
docker pull srault95/docker-proxy-api
```

Run it
```sh
docker run --restart always -d --name docker-proxy \
-e DOCKER_USER=user -e DOCKER_PASSWORD=pass srault95/docker-proxy-api
```

Could not verify.
