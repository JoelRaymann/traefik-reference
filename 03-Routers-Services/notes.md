# Routers & Services

![](C:\Users\JoelRaymann\AppData\Roaming\marktext\images\2023-03-06-13-47-10-image.png)

## Load Balancing & Routing

- How the incoming requests flow through Traefik is determined by the below components
  
  - **Providers** discover the services that live on your infrastructure (their IP, health, ..)
  
  - **Entrypoints** listen for incoming traffic (ports, ...)
  
  - **Routers** analyse the requests (host, path, headers, SSL, ...)
  
  - **Services** forward the request to your services (load balancing, ...)
  
  - **Middlewares** may update the request or make decisions based on the request (authentication, rate limiting, headers, ...)

## Routers - Connecting Requests to Services

#### Understanding Traefik Labels

- Let's take a look at how the Traefik labels are constructed

![](C:\Users\JoelRaymann\AppData\Roaming\marktext\images\2023-03-06-13-53-19-image.png)

### Router Rules

![](C:\Users\JoelRaymann\AppData\Roaming\marktext\images\2023-03-06-13-54-17-image.png)

- Note that when a new rule is created, Traefik automatically creates a corresponding service and router if not provided.

- The service automatically gets a server per instance of the container and the router automatically gets a rule defined by defaultRule (if no rule for it was defined in labels)
