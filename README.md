
# Nginx Deployment with Traefik in Docker Swarm

### Deploy a simple Nginx server using Docker Swarm, integrated with Traefik as a reverse proxy. This setup allows you to serve your static web applications efficiently and securely.
- <u>**Nginx**</u>: A high-performance web server used to serve static content for your applications.


### External Dependencies:
- <u>**Traefik**</u>: A modern reverse proxy and load balancer designed for dynamic container environments, used to route external traffic to Docker Swarm services.


## Table of Contents

1. [Initial Setup](#initial-setup)
   - [Domains and Subdomains](#domains-and-subdomains)
   - [Setup Repository](#setup-repository)
   - [Copy Templates](#copy-templates)
   - [Edit Configuration](#edit-configuration)
     - [.env Configuration](#env-configuration)
2. [Deployment](#deployment)
   - [Deploy the Stack](#deploy-the-stack)
3. [Maintenance](#maintenance)
   - [Scaling Services](#scaling-services)
   - [Cache Management](#cache-management)
4. [Customizations](#customizations)
   - [VCard Integration](#vcard-integration)
   - [Serving Multiple Business Cards](#serving-multiple-business-cards)
5. [License](#license)

# Initial Setup

## Domains and Subdomains

Ensure that the domains and subdomains you plan to use are properly configured and point to the manager node of your Docker Swarm cluster.

Examples:

- `business.info.felicitas-wisdom.de`

This setup allows you to host multiple business cards or services under different subdomains.

## Setup Repository

Set up the repository at your desired location on the server:

```bash
# Choose location on server (glusterfs when using multiple nodes is recommended).
mkdir -p /gluster_storage/swarm/business-cards/<DOMAINNAME>
cd /gluster_storage/swarm/business-cards/<DOMAINNAME>
git clone https://github.com/Sokrates1989/swarm-nginx.git .
```

## Copy Templates

Copy the necessary template files to start your configuration:

```bash
# Copy ".env.template" to ".env"
cp .env.template .env

# Copy "config-stack.yml.template" to "config-stack.yml"
cp config-stack.yml.template config-stack.yml

# Copy Nginx configuration template if provided
cp nginx_conf/conf.d/default.conf.template nginx_conf/conf.d/default.conf
```

## Edit Configuration


### Cache Configuration

The following settings allow you to control the cache size and duration in Nginx. This can be useful in performance optimization by controlling how long and how much data is cached, reducing server load and speeding up response times.


#### Replacement Commands for Cache Configuration

Use the following commands to replace placeholders in the `default.conf` file with real values for cache size and duration.

```bash
# Replace the cache size placeholder with your desired value.
sed -i -e 's/NGINX_MAX_CACHE_SIZE_PLACEHOLDER/1g/g' ./nginx_conf/conf.d/default.conf

# Replace the cache duration placeholder with your desired value.
sed -i -e 's/NGINX_CACHE_DURATION_PLACEHOLDER/30m/g' ./nginx_conf/conf.d/default.conf
```


#### What values should I use?
- **NGINX_MAX_CACHE_SIZE_PLACEHOLDER**: Specifies the maximum size of the cache.
    - **Supported Units**: 
        - k: Kilobytes (1 k = 1024 bytes)
        - m: Megabytes (1 m = 1024 kilobytes)
        - g: Gigabytes (1 g = 1024 megabytes)
    - **Recommended Values**: 
        - Small Site: `500m`
        - Medium Site: `1g`
        - Large Site: `5g` or more based on traffic

- **NGINX_CACHE_DURATION_PLACEHOLDER**: Determines how long cached data is kept before it is considered stale.
    - **Supported Units**: 
        - ms: Milliseconds (1 ms = 0.001 seconds)
        - s: Seconds (1 s = 1000 ms)
        - m: Minutes (1 m = 60 seconds)
        - h: Hours (1 h = 60 minutes)
        - d: Days (1 d = 24 hours)
    - **Recommended Values**: 
        - Low Traffic Sites: `30m` (30 minutes)
        - Medium Traffic Sites: `1h` (1 hour)
        - High Traffic Sites: `6h` (6 hours)

#### Suggested Configurations Based on Traffic Scenarios

1. **Low Traffic Blog or Small Site**: 
    - Cache Duration: `15m`
    - Cache Size: `200m`
    - This setup is ideal for small blogs or websites with limited traffic, providing a balance between caching efficiency and storage usage.

2. **Medium Traffic Business Site**:
    - Cache Duration: `1h`
    - Cache Size: `1g`
    - Suitable for sites with moderate traffic where performance is important, but the cache size can still be limited.

3. **High Traffic E-commerce or News Site**:
    - Cache Duration: `6h`
    - Cache Size: `5g`
    - High-traffic sites can benefit from extended cache durations and larger cache sizes to handle heavy loads and frequent user requests.


### .env

Replace the default domain with the actual domain as setup in [Domains and Subdomains](#domains-and-subdomains) 
```bash
sed -i -e 's/default-domain.com/your-domain.com/g' ./.env

# Example for test.felicitas-wisdom.de.
sed -i -e 's/default-domain.com/test.felicitas-wisdom.de/g' ./.env
```

Edit the variables in the `.env` file:
```bash
vi .env
```

# Deployment

## Deploy the Stack

Use Docker Compose to deploy your stack to Docker Swarm:

```bash
docker stack deploy -c <(docker-compose -f config-stack.yml config) <STACK_NAME>
```


### Verify Deployment

Check the status of your services:

```bash
docker stack services <STACK_NAME>
```

## Cache Management

If your Nginx configuration includes caching and you need to purge the cache:

```bash
# Remove cached content
rm -rf ${DATA_ROOT}/nginx_cache/*
```

No need to reload Nginx; it will regenerate the cache as users access the site.

# Customizations

## VCard Integration

If your web application includes VCard downloads (e.g., for contact information), ensure that Nginx is configured to serve `.vcf` files correctly.

### Nginx Configuration

Add the following to your Nginx configuration file (`nginx_conf/nginx.conf` or appropriate site configuration):

```nginx
http {
    ...

    types {
        text/vcard vcf;
        # Include other MIME types
    }

    ...
}
```

This configuration sets the correct MIME type for VCard files, ensuring that browsers handle them properly.


# License

This project is licensed under the terms of the GNU License. See the [LICENSE](./LICENSE) file for details.
