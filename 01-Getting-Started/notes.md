# Getting Started

## Providers

- Providers are service providers such as Docker, Kubernetes etc. which provide the backend services.
- Once connected to traefik, Traefik automatically listens to these providers for any changes and automatically updates its routes to handle the change.
- Note that even a simple configuration yaml file which tells traefik what to do - could be a provider as well!

## Simple Whoami in our localhost:

- Lets start with a simple whoami in our localhost. We will be using docker compose for this.

### 1. Starting the Traefik Dashboard

- We will link traefik to docker so that traefik can listen to new services booting in our docker. To do so, we will have to explicitly tell traefik the provider using the `--providers.docker` command.
- We can also enable logging and dashboard of traefik using the command `--log.level=INFO`.
- Finally, we are only focused on HTTP for the time being, so we will be using the `--api.insecure=true` command as well.
- The docker service for traefik will look something like this:
  
  ```yaml
  version: '3'
  services:
  traefik:
    # The latest version of Traefik
    image: traefik:latest
    # Enables the Dashboard web UI and tells Traefik to listen to docker
    command: --api.insecure=true --providers.docker --log.level=INFO
    # Expose the HTTP and the 8080 port for the Dashboard Web UI
    ports:
      # Exposing HTTP Port for incoming web request to whoami service
      - "80:80"
      # Exposing 8080 Port for Traefik Dashboard Web UI
      - "8080:8080"
    volumes:
      # So that Traefik can listen to the Docker events
      - /var/run/docker.sock:/var/run/docker.sock
      # Traefik configuration file
      - ./traefik.toml:/etc/traefik/traefik.toml
  ```
- Did you notice the volumes? Well, the first volume is a must as it is needed for traefik to listen to docker provider.
- The second one is for the configuration file. If we have a configuration file as a provider for traefik, we would like to give it to traefik as such.

### 2. Starting the Whoami Service:

- Now, let's link the whoami service to traefik. This can be done with the help of labels.

- Now, docker allows you to label your containers. These labels can be read by traefik which allows us to tell traefik how to configure the service.

- Some basic labels are as follows:
  
  - `traefik.enable=true` - This tells traefik to look into this docker
  - `traefik.http.routers.whoami.rule=Host('whoami.docker.localhost')` - This tells traefik that any **HTTP URL** hit done by clients with the URL: http://whoami.docker.localhost must go to this service.
  - `traefik.http.routers.whoami.entrypoints=http` - This tells traefik to use the HTTP port 80 as means to talk to this service. This effectively makes all HTTP routing to go to default port 80 for this service.
  - `traefik.http.services.whoami.loadbalancer.server.port=80` - This tells traefik to do loadbalancing on this service `whoami` for the port `80`.

- The final docker-compose for `whoami` service alone would look like this:

```yaml
  whoami:
    # Add an whoami service to test the routing
    # A container that exposes an API to show its IP address
    image: containous/whoami
    # Enable Traefik to listen to the whoami service by
    # adding the following labels
    labels:
      # Enable Traefik to listen to the whoami service
      - "traefik.enable=true"
      # Define the HTTP entry point for the whoami service
      - "traefik.http.routers.whoami.entrypoints=http"
      # Define the rule to route the incoming request to the whoami service
      - "traefik.http.routers.whoami.rule=Host(`whoami.docker.localhost`)"
      # Define the service for the whoami service
      - "traefik.http.services.whoami.loadbalancer.server.port=80"
```

- The service is running. We can check this at http://whoami.docker.localhost/ . 

![](C:\Users\JoelRaymann\AppData\Roaming\marktext\images\2023-03-06-10-33-39-image.png)

### Load balancing Whoami Services

![](C:\Users\JoelRaymann\AppData\Roaming\marktext\images\2023-03-06-10-34-38-image.png)

- Currently, we are going to scale the single `whoami` instance into 3 instances using the docker-compose command.

- The loadbalancing method used by default is using the `round-robin` method (Refer round robin in Operating Systems Books).

![](C:\Users\JoelRaymann\AppData\Roaming\marktext\images\2023-03-06-10-36-25-image.png)

- All we need to do for this is to just tell docker-compose to scale the `whoami` instance to 3 using the flag `--scale`. Traefik automatically detects this and accomodates accordingly.


