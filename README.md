# Assignment 1 - Agile Software Practice

Link to [demo](https://www.youtube.com/watch?v=qLD32xmPmOM&ab_channel=ConorCoker)

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

4. ## Step Four
    ### Network Configuration (Isolation Adjustments)
    - **Network Isolation**: After re-reading the assignment spec, I confirmed that the Movies API container **is allowed** to share a network with the MongoDB container. To achieve the necessary isolation while enabling this connection, I created a new network, `express-network-assign-one`, which only the `mongo-express` and `database` containers connect to. This setup ensures that:
     - The Movies API can communicate with MongoDB by adding `api` to `web-app-db-network-assign-one`.
     - The Express web app (Mongo Express) remains **inaccessible from both the Redis and Movies API containers**.
     - The Redis container has no access to MongoDB, preserving required isolation.

    This configuration allows the Movies API to retrieve data from MongoDB at `http://localhost:9000/movies` while maintaining all specified access restrictions.

    ### Movies API retrieving the movies
    - **Database and Collection Naming**: After SSH-ing into the `movies-api` container, I discovered that it expects the MongoDB database name to be `tmdb_movies` with a collection name of `movie`. To ensure compatibility, I updated the MongoDB seeding configuration accordingly, so the API can query the database correctly. This change allows movies to display as expected in the browser at `http://localhost:9000/movies`

5. ## Step Five
    ### Setting up a developer profile
    - **Making mongo express boot up only for devs** I have added `profiles: - developer` to my `mongo-express` container, this means that we must add the flag `docker-compose --profile developer up` to have the `mongo-express` boot up. The idea behind this is that in a production enviroment we may not need to waste resources on `mongo-express` so we add a flag for when we need it. 
