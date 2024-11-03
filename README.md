# Assignment 1 - Agile Software Practice

1. ## Step One
    I refactored slightly the `docker-compse.yml` file from my [docker-profile-app](https://github.com/ConorCoker/docker-profile-app/blob/master/compose.yaml) lab to use here.

2. ## Step Two
    ### Network Configuration
    - **Network Isolation**: Configured two Docker networks (`api-cache-network-assign-one` and `web-app-db-network-assign-one`) to control communication between services, enhancing security by restricting direct access to certain services.
    - **Service Connectivity**: Placed the `movies-api` and `caching` (Redis) services on the same network (`api-cache-network-assign-one`), allowing them to communicate without exposing Redis to other networks.

    ### Redis Connection Issue
    - **Redis URI Resolution**: Encountered a `Connection refused (os error 111)` issue with Redis. After troubleshooting, including reviewing [Redis Quick Start Guide](https://redis.io/learn/howtos/quick-start) and [DragonflyDB's Redis Connection Error Solutions](https://www.dragonflydb.io/error-solutions/redis-connection-error-111), I resolved the problem by using the Redis container name (`caching`) in the `REDIS_URI` environment variable. I also included the default Redis port (`6379`) as part of the URI. While it may not be strictly necessary, it’s included for now as the configuration works correctly.