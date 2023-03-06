# Configurations

## Static vs. Dynamic Configurations

- Static configuration is referred to the startup configuration. These configurations are responsibler for setting the base configuration of traefik

- Dynamic configurations is just as the name suggests **DYNAMIC**. This Dynamic defines how all your requests are handled and hot-reloaded anytime there are changes.

## ## Static Configuration

There are three ways to set this one:

- In a configuration file

- In the command line arguments

- As a environment variables

NOTE: You are allowed to choose only one option from above. You CANNOT mix and match.

### 1. Config file

- Traefik looks for YAML or TOML file and reads it for static configuration file.

### 2. CLI args

- This is how we did in the previous `docker-compose` method. This method is preferred if you like to keep everything in the same file - in this case, it is in the same docker-compose file. An example of this would be:

![](C:\Users\JoelRaymann\AppData\Roaming\marktext\images\2023-03-06-10-59-03-image.png)

- We can see that those commands marked with red box are all the CLI commands.

### 3. Using Environment Variables

- This is same as CLI arguments except you will send it as environment variables to Docker containers. This is extremely useful if you wish to modify or control variables with Infrastructure as Code (IaC).

![](C:\Users\JoelRaymann\AppData\Roaming\marktext\images\2023-03-06-11-09-24-image.png)

## Static Configuration - File based

- File based can be done either with YAML or TOML file. Its entirely up to you. I will be using `YAML` configuration as it is less confusing than TOML. For example take a look at this toml file:

```toml
# traefik.toml
# ============== Traefik UI =================
[api]
dashboard = true
insecure = true

# ============== Enable Logging ==============
[log]
level = "INFO"


# ============== Set Providers ==============
# Set the providers
[providers]
# Configure the Docker provider
[providers.docker]
watch = true
exposedByDefault = true
swarmMode = false
```

- The above is a simple `traefik.toml` file. Let's go through this. Well, first, we are telling the Traefik to enable its dashboard with the `[api]` table section.

- Setting `dashboard=true` and `insecure=true` makes us call the dashboard with HTTP based calls. NOTE that `insecure=true` is a must for allowing insecure HTTP calls to work.

- Next, we are setting the logging level of Traefik by configuring the `[log]` table section.

- Finally, we are configuring the providers. In our case, we have one provider - docker. We are setting traefik to `watch` docker for new containers and also telling traefik to expose all containers to world by default. We are finally telling traefik that docker swarming is not enabled and don't worry about swarms.

Thats all. This file is pretty simple. Now, we need to mount this file into the `/etc/traefik/traefik.toml` inside the `traefik` container. This is done as follows:

```yaml
# Configure traefik to use file based configuration and to use the docker provider
version: '3'

services:
  traefik:
    # The latest official Traefik docker image
    # Enables the web UI and tells Traefik to listen to docker
    image: traefik:v2.9
    # Open HTTP port and web UI Port
    ports:
      # Expose HTTP port
      - "80:80"
      # Expose Traefik dashboard
      - "8080:8080"
    # Link to the docker socket so that Traefik can listen to the events
    # Also, link to the configuration file
    volumes:
      # So that Traefik can listen to the Docker events
      - /var/run/docker.sock:/var/run/docker.sock
      # Traefik configuration file
      - ./traefik.toml:/etc/traefik/traefik.toml
```

- Certainly, I am confused with the TOML file. How about YAML?

```yaml

# ============== TRAEFIK API ==================
api:
  dashboard: true
  insecure: true
# ============== ENABLE LOGGING ===============
log:
  level: "INFO"
# ============== SET PROVIDERS ================
# Set the providers
providers:
  # Configure the Docker provider
  docker:
    watch: true # Enable watch for traefik to automatically detect new containers
    exposedByDefault: true # We don't want to expose all containers by default
    swarmMode: false # We don't want to use swarm mode
# ============== SET ENTRYPOINTS =============
entryPoints:
  # Set the entrypoints
  web:
    # Configure the `web` entrypoint - NOTE: This `web` is a user defined name. It can be anything.
    address: ":80" # Set the address to listen on
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https

  websecure:
    # Configure the `websecure` entrypoint
    address: ":443"
```

- as we can see, it seems pretty readable. Its up to you on what to use.

## Port Detection

- If a container exposes a **Single** port, then Traefik uses it for internal comms.

- If a container exposes multiple or no ports, then a label should be defined mentioning the port to use.

## Entrypoints

- This tells traefik which URLs, addresses can ingress to our services. This can also route and rewrite and even add extra headers to the request. The following can be done:
  
  - Define which ports ot listen for incoming conns/packets
  
  - Redirect connections from HTTP to HTTPS
  
  - Forward Header Configuration
  
  - Override default TLS with user-defined TLS

- Now, this can be defined using TOML, YAML or even CLI. Example:

```yaml
# Traefik.yaml
# ============== TRAEFIK API ==================
api:
  dashboard: true
  insecure: true
# ============== ENABLE LOGGING ===============
log:
  level: "INFO"
# ============== SET PROVIDERS ================
# Set the providers
providers:
  # Configure the Docker provider
  docker:
    watch: true # Enable watch for traefik to automatically detect new containers
    exposedByDefault: true # We don't want to expose all containers by default
    swarmMode: false # We don't want to use swarm mode
# ============== SET ENTRYPOINTS =============
entryPoints:
  # Set the entrypoints
  web:
    # Configure the `web` entrypoint - NOTE: This `web` is a user defined name. It can be anything.
    address: ":80" # Set the address to listen on
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https

  websecure:
    # Configure the `websecure` entrypoint
    address: ":443"


```

The docker compose would be as follows:

```yaml
# Configure traefik to use file based configuration and to use the docker provider
version: '3'

services:
  traefik:
    # The latest official Traefik docker image
    # Enables the web UI and tells Traefik to listen to docker
    image: traefik:v2.9
    # Open HTTP port and web UI Port
    ports:
      # Expose HTTP port
      - "80:80"
      - "443:443"
      # Expose Traefik dashboard
      - "8080:8080"
    # Link to the docker socket so that Traefik can listen to the events
    # Also, link to the configuration file
    volumes:
      # So that Traefik can listen to the Docker events
      - /var/run/docker.sock:/var/run/docker.sock
      # Traefik configuration file
      - ./traefik.yaml:/etc/traefik/traefik.yaml

  catapp:
    # This is a test app that displays cats
    image: mikesir87/cats:1.0
    # We need to add the following labels to tell Traefik how to handle the service
    # ports:
    #   - "5000:5000"
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.catapp.rule=Host(`catapp.localhost`)"
      - "traefik.http.routers.catapp.entrypoints=web"
      - "traefik.http.services.catapp.loadbalancer.server.port=5000"


```

## Dynamic Configs - Routers, Middlewares & Services

- Dynamic configs are provided by the provider themselves.

![](C:\Users\JoelRaymann\AppData\Roaming\marktext\images\2023-03-06-13-41-06-image.png)

- We add the label to the container and we can config the container to have those configuration to the service.

- By attaching labels to a service/container, we can:
  
  - Define the routers
  
  - Define middlewares
  
  - Update config for the service

![](C:\Users\JoelRaymann\AppData\Roaming\marktext\images\2023-03-06-13-42-50-image.png)


