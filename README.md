# Assignment 1 - Agile Software Practice

1. ## Step One
    I refactored slightly the `docker-compse.yml` file from my [docker-profile-app](https://github.com/ConorCoker/docker-profile-app/blob/master/compose.yaml) lab to use here.

2. ## Step Two
    ### Network Configuration
    - **Network Isolation**: Configured two Docker networks (`api-cache-network-assign-one` and `web-app-db-network-assign-one`) to control communication between services, enhancing security by restricting direct access to certain services.
    - **Service Connectivity**: Placed the `movies-api` and `caching` (Redis) services on the same network (`api-cache-network-assign-one`), allowing them to communicate without exposing Redis to other networks.

    ### Redis Connection Issue
    - **Redis URI Resolution**: Encountered a `Connection refused (os error 111)` issue with Redis. After troubleshooting, including reviewing [Redis Quick Start Guide](https://redis.io/learn/howtos/quick-start) and [DragonflyDB's Redis Connection Error Solutions](https://www.dragonflydb.io/error-solutions/redis-connection-error-111), I resolved the problem by using the Redis container name (`caching`) in the `REDIS_URI` environment variable. I also included the default Redis port (`6379`) as part of the URI. While it may not be strictly necessary, it’s included for now as the configuration works correctly.

3. ## Step Three 
    ### Fix Isolation Oversight 
    - **Removing Movies API from `web-app-db-network-assign-one` network**: The assignment spec mentions
    that the Express web app should not be accessible from the Movies API container. Since the express web app
    container lives in the `web-app-db-network-assign-one` network, it was incorrect for me to have the `Movies API container` also live inside the `web-app-db-network-assign-one` network so I fixed it within this commit.

    ### Fix error messages coming from MongoDB and Express

    - **Researching a solution**: When running `docker-compose up`, I encountered error messages related to database connectivity issues, such as:
    - `Could not connect to database using connectionString` (from `mongo-express`)
    - `/docker-entrypoint.sh: connect: Connection refused` (from MongoDB)

    - **Solution for MongoDB connection issue**: After finding [this GitHub issue](https://github.com/mongo-express/mongo-express/issues/437), I learned that changing the container name to `mongo` was needed. This is because `mongo-express` now expects the hostname `mongo` to resolve to the MongoDB service, so renaming the MongoDB container from `mongoDB` to `mongo` resolved this part of the issue.

    - **Solution for connection string error**: To address the `Could not connect to database using connectionString` error, switching to `ME_CONFIG_MONGODB_URL` fixed it. Recent versions of `mongo-express` require a full connection string rather than just the server name. I found this solution through [this GitHub discussion](https://github.com/mongo-express/mongo-express-docker/issues/67).

    ### Implementing Database Seeding for MongoDB

    - **Setting up database seeding**: To populate the MongoDB, I created a container, `fill-mongo`, which seeds the database with the provided JSON movie data saved on my PC. This container's sole purpose is to connect to the MongoDB container at `mongodb://${MONGODB_USERNAME}:${MONGODB_PASSWORD}@database:27017`, and mount the JSON file `seeding.json` on my PC to a file `movies_via_seeding.json` on the DB container.

    - **Seeding process**: Following the usual way to import data to MongoDB that I learned throughout college and the official documentation at [MongoDB mongoimport documentation](https://www.mongodb.com/docs/database-tools/mongoimport/), I used `command` to run a command on the DB container which I learned about through [this Stack Overflow answer](https://stackoverflow.com/a/64372237), It was here where I ran the `mongoimport` with the necessary flags:
    - `--authenticationDatabase=admin`: Tells that the user credentials for DB are stored in the admin database.
    - `--db moviesDB` and `--collection titles`: The new DB name i chose and the collection name.
    - `--drop`: This replaces the DB and creates a new one if it is already present.
    - `--file /movies_via_seeding.json --jsonArray`: Loads JSON array data from the shared file received from my PC to the database.

    - **Solution for connection string error**: To address the `Could not connect to database using connectionString` error, switching to `ME_CONFIG_MONGODB_URL` fixed it. Recent versions of `mongo-express` require a full connection string (including username, password, and port) rather than just the server name. This solution was confirmed through [this helpful GitHub discussion](https://github.com/mongo-express/mongo-express-docker/issues/67).
