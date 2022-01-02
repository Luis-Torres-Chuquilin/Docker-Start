# Docker-Started

Download and install [Docker Hub](https://www.docker.com/get-started) <br>

Follow the first instruction and instal de Container which containt the Tutorial <br>

Open the [Docker Tutoria](http://localhost/tutorial/) <br>

## The command you ran to Open the Tutorail: 

docker run -d -p 80:80 docker/getting-started <br>

<b>-d </b>- run the container in detached mode (in the background) <br>
<b>-p 80:80 </b> - map port 80 of the host to port 80 in the container <br>
docker/getting-started - the image to use <br>

You can combine single characters flags to shorten the full command as: <br>
docker run -dp 80:80 docker/getting-started

To see the all the Containers run docker ps -a in the CLI, by. default docker ps show the Containers running. For more informatin about docker ps checkout [here](https://docs.docker.com/engine/reference/commandline/ps/) or  [about other commands](https://docs.docker.com/engine/reference/commandline/docker/).



## Running your first application

You can download or clone the next repository (this belongs to docker).
I recommend to use VSCODE to Open it, because it is easy to use and you find a lot of tutorial on internet related to this [CODE EDITOR](https://medium.com/@rithmschool/intro-to-vs-code-7a4caadb7c70).


### Building and Running the App's Container Image

To build the application's Container Image, you need to create a Dockerfile. The Dockerfile is a script which contain the instrucctions used to create a container image. It is in the same folder as the package.json.

```
FROM node:12-alpine

#Adding build tools to make yarn install work on Apple silicon / arm64 machines

RUN apk add --no-cache python2 g++ make 
WORKDIR /app 
COPY . . 
RUN yarn install --production 
CMD ["node", "src/index.js"]
```

1. Run the next line in your CMD open from the directory with the Dockerefile: <br>
  This command used the Dockerfile to build a new container image. You might have noticed that a lot of "layers" were downloaded. This is because we instructed the builder that we wanted to start from the node:12-alpine image. But, since we didn't have that on our machine, that image needed to be downloaded.
```
docker build -t getting-started .
```
This command used the Dockerfile to build a new container image. You might have noticed that a lot of "layers" were downloaded. This is because we instructed the builder that we wanted to start from the node:12-alpine image. But, since we didn't have that on our machine, that image needed to be downloaded.

After the image was downloaded, we copied in our application and used yarn to install our application's dependencies. The CMD directive specifies the default command to run when starting a container from this image.

Finally, the -t flag tags our image. Think of this simply as a human-readable name for the final image. Since we named the image getting-started, we can refer to that image when we run a container.

The . at the end of the docker build command tells that Docker should look for the Dockerfile in the current directory.

2. Running our Apps's Container
```
docker run -dp 3000:3000 getting-started
```

Remember the -d and -p flags? We're running the new container in "detached" mode (in the background) and creating a mapping between the host's port 3000 to the container's port 3000. Without the port mapping, we wouldn't be able to access the application. <br>

You can open your application in [http://localhost:3000](http://localhost:3000).

#### Recap

1. First Create your Dockerfile and Build your App's  Container Image.
2. Run your  App's  Container Image.

## Persisting our DB
### Container Volumes

Volumes provide the ability to connect specific filesystem paths of the container back to the host machine. If a directory in the container is mounted, changes in that directory are also seen on the host machine. If we mount that same directory across container restarts, we'd see the same files.
<br>
There are two main types of volumes. We will eventually use both, but we will start with named volumes.


### Using Volume: <br>
Create a Volume manage by Docker:
```
docker volume create todo-db
```
Connect your App to the volume: <br>
By default, the todo app stores its data in a SQLite Database at /etc/todos/todo.db. 

We connect the volume we already created to the database path in our app
```
docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
```
You can check into the volume database:
```
docker volume inspect todo-db
[
    {
        "CreatedAt": "2019-09-26T02:18:36Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/todo-db/_data",
        "Name": "todo-db",
        "Options": {},
        "Scope": "local"
    }
]
```
To see your DATABASE run the following comand:

To opem MYSQL in our Volume Container:

```
docker exec -it <mysql-container-id> mysql -p
```
To see the DATABASES RUN:
```
SHOW DATABASES
```
To run a Specifict Database in our countainer,
```
docker exec -it <mysql-container-id> mysql -p todos
``

### Using Bind Mounts for Local development setup

With bind mounts, we control the exact mountpoint on the host. We can use this to persist data, but is often used to provide additional data into containers. When working on an application, we can use a bind mount to mount our source code into the container to let it see code changes, respond, and let us see the changes right away.

For Node-based applications, nodemon is a great tool to watch for file changes and then restart the application. There are equivalent tools in most other languages and frameworks.

```
docker run -dp 3000:3000 \
    -w /app -v "$(pwd):/app" \
    node:12-alpine \
    sh -c "apk add --no-cache python2 g++ make && yarn install && yarn run dev"
```

Now we going to change a line of code in our local, and it will be reflected in our Runing Container, afther the next change in the line 109 at src/static/js/app.js , refresh the page 
```
-                         {submitting ? 'Adding...' : 'Add Item'}
+                         {submitting ? 'Adding...' : 'Add'}
```

## Multi-Container Apps or Services

We run our App and our Databases in separate containers and we connect them, as it looks in the next picture.
![image](https://user-images.githubusercontent.com/67773071/147862227-5298bf63-e5a2-41d5-9c07-180eba3f83bd.png)

To work with this we will use Docker Compose.

```
version: "3.7"

services:
  app:
    image: node:12-alpine
    command: sh -c "apk add --no-cache python2 g++ make && yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos

  mysql:
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment: 
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```
### App Service:

-p 3000:3000  -->   ports: 
                      - 3000:3000 <br>
-w /app -->     working_dir: /app <br>

-v "$(pwd):/app") -->  volumes: 
                             - ./:/app <br>

```
used in: 

```
docker run -dp 3000:3000 \
  -w /app -v "$(pwd):/app" \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:12-alpine \
  sh -c "yarn install && yarn run dev"
  ```
  
 ### MySQL Service
 
 Defining the new service and name it mysql, it automatically gets the network alias.  
 
 Running the command (next one) created a volumen. It doesn't happend when we run the Compose, we have to specify the Volumen. We need to define the volume in the top-level volumes: section and then specify the mountpoint in the service config. By simply providing only the volume name, the default options are used
 
 ```
 docker run -d \
  --network todo-app --network-alias mysql \
  -v todo-mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=todos \
  mysql:5.7
  ```

### Running the Compose file:
```
docker-compose up -d
```


## Add and Connect a Database to your Application



