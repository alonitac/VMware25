# Docker compose brief

Are you tired by executing `docker run` for multiple containers? so are we.

[Docker Compose](https://docs.docker.com/compose/) is a tool for defining and running multi-container Docker applications.
With Compose, you use a YAML file to configure your application's services. 
Then, with a single command, you create and start all the services from your configuration.

Using Compose is essentially a two-step process:

1. Define the services that make up your app in `docker-compose.yml` so they can be run together in an isolated environment.
2. Run `docker compose up` and the Docker compose command starts and runs your entire app.

A `docker-compose.yml` looks like this:

```yaml
services:
  web:
    image: alonithuji/counter_web:1
    ports:
      - "8000:5000"
    volumes:
      - logvolume01:/var/log
    depends_on:
      - redis
  redis:
    image: redis

volumes:
  logvolume01:
```

The given Docker Compose file describes a multi-service application with two services: `web` and `redis`. 
Here's a breakdown of some components:

1. `services:` begins the services section, where you define the containers for your application.
2. `web:` defines a service named `web`. 
3. `volumes:` defines volume mappings between the host and container.
4. `redis:` defines a service named `redis`. This service uses the official Redis image.

## Compose benefits 

Using Docker Compose offers several benefits:

- **Simplified Container Orchestration**: Docker Compose allows for the definition and management of multi-container applications as a single unit.
- **Reproducible Environments**: Since compose is defined in a YAML file, it's easy to deploy the same environment in different machine without missing any `docker run` command. This ensures that the application runs consistently across different machines.
- **Automate Volumes and Networking**: Docker Compose automatically creates a network for the application and assigns a unique DNS name to each service. No need to create networks and volumes. 

## Introducing YAML

YAML (YAML Ain't Markup Language) is a human-readable data format commonly used for configuration files and data exchange between systems.
YAMl is the cousin of JSON, by means that it uses indentation and key-value pairs to represent structured data, very similarly to JSON, by in a more clean syntax.

To prepare for working with Docker Compose, and later with Kubernetes, it is important to familiarize yourself with YAML syntax, which is the preferred syntax for writing Docker Compose files.

Find your favorite tutorial (there are hundreds YouTube videos and written tutorials) to see some YAML basic syntax (it takes no more than 20 minutes).

# Exercises

### :pencil2: Deploy the Netflix stack & Monitoring stack as a Docker Compose Project

Create a `docker-compose.yaml` file for the following:

![][docker_compose_netflix_n_monitoring2]

- Upon `docker compose up`, all service should be up and running. 
- Note containers that persis data in volumes.
- Note containers dependencies, e.g. the monitoring stack containers depend on the initializations of the netflix stack. 

[docker_compose_netflix_n_monitoring2]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/docker_compose_netflix_n_monitoring2.png


