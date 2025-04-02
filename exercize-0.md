# [Docker](https://www.docker.com/ "Docker")

Containerization has revolutionized application development and deployment, with Docker leading the way. Docker packages applications into **images** that contain everything needed to run them: code, runtime environment, libraries, and configuration files. These images run in **containers**, isolated processes that consume only the resources they need.


## Table of Contents
* [Key Components of Docker](#key-components-of-docker)
* [Why Use Docker?](#why-use-docker?)
* [Common instructions](#common-instructions)
* [Example with dockerfile Deploying of Flask](#example-with-dockerfile-deploying-of-flask)
    * [Example with Flask](#example-with-flask)
    * [Deploying](#deploying)
    * [Expected result](#expected-result)
* [Docker Compose](#docker-compose)
* [Key Components of Docker Compose](#key-components-of-docker-compose)
* [Example with docker compose Deploying of Flask & Redis](#example-with-docker-compose-deploying-of-flask-and-redis)
    * [Example with Flask & Redis](#example-with-flask-and-redis)
    * [Deploying service](#deploying-service)
    * [Expected results](#expected-results)
* [Docker Swarm](#docker-swarm)
* [Key Components of Docker Swarm](#key-components-of-docker-swarm)
* [Example with Docker Swarm Deploying of Nginx](#example-with-docker-swarm-deploying-of-nginx)
    * [Initialize a Swarm Manager](#initialize-a-swarm-manager)
    * [Check the Swarm Manager’s IP (if already running)](#check-the-swarm-managers-ip-if-already-running)
    * [Deploy Nginx with 3 Replicas](#deploy-nginx-with-3-replicas)
    * [Verify the Service](#verify-the-service)
    * [Create a docker-compose.yml for Swarm (Declarative Service Model)](#create-a-docker-composeyml-for-swarm-declarative-service-model)
    * [Deploy the Stack](#deploy-the-stack)

* [Common Use Cases for Docker](#common-use-cases-for-docker)
* [Troubleshooting Common Issues](#troubleshooting-common-issues)
* [Conclusion](#conclusion)

## Key Components of Docker

Here are some of the key components of Docker:

- **Docker Engine**: The core of Docker, responsible for creating and managing containers.
- **Docker Image**: A read-only template used to create containers. It includes the application code and its dependencies.
- **Docker Hub**: A cloud-based repository for finding and sharing container images.
- **Dockerfile**: A script containing instructions to build a Docker image.
- **Docker Registry**: A storage and distribution system for Docker images. It supports both public and private image repositories.


## Why Use Docker?

Docker offers numerous benefits, such as:

- Simplified application deployment and management.
- Improved resource efficiency by isolating applications.
- Compatibility across different environments.


## Basic Docker Commands

Here are some essential Docker commands:

- **docker run <image_name>**: Runs a container from a Docker image.
- **docker ps**: Lists all running containers.
- **docker build -t <image_name> <path_to_dockerfile>**: Builds a Docker image from a Dockerfile.
- **docker stop <container_id>**: Stops a running container.
- **docker rm <container_id>**: Removes a stopped container.
- **docker pull <image_name>**: Pulls an image from Docker Hub or a registry.


## Common instructions
Some of the most common instructions in a Dockerfile include:

- **FROM <image>** : this specifies the base image that the build will extend.
- **WORKDIR <path>** : this instruction specifies the "working directory" or the path in the image where files will be copied and commands will be executed.
- **COPY <host-path> <image-path>** : this instruction tells the builder to copy files from the host and put them into the container image.
- **RUN <command>** : this instruction tells the builder to run the specified command.
- **EXPOSE <port-number>** : this instruction sets configuration on the image that indicates a port the image would like to expose.
- **CMD ["<command>", "<arg1>"]** : this instruction sets the default command a container using this image will run.

## Example with dockerfile Deploying of Flask
In this example, we use the official Python Docker image and install Flask to run a simple web application:

### Example with Flask

[Dockerfile](Dockerfile)
```yaml
FROM python:3.10
RUN pip install flask
COPY app.py /app.py
CMD ["python","app.py"]
```


[app.py](app.py)
```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "hello world!"

if __name__ == "__main__":
    app.run(host="0.0.0.0")
```

### Deploying

Build the Docker image:
```shell
docker build -t testfask .
```
Check the available images:
```shell
docker images
```
Run the Docker container:
```shell
docker run -d -p 5000:5000 testfask
```
Access the application:

Open http://localhost:5000/ in your browser to see the running Flask app.

### Expected result

Listing containers must show one container running and the port mapping as below:
```console
$ docker container ls
NAME                COMMAND             SERVICE             STATUS              PORTS
flask-web-1         "python3 app.py"    web                 running             0.0.0.0:5000->5000/tcp
```


## Docker Compose
Docker Compose is a tool for defining and running multi-container applications. It is the key to unlocking a streamlined and efficient development and deployment experience.

Compose simplifies the control of your entire application stack, making it easy to manage services, networks, and volumes in a single, comprehensible YAML configuration file. Then, with a single command, you create and start all the services from your configuration file.

## Key Components of Docker Compose
Here are some common instructions used in a Docker Compose file (docker-compose.yml):

- **version: Specifies the version of Docker Compose syntax you are using.
- **services: Defines the containers to be run. Each service is essentially a container.
- **image: Specifies the image to use for the container, either from Docker Hub or a local image.
- **build: Builds the container image from a Dockerfile in the specified directory.
- **ports: Maps the container ports to the host machine’s ports.
- **volumes: Mounts a host directory or volume to a container’s filesystem.
- **environment: Sets environment variables inside the container.
- **depends_on: Defines dependencies between services, ensuring a startup order.
- **networks: Defines the networks that containers can join.
- **restart: Configures the container’s restart policy.

## Example with docker compose Deploying of Flask and Redis
In this example, we use the official Python & Redis image Docker image and install Flask to run a simple web application:

### Example with Flask and Redis

[Dockerfile](Dockerfile)
```yaml
# syntax=docker/dockerfile:1
FROM python:3.10-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run", "--debug"]
```



[app.py](app.py)
```python
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return f'Hello World! I have been seen {count} times.\n'
```

[Dockerfile](Dockerfile)
```yaml
services:
  web:
    build: .
    ports:
      - "8000:5000"
  redis:
    image: "redis:alpine"
```

### Deploying service

```console
$ docker compose up

Creating network "composetest_default" with the default driver
Creating composetest_web_1 ...
Creating composetest_redis_1 ...
Creating composetest_web_1
Creating composetest_redis_1 ... done
Attaching to composetest_web_1, composetest_redis_1
web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
redis_1  | 1:C 17 Aug 22:11:10.480 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis_1  | 1:C 17 Aug 22:11:10.480 # Redis version=4.0.1, bits=64, commit=00000000, modified=0, pid=1, just started
redis_1  | 1:C 17 Aug 22:11:10.480 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
web_1    |  * Restarting with stat
redis_1  | 1:M 17 Aug 22:11:10.483 * Running mode=standalone, port=6379.
redis_1  | 1:M 17 Aug 22:11:10.483 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
web_1    |  * Debugger is active!
redis_1  | 1:M 17 Aug 22:11:10.483 # Server initialized
redis_1  | 1:M 17 Aug 22:11:10.483 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
web_1    |  * Debugger PIN: 330-787-903
redis_1  | 1:M 17 Aug 22:11:10.483 * Ready to accept connections
```
Access the application:

Open http://localhost:8000/ in your browser to see the running Flask app.

Refresh the page.

The number should increment


### Expected results

Listing containers must show one container running and the port mapping as below:
```console
$ docker container ls
NAME                COMMAND             SERVICE             STATUS              PORTS
flask-web-1         "python3 app.py"    web                 running             0.0.0.0:5000->5000/tcp
```

## Docker Swarm
Docker Swarm is Docker’s native container orchestration tool that lets you manage and scale multiple containers across multiple machines (nodes). It enables you to deploy and manage containerized applications in a clustered environment.

# Key Components of Docker Swarm
- **Cluster Management** – Groups multiple Docker hosts (machines) into a Swarm cluster (manager and worker nodes).
- **Load Balancing** – Distributes traffic between containers automatically.
- **Scaling** – Easily scale services up or down using docker service scale.
- **Rolling Updates** – Update services with minimal downtime.
- **Fault Tolerance** – If a container or node fails, Swarm automatically restarts the service elsewhere.
- **Declarative Service Model** – Uses docker-compose.yml for defining services.

# Example with docker Swarm Deploying of Nginx
create an Nginx service with 3 replicas:

# Initialize a Swarm Manager
If you want to initialize a new Docker Swarm cluster, use:
```console
$ docker swarm init --advertise-addr <MANAGER_NODE_IP>
```
-** <MANAGER_NODE_IP> is the IP address of the machine acting as the manager.
with this comand you can find NODE IP
```console
$ hostname -I | awk '{print $1}'
```
# Check the Swarm Manager’s IP (if already running)

If Docker Swarm is already initialized, run:
```console
$ docker node ls
```
This will list all nodes. The manager node’s IP will be shown.

# Deploy Nginx with 3 Replicas
Run this command on the manager node to create an Nginx service with 3 replicas:

```console
$ docker service create --name nginx-service --replicas 3 -p 80:80 nginx
```

# Verify the Service
Check if the service is running:

```console
$ docker service ls
```
# Create a docker-compose.yml for Swarm(Declarative Service Model)
Docker Swarm uses a stack instead of docker-compose up. Below is an easy example:

```yaml
version: "3.8"

services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    deploy:
      replicas: 3  # Run 3 instances of Nginx
      restart_policy:
        condition: on-failure
```

# Deploy the Stack
Run the following command to deploy this stack in Swarm mode:

```console
docker stack deploy -c docker-compose.yml nginx2test
```
Check if the service is running:

```console
$ docker service ls
```

## Common Use Cases for Docker

1. **Microservices Architecture**: Easily deploy and manage microservices with isolated containers.
2. **Continuous Integration and Continuous Deployment (CI/CD)**: Simplifies CI/CD pipelines by packaging applications consistently.
3. **Hybrid and Multi-Cloud Applications**: Run applications seamlessly across on-premises and cloud environments.
4. **Development and Testing**: Create reproducible development environments to eliminate "it works on my machine" issues.
5. **Resource Optimization**: Efficiently use system resources by running lightweight containers.


## Troubleshooting Common Issues

1. **Container Not Starting**: Use `docker logs <container_id>` to check for error messages and identify the issue.
2. **Image Build Failures**: Review the Dockerfile for syntax errors or missing dependencies.
3. **Network Connectivity Issues**: Verify network settings and use `docker network ls` to inspect available networks.
4. **Storage Issues**: Check available disk space and ensure proper configuration of volume mounts.


## Conclusion

Docker is a powerful and versatile containerization platform that simplifies application development, deployment, and management across diverse environments. By leveraging Docker, teams can improve efficiency, scalability, and consistency.
