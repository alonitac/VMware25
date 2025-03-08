# Docker containers storage

## Docker data persistence

By default, all files created inside a container are stored on a writable container layer.
This means that the data doesn't persist when that container no longer exists.

Docker has two options for containers to store files on the host machine, so that the files are persisted even after the container stops: **volumes**, and **bind mounts**.

## Bind mounts 

**Bind mounts** provide a way to mount a directory or file **from the host machine into a container**.
Bind mounts directly map a directory or file on the host machine to a directory in the container. 

Let's take the `cassandra` image as an example:

```bash
docker run --rm -p 9042:9042 -v /path/on/host:/var/lib/cassandra --name my-cassandra cassandra:latest
```

In this example, we're running a Cassandra container while the `-v /path/on/host:/var/lib/cassandra` flag specifies the bind mount.
It maps the directory `/path/on/host` on the host machine to the `/var/lib/cassandra` directory inside the container.

Whenever the `my-cassandra` container is accessing `/var/lib/cassandra` path, the directory it actually sees is `/path/on/host` on the host machine.
More than that, every change the container does to the `/var/lib/cassandra` directory will reflect in the corresponding location on the host machine, `/path/on/host`, due to the bind mount.
The other way also applies, changes made on the host machine in `/path/on/host` will be visible inside the container under `/var/lib/cassandra`.

As you may know, `/var/lib/cassandra` is the location from which Cassandra store data.

Bind mounts are commonly used for development workflows, where file changes on the host are immediately reflected in the container without the need to rebuild or restart the container. 
They also allow for easy access to files on the host machine, making it convenient to provide configuration files, logs, or other resources to the container.

### 🧐 Try it yourself - Persist cassandra 

Use `cqlsh` (Cassandra Query Language Shell) to interact with Cassandra:

```bash
docker exec -it my-cassandra cqlsh
```

If successful, you should see the `cqlsh` prompt.
Inside `cqlsh`, run the following commands:

```sql
CREATE KEYSPACE IF NOT EXISTS my_keyspace 
WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};

USE my_keyspace;

CREATE TABLE movies (
    id UUID PRIMARY KEY,
    name text,
);
```

Insert a sample movie:

```sql
INSERT INTO movies (id, name) 
VALUES (uuid(), 'Alice in Wonderland');
```

Then try to retrieve data after you've killed the container and start a new one:

```sql
SELECT * FROM movies;
```

## Volumes 

**Docker volumes** is another way to persist data in containers. 
While bind mounts are dependent on the directory structure and OS of the host machine, volumes are logical space that completely managed by Docker.
Volumes offer a higher level of abstraction, allow us to work with different kind of storages, e.g. volumes stored on remote hosts or cloud providers.
Volumes can be shared among multiple containers.

### Create and manage volumes

Unlike a bind mount, you can create and manage volumes outside the scope of any container.

```bash 
docker volume create cassandra-data
```

Let's inspect the created volume:

```bash
$ docker volume inspect cassandra-data
[
    {
        "CreatedAt": "2023-05-10T14:28:15+03:00",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/cassandra-data/_data",
        "Name": "cassandra-data",
        "Options": {},
        "Scope": "local"
    }
]
```

The `local` volume driver is the default built-in driver that stores data on the local host machine.
But docker offers [many more](https://docs.docker.com/engine/extend/legacy_plugins/#volume-plugins) drivers that allow you to create different types of volumes that can be mapped to your container.

The following example mounts the volume `cassandra-data` into `/var/lib/cassandra` in the container.

```bash
docker run --rm -p 9042:9042 -v cassandra-data:/var/lib/cassandra --name my-cassandra cassandra
```

Essentially we've got the same effect as the above example, but now the mounted volume is not just a directory in the OS, 
but logically managed by docker. 

Can you see how elegant is it? volumes provide a seamless abstraction layer for persisting data within containers.
The container believe it reads and writes data from some location in his file system (`/var/lib/cassandra`), without any knowledge of the underlying storage details (of the `cassandra-data` volume). 

Remove a volume:

```bash
docker volume rm cassandra-data
```

### More about volumes

- A given volume can be mounted into multiple containers simultaneously. 
- When no running container is using a volume, the volume is still available to Docker and is not removed automatically.
- If you mount an **empty volume** into a directory in the container in which files or directories exist, these files or directories are **propagated (copied)** into the volume. 
- Similarly, if you start a container and specify a volume which does not already exist, an empty volume is created for you. This is a good way to pre-populate data that another container needs.
- If you mount a **bind mount** or **non-empty volume** into a directory in the container in which some files or directories exist, these files or directories are **obscured by the mount**.

# Exercises

### :pencil2: Persist Grafana and Prometheus data

Your goal is to run the Prometheus container from the [previous exercise](docker_containers.md#exercises) while the data is stored persistently.

Make sure the data is persistent by `docker rm` the container, then create a new container and see the metrics data that was collected from the previous one.  

### :pencil2: Understanding user file ownership in docker

When running a container, the default user inside the container is often set to the `root` user, and this user has full control of the container's processes.
Since containers are isolated process in general, we don't really care that `root` is the user operating within the container.

But, when mounting directories from the host machine using the `-v` command, it is important to be cautious when using the root user in a container. Why?

We will investigate this case in this exercise...

1. On your host machine, create a directory under `~/test_docker`.
2. Run the `ubuntu` container while mounting `/test` within the container, into `~/test_docker` in the host machine:

```bash
docker run -it -v ~/test_docker:/test ubuntu /bin/bash
```

3. From your host machine, create a file within `~/test_docker` directory.
4. From the `ubuntu` container, list the mounted directory (`/test`), can you see the file you've created from your host machine?
   Who are the UID (user ID) and GID (group ID) owning the file?
5. From within the container, create a file within `/test`.
6. List `~/test_docker` from your host machine. Who are user and group owning the file created from the container?
7. Try to indicate the potential vulnerability: "If an attacker gains control of the container, they may be able to..."
8. Repeat the above scenario, but instead of using `-v`, use docker volumes. Starts by creating a new volume by: `docker create volume testvol`. Describe how using docker managed volume can reduce the potential risk.


[docker_volumes]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/docker_volumes.png