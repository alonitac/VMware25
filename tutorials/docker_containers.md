# Containers 

## Running your first container

Run the [`exit0academy/netflix-frontend:0.0.1`](https://hub.docker.com/r/exit0academy/netflix-frontend) container by: 

```bash 
docker pull exit0academy/netflix-frontend:0.0.1
docker run exit0academy/netflix-frontend:0.0.1
```

When you run this command, the following happens (assuming you are using the default DockerHub registry configuration):

1. Docker pulls the `exit0academy/netflix-frontend` image, tag version `0.0.1`, from DockerHub. 
2. Docker creates a new container from the `exit0academy/netflix-frontend` image. The `exit0academy/netflix-frontend` image is a ready-to-run container image that encapsulates the [NetflixFrontend][NetflixFrontend] app, along with its dependencies and configuration. 
3. Docker allocates a dedicated read-write filesystem to the container (which is completely different and isolated from the host machine fs). This allows a running container to create or modify files and directories in its local filesystem.
4. Docker creates a **virtual network interface** to connect the container to the network. This includes assigning an IP address to the container. By default, containers can connect to external networks using the host machine's network connection.
5. Docker starts the container.
6. When you type `CTRL+c` the container stops but is not removed. You can start it again or remove it. When a container is removed, its file system is deleted. 

## Container management and lifecycle

To see your **running** containers, type:

```bash
docker ps 
```

or add  `-a` flag to list also stopped containers:

```console
$ docker ps -a
CONTAINER ID   IMAGE                COMMAND                  CREATED              STATUS                      PORTS     NAMES
d841a2fe07f9   netflix-frontend     "/docker-entrypoint..."  About a minute ago   Exited (0) 14 seconds ago             funny_blackburn
```

In the above output: 

- `d841a2fe07f9` is the **container ID** - a unique identifier assigned to each running container in Docker.
- `/docker-entrypoint...` is the (beginning) of the actual linux command that has run to initiate the process of the container. 
- `funny_blackburn` is a random alphabetical name that docker assigned to the container. 


#### 🧐 Try it yourself

Pull and run the container [`hello-world`](https://hub.docker.com/_/hello-world).

1. What is the status of the container after some moments of running?
2. Use `docker images hello-world` to get some information about the image from which the container has run. What is the image size?
3. What is the command used to launch the container `hello-world`? 


### Published ports

By default, when you run a container using the `docker run` command, the container doesn't expose any of its ports to the outside world.
To make a port available to services outside of Docker, or to Docker containers running on a different network, use the `--publish` or `-p` flag. 
This creates a firewall rule in the container, mapping a container port to a port on the host machine to the outside world.

Here's an example:

```bash
docker run --name netflix-frontend-1 -p 3000:3000 exit0academy/netflix-frontend:0.0.1
```

`-p 3000:3000` maps port 3000 **in the host machine** to port 3000 **within the container**. 

You can then access the netflix-frontend web app by opening a web browser and navigating to `http://localhost:3000`.

### Set environment variables for containers

When running a container using the `docker run` command, you can specify environment variables using the `-e` or `--env` flag.
For example:

```bash
docker run --name netflix-frontend-2 -e MOVIE_CATALOG_SERVICE=http://localhost:8080 exit0academy/netflix-frontend:0.0.1
```

### Running containers in the background 

When running containers with Docker, you have the option to run them in the background, also known as **detached mode**. This allows containers to run independently of your current terminal session, freeing up your terminal for other tasks.

To run a container in the background, you can use the `-d` or `--detach` flag with the docker run command. 

Let's run another netflix-frontend container: 

```console
$ docker run -d --name netflix-frontend-3 -e MOVIE_CATALOG_SERVICE=http://localhost:8080 exit0academy/netflix-frontend:0.0.1
310f1c48e402648ce4db41817dd76027d4528e481b25e985296fccc83421ddcb
```

When a container is running in the background, Docker assigns a unique container ID and displays it as output. You can use this ID to reference and manage the container later.

To view the list of running containers, you can use the `docker ps` command.
This command lists all the running containers along with their respective container IDs, names, and other information.

Since `netflix-frontend-3` is running in the background, the `docker logs` command can help you to view the logs generated by a running Docker container.
It allows you to retrieve and display the standard output (stdout) and standard error (stderr) logs generated by the container's processes.


```console
$ docker logs netflix-frontend-3

> netflix-clone@0.1.0 start /app
> next start

ready - started server on 0.0.0.0:3000, url: http://localhost:3000
```

If you want a real-time view, add the `-f` (`--follow`) flag. 

```console
$ docker logs netflix-frontend-3 -f
...
```

### Killing, stopping, starting and removing containers 

The `docker stop` command is used to stop one or more running containers in Docker.
It **gracefully stops** the containers by sending a `SIGTERM` signal to the main process running inside each container and then waits for a specified timeout (default is 10 seconds) before forcefully terminating them with a `SIGKILL` signal if needed.

```bash
docker container stop netflix-frontend-3
```

The `docker kill` command is used to forcefully terminate one or more running containers in Docker. It immediately sends a `SIGKILL` signal to the main process running inside each container, causing them to stop abruptly without any graceful shutdown.

Stopped or killed containers can be started using the `docker start` command, which resumes their execution from the point where they were stopped or killed. 
The container retains its configuration and any changes made inside the container's file system prior to stopping.


The `docker rm` command is used to remove one or more stopped or killed containers. 
It allows you to delete containers that are no longer needed, freeing up disk space and cleaning up resources.
It's important to note that the containers you wish to remove must be in a stopped state. If you attempt to remove a running container, you will encounter an error. 

```bash
docker stop netflix-frontend-3
docker rm netflix-frontend-3
docker ps -a 
```

> [!NOTE]
> The state of a container (stopped or killed) does not affect the Docker images associated with it. 
> Docker images remain unchanged and can be used to create and start new containers as needed.

> [!TIP]
> The `--rm` flag in the `docker run` command is used to automatically remove the container when it exits or stops running. It can be handy when you don't want to retain the container after it has served its purpose.
> 
> ```bash
> docker run --rm netflix-frontend-3
> ```


### Interacting with a running containers

You can interact with a **running** containers using the `docker exec` command.

The `docker exec` command allows you to execute a command inside a running container. Here's the basic syntax:

```bash 
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

Let's see it in action...

Start a new `netflix-frontend` container and keep it running. Give it a meaningful name instead the one Docker generates: 

```bash 
docker run --name netflix-frontend-4 exit0academy/netflix-frontend:0.0.1
```

Make sure the container is up and running. Since the running container occupying your current terminal session, open up another terminal session and execute:

```console
$ docker ps
CONTAINER ID   IMAGE                COMMAND                  CREATED              STATUS              PORTS       NAMES
89cf04f27c04   netflix-frontend     "/docker-entrypoint.…"   About a minute ago   Up About a minute   3000/tcp    netflix-frontend-4
```

Now say we want to debug the running `netflix-frontend-4` container, and perform some maintenance tasks, or executing specific commands within the containerized environment, we can achieve it by:

```bash 
docker exec -it netflix-frontend-4 /bin/bash
```

The `-it` flag is a combination of two flags: `-i` keeps STDIN open for the container, and `-t` allocates a pseudo-TTY to allow interaction with the container's terminal.

And you're in... You can execute any command you want within the running `netflix-frontend-4` container. 

**Tip**: if you don't know the container name, you can `exec` a command also using the container id:

```bash 
docker exec -it 89cf04f27c04 /bin/bash
```

#### 🧐 Try it yourself - Playing with the running container

In your open NetflixFrontend container terminal session:

1. What is the current user?
2. What is the hostname? 
3. Is the container connected to the internet? Can you ping `google.com`? Oh, don't have the `ping` command? Install it inside the container!
4. What is the user's home directory? 
5. How many processes are running in the container? What could that indicate? 
6. Do you have `docker` installed in the container? 


### Mounting directories from the host machine into containers

The `-v` flag allows you to mount directories from your host machine into a running container.
This is useful for dynamically updating files, sharing configurations, or debugging without rebuilding the container.

Imagine you need to update the `config/` dir inside the netflix-frontend container.
Instead of rebuilding the image again (either of you use an existed image, or your own built image), you can mount the directory dynamically:

```bash
docker run -v $(pwd)/local-config/:/app/config/ --name netflix-frontend-5 -p 3001:3000 exit0academy/netflix-frontend:0.0.1
```

Whenever the app will access the `/app/config/` dir, **from within the container**, it actually sees `$(pwd)/local-config/` from the **host machine**, outside the container. 

> [!NOTE]
> You can mount both files or directories. 

### Inspecting a container 

The `docker inspect` command is used to retrieve detailed information about Docker objects such as containers, images, networks, and volumes. It provides a comprehensive JSON representation of the specified object, including its configuration, network settings, mounted volumes, and more.

The basic syntax for the docker inspect command is:

```bash 
docker inspect [OPTIONS] OBJECT
```

Where `OBJECT` represents the name or ID of the Docker object you want to inspect.


Inspect your running container by:

```console
$ docker inspect netflix-frontend-4
....
```

# Exercises

### :pencil2: Running the Netflix service

Run both the [NetflixFrontend](https://hub.docker.com/r/exit0academy/netflix-frontend) and [NetflixMovieCatalog](https://hub.docker.com/r/exit0academy/netflix-movie-catalog) containers to make the service up and running:

![docker_netflix_simple][docker_netflix_simple]


### :pencil2: Collect and visualize metrics form the NetflixMovieCatalog

In this exercise, you will use containers to build a monitoring solution for your NetflixMovieCatalog:

- A [Prometheus](https://prometheus.io/) container, which collects and stores metrics from the NetflixMovieCatalog app.
- A [Grafana](https://grafana.com/) container, which visualizes the metrics. 

> [!NOTE]
> You should have a running instance of the NetflixMovieCatalog app. 

We'll use `docker run` commands to launch each container and guide you through accessing the system.

The goal is to monitor how many times the home page of the NetflixMovieCatalog (`/`) was called, and visualize the results in Grafana.


Prometheus is a system that collects metrics from various services and stores them in a time-series database.

Your NetflixMovieCatalog app is already configured to expose metrics in a format that prometheus can collect. 
Visit the `http://<movie-catalog-container-ip>:8080/metrics` endpoint in your NetflixMovieCatalog app to watch the metrics data. 

In order to configure prometheus to scrape the exposed metrics, you have to create a file named `prometheus.yml`:

```yaml
scrape_configs:
  - job_name: 'movie-catalog'
    scrape_interval: 5s
    static_configs:
      - targets: ['<movie-catalog-container-ip>:8080']
```

While changing `<movie-catalog-container-ip>` to the IP of the container. 

As can be seen, we'll configure Prometheus to scrape 1 target named `movie-catalog`, every 5 seconds. 

> [!NOTE]
> Why **can't** we use `http://localhost:8080` as the address of your NetflixMovieCatalog app, even though the address works from the host machine?

Let's run the [`prom/prometheus`](https://hub.docker.com/r/prom/prometheus) container by executing the following command **from the same directory where your `prometheus.yml` was created on your host machine**:

```bash
docker run -d -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml --name prometheus -p 9090:9090  prom/prometheus
```

- `-p 9090:9090`: Maps port `9090` on the container (Prometheus web UI) to port 9090 on your host.
- `-v`: Mounts your prometheus configuration file (`$(pwd)/prometheus.yml`) into the container, where prometheus expect to find it (`/etc/prometheus/prometheus.yml`).

Great. Now prometheus collects and stores the metrics. 

Next, run Grafana, which will visualize the metrics stored in Prometheus.

```bash
docker run -d --name grafana -p 3000:3000 grafana/grafana
```

Open your browser and visit http://localhost:3000. The default username and password are both `admin`.

- Set up Prometheus data source in Grafana:
  - Log into Grafana.
  - On the left panel, click **Connections** → **Data sources** → **Add data source**.
  - Select **Prometheus** and enter the URL: `http://<prometheus-container-ip>:9090`.
  - Click **Save & Test**.

Great. Now you can visualize many useful information collected by prometheus. 
For example, let's create a graph - **requests per minute**:
  - On the left panel enter the **Explore** panel:
  - In metric, choose `flask_http_request_total`.
  - Click on the **+ Operations**, then under **Range function**, choose **Rate**. 

You should see a graph "requests per second" to the NetflixMovieCatalog app (generate some traffic to see something...). 

### :pencil2: Communication between containers and the internet

Run two [`ubuntu`](https://hub.docker.com/_/ubuntu) containers named `ubuntu1` and `ubuntu2`.
Use the `-it` flags to make an interactive interaction with the running containers using a tty terminal.

Your goal is to be able to successfully `ping` the `ubuntu1` container from `ubuntu2`, i.e. to verify communication between the two containers.
Install ping if needed, `inspect` the containers to discover their IP addresses.



[docker_grafana-ts]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/docker_grafana-ts.png
[NetflixMovieCatalog]: https://github.com/exit-zero-academy/NetflixMovieCatalog
[NetflixFrontend]: https://github.com/exit-zero-academy/NetflixFrontend
[docker_netflix_simple]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/docker_netflix_simple.png