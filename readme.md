
# Jenkins 101 
This repository is used to learn basic concepts of Jenkins like Jenkins setup, running cloud agents, pipeline creation. It is adapted from the course of https://github.com/devopsjourney1/jenkins-101/tree/master. 
![Logo](jenkins.png)
## YouTube Link
For the full 1 hour course watch on youtube:
https://www.youtube.com/watch?v=6YZvp2GwT0A

# Installation

## How to setup Jenkins from Docker

Assuming you built the Jenkins image. If not: checkout [Docker (jenkins.io)](https://www.jenkins.io/doc/book/installing/docker/)

1. Create a bridge network in Docker
    
    ```bash
    docker network create jenkins
    ```
    
2. In order to execute Docker commands inside Jenkins nodes, run the following image:
    
    ```bash
    docker run --name jenkins-docker --rm --detach \
      --privileged --network jenkins --network-alias docker \
      --env DOCKER_TLS_CERTDIR=/certs \
      --volume jenkins-docker-certs:/certs/client \
      --volume jenkins-data:/var/jenkins_home \
      --publish 2376:2376 \
      docker:dind --storage-driver overlay2
    ```
    
3. Run Jenkins image using the `docker:dind` as Docker Host:
    
    ```bash
    docker run --name jenkins-blueocean --restart=on-failure --detach \
      --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
      --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
      --publish 8080:8080 --publish 50000:50000 \
      --volume jenkins-data:/var/jenkins_home \
      --volume jenkins-docker-certs:/certs/client:ro \
      myjenkins-blueocean:2.414.3-1
    ```
    
4. Checkout the Jenkins UI at `http:localhost:8080`.

## Test cloud connection

Cloud providers can be: Docker, Kubernetes, AWS, etc. Here we create a Docker cloud provider on Host Machine and we connect it to Jenkins. 

Create an `alpine/socat` container (i.e. cloud container) to forward traffic from Jenkins to to Docker Desktop on Host Machine:

```bash
docker run -d --restart=always -p 127.0.0.1:2377:2375 --network jenkins -v /var/run/docker.sock:/var/run/docker.sock alpine/socat tcp-listen:2375,fork,reuseaddr unix-connect:/var/run/docker.sock
```

Make sure you set up the Docker Host URI within Jenkins:

1. Check IP address of cloud provider IP address:
    
    `docker inspect [container_name]` and look for `IPAddress` . 
    
2. Set up the Docker Host URI: `tcp://[IPAddress]:2375` where you substitute `[IPAddress]` with the correct address.
3. Click on `Test connection`.
## Using Jenkins Python Agent
```
docker pull devopsjourney1/myjenkinsagents:python
```
