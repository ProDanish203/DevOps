# Docker Notes

## Main points

- Image -> Container -> Volume
- Maintains Isolation
- Docker Network -> Multiple containers
- Docker Compose -> Automates the process by using .yaml file

## DOCKER Workflow

1. Docker Client -> User interface (CLI, GUI)
2. Docker Host (Docker Daemon) -> Creates and manages containers -> Builds Images
3. Docker Registry (Docker Hub) -> Similar to GitHub

## Docker Keywords

1. `FROM image[:tag] [AS name]` -> Specifies base Image
2. `WORKDIR /path/to/workdir` -> Sets the working directory
3. `COPY [--chown=<user>:<group>] <src>... <dest>` -> Copies files and directories from the build context to the image
4. `RUN <command>` -> Executes command in the shell during image build
5. `EXPOSE <port> [<port>/<protocol>]` -> Informs Docker that the container will listen on specified network ports at runtime
6. `ENV KEY=VALUE` -> Sets environment variables
7. `ARG <name>[=<default value>]` -> Defines build time variable
8. `VOLUME ["/data"]` -> Creates a mount point for externally mounted volumes
9. `CMD ["executable", "param1", "param2"]` -> Provides default commands to execute when the container starts
10. `ENTRYPOINT` != `CMD` -> Cannot be overridden

## Docker Commands

Here are some useful Docker commands for working with this project:

## Dockerfile Example

```dockerfile
FROM node:latest
WORKDIR /app
COPY package*.json .
COPY . /app
RUN npm run dev
EXPOSE 3000
ENV NODE_ENV=production
ARG NODE_VERSION=20
VOLUME /myvol
CMD ["npm", "start"]
```

### Build the Docker image

```bash
docker build -t [image_name] .
```

### View Docker images

```bash
docker images
```

### Run the Docker container with Port Mapping

```bash
docker run -p 5173:5173 [image_name]
```

### Get all running containers

```bash
docker ps
```

### Get all containers

```bash
docker ps -a
```

### Stop a running container

```bash
docker stop [container_id/name]
```

### Get rid of all running container

```bash
docker container prune
```

### Remove a specific container

```bash
docker rm [container_id/name] --force
```

### Delete an image
```bash
docker rmi [image_id] --force
```

### Run with hot reloading

```bash
docker run -p 5173:5173 -v "$(pwd):/app" -v /app/node_modules [image_name]
```

### Other commands to run the image

```bash
docker start [container_id]
docker restart [container_id]
docker pause [container_id]
docker unpause [container_id]
```

## Publish a Docker Image

```bash
docker login
docker tag [image_name] docker_username/image_name
docker push docker_username/image_name
```

## Docker Compose

## compose.yaml file structure

```bash
services:
  web:
    build:
      context: .
    ports:
      - 5173:5173
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - CHOKIDAR_USEPOLLING=true
    command: npm run dev
    develop:
      watch:
        - action: sync
          path: .
          target: /app
          ignore:
            - node_modules/
        - action: rebuild
          path: package.json
```

### Start the containers in watch mode:

```bash
docker compose up --watch
```

## Docker Compose for a MERN project

```bash
# specify the version of docker-compose
version: "3.8"

# define the services/containers to be run
services:
  # define the frontend service
  # we can use any name for the service. A standard naming convention is to use "web" for the frontend
  web:
    # we use depends_on to specify that service depends on another service
    # in this case, we specify that the web depends on the api service
    # this means that the api service will be started before the web service
    depends_on:
      - api
    # specify the build context for the web service
    # this is the directory where the Dockerfile for the web service is located
    build: ./frontend
    # specify the ports to expose for the web service
    # the first number is the port on the host machine
    # the second number is the port inside the container
    ports:
      - 5173:5173
    # specify the environment variables for the web service
    # these environment variables will be available inside the container
    environment:
      VITE_API_URL: http://localhost:8000

    # this is for docker compose watch mode
    # anything mentioned under develop will be watched for changes by docker compose watch and it will perform the action mentioned
    develop:
      # we specify the files to watch for changes
      watch:
        # it'll watch for changes in package.json and package-lock.json and rebuild the container if there are any changes
        - path: ./frontend/package.json
          action: rebuild
        - path: ./frontend/package-lock.json
          action: rebuild
        # it'll watch for changes in the frontend directory and sync the changes with the container real time
        - path: ./frontend
          target: /app
          action: sync

  # define the api service/container
  api:
    # api service depends on the db service so the db service will be started before the api service
    depends_on:
      - db

    # specify the build context for the api service
    build: ./backend

    # specify the ports to expose for the api service
    # the first number is the port on the host machine
    # the second number is the port inside the container
    ports:
      - 8000:8000

    # specify environment variables for the api service
    # for demo purposes, we're using a local mongodb instance
    environment:
      DB_URL: mongodb://db/anime

    # establish docker compose watch mode for the api service
    develop:
      # specify the files to watch for changes
      watch:
        # it'll watch for changes in package.json and package-lock.json and rebuild the container and image if there are any changes
        - path: ./backend/package.json
          action: rebuild
        - path: ./backend/package-lock.json
          action: rebuild

        # it'll watch for changes in the backend directory and sync the changes with the container real time
        - path: ./backend
          target: /app
          action: sync

  # define the db service
  db:
    # specify the image to use for the db service from docker hub. If we have a custom image, we can specify that in this format
    # In the above two services, we're using the build context to build the image for the service from the Dockerfile so we specify the image as "build: ./frontend" or "build: ./backend".
    # but for the db service, we're using the image from docker hub so we specify the image as "image: mongo:latest"
    # you can find the image name and tag for mongodb from docker hub here: https://hub.docker.com/_/mongo
    image: mongo:latest

    # specify the ports to expose for the db service
    # generally, we do this in api service using mongodb atlas. But for demo purposes, we're using a local mongodb instance
    # usually, mongodb runs on port 27017. So we're exposing the port 27017 on the host machine and mapping it to the port 27017 inside the container
    ports:
      - 27017:27017

    # specify the volumes to mount for the db service
    # we're mounting the volume named "anime" inside the container at /data/db directory
    # this is done so that the data inside the mongodb container is persisted even if the container is stopped
    volumes:
      - anime:/data/db

# define the volumes to be used by the services
volumes:
  anime:
```


## To specify the file to build the docker image
```bash
docker build -f Dockerfile.dev -t [image_name]:[tag] .
```

## To specify the file to compose the docker container
```bash
docker compose -f compose.prod.yaml up --build
```

## To compose a specific service
```bash
docker compose up [service_name] --build
```