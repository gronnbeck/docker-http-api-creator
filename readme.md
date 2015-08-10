# Experiment: Build and run docker containers using HTTP API

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
