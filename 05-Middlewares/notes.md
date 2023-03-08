# Middlewares

- **Middlewares connect to Routers** - One or more Middlewares can connect to one or more routers

- **Intercepts Requests** - The Middleware transforms requests before they are sent to a service.

![](notes-assets/2023-03-07-11-11-10-image.png)

## Types of Middlewares

- **Authentication**- Handles user/service authentication

- **Content Modifier** - Manages the served content

- **Middleware Tool** - Middleware Management

- **Path Modifier** - Configures the handling of the URL structure

- **Request Lifecycle** - Applies rules to service requests

- **Security**- Secures access to the service

## Some Middlewares

- BasicAuth - Add Basic Authentication to services

- Compress - Enables gzip compression on severed content

- ErrorPage - Display custom error pages for 400/500 Errors

- RateLimit - Configure the number of allowed requests to the service

- RedirectScheme - Rewrite HTTP Requests to HTTPS

## Middleware Deployment

1. Create Middleware - Define the middleware with a nice name

2. Router <-> Middleware - Connect  the router to the middleware with that nice name

![](notes-assets/2023-03-07-11-21-37-image.png)

![](notes-assets/2023-03-07-11-22-02-image.png)

---

## BasicAuth Middleware

- Restricts access to the service using basic user/password

![](notes-assets/2023-03-08-11-19-25-image.png)

### Declare and set up a hashed password

![](notes-assets/2023-03-08-11-20-21-image.png)

- We can generate a hashed value of password with the help of `htpasswd -nb user password | sed -e s/\\$/\\$\\$/g`

----

## Compress Middleware

- Compressing the Response before Sending it to the client using gzip compression

- This is a common practice to include compression to all web services as it helps in faster response and loading time.

![](notes-assets/2023-03-08-11-34-36-image.png)

![](notes-assets/2023-03-08-11-34-45-image.png)

---

## Error Page Middleware

- Serve custom Error pages for 4xx/5xx errors

![](notes-assets/2023-03-08-11-40-00-image.png)

![](notes-assets/2023-03-08-11-41-13-image.png)

## Rate Limiting Middleware

- Limits the rate of access to a docker service

![](notes-assets/2023-03-08-12-02-00-image.png)

- **Average** - Maximum rate, in requests/second for a given source

- **Period** - The time period
  
  - Rate = average / period

- **Sources** - Headers, IP's or Hosts

### RedirectScheme (HTTP -> HTTPS) Middleware

- This allows us to redirect traffic from HTTP to HTTPS

![](notes-assets/2023-03-08-12-16-47-image.png)
