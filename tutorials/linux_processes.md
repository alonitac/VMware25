# Linux Processes

## Process Definition

A process is a currently executing program (or command).
A program is a series of instructions that tell the computer what to do.
When we run a program, those instructions are copied into memory and space is allocated for variables and other stuff required to manage its execution.
This running instance of a program is called a **process**.

The top command can show running processes in the terminal in real-time.

```console
myuser@hostname:~$ top
...
```

A process consists of its **execution context** (memory), **I/O context** (open files, working directory), **environment variables**, **heritage info** (PID, parent/child processes), **credentials** (user/group), and **resource stats** (CPU, RAM usage).
Each running process is represented as a subdirectory in `/proc/<PID>`.

## Processes state

In linux, multiple users running multiple processes, at the same time and on the same system. The CPU manages all these processes simultaneously, according to the below state diagram:

![][linux_process_state]

A process can be in one of several states at any given time, indicating its current status or activity. These states are important to understand for managing and troubleshooting system performance.

Here are a short description of each state:

1. **Running**: A process is currently executing on a CPU core (a.k.a process CPU burst).
2. **Waiting**: A process is waiting for some external event, such as user input, disk, or a network I/O operation to complete.
3. **Ready**: Is often used to refer to a process that is waiting to be executed on a CPU. When a process is in the ready state, it is typically placed in a queue, waiting for an available CPU to execute on. Once the CPU becomes available, the process is moved from the ready state to the running state and starts executing. The amount of time a process spends in the ready state is dependent on the scheduling algorithm and the current system load.
4. **Zombie**: A process has completed its execution but its parent process has not yet collected its exit status.
5. **Initialized**: A process that has been created but has not yet been assigned a process identifier (PID)
6. **Terminated**: Indicates that a process has finished its execution and has exited.

## The process tree

A new process is created because an existing one makes an exact copy of itself (forking). This implies a tree structure of processes:

```console
myuser@hostname:~$ pstree
systemd─┬─ModemManager───2*[{ModemManager}]
        ├─NetworkManager───2*[{NetworkManager}]
        ├─2*[SACSrv───3*[{SACSrv}]]
        ├─accounts-daemon───2*[{accounts-daemon}]
        ├─acpid
        ├─atd
        ├─avahi-daemon───avahi-daemon
.
.
.
```

This child process has the same environment as its parent, only different process id (PID). When a process ends normally (it is not killed or otherwise unexpectedly interrupted), the program returns its **exit code** (a number) to the parent. Only exit status of 0 means success.

A process has a series of characteristics, which can be viewed with the `ps` command:

```console
myuser@hostname:~$ ps -aux
USER     	PID %CPU %MEM	VSZ   RSS TTY  	STAT START   TIME COMMAND
root       	1  0.0  0.1 172752  9440 ?    	Ss   13 Feb  7:21 /sbin/init splash
root       	2  0.0  0.0  	0 	0 ?    	S	13 Feb  0:00 [kthreadd]
```

The `ps` command can display a lot of information about processes, including their PID, state (STAT column), CPU and memory usage, the user owning this process, and the command that initiated the process.


## Signals 

Processes are communicating with each other and with the kernel using **signals**. There are multiple signals that you can send to a process. Use the `kill` command to send a signal to a process.

| Signal Name | Signal Number | Meaning                                                     |
|-------------|---------------|-------------------------------------------------------------|
| SIGTERM     | 15            | Terminate the process in orderly way                        |
| SIGINT      | 2             | Interrupt the process. The process can ignore this signal   |
| SIGKILL     | 9             | Interrupt the process. The process can't ignore this signal |
| SIGHUP      | 1             | The parent process was terminated                           |


Use `man 7 signal` for a comprehensive list of linux signals.

Let's kill a process by send it a KILL signal:

```console
myuser@hostname:~$ sleep 600
```

The above command initiates a process that just sleeps 600 seconds, and ends.

In **another terminal**, use the `ps` to get the PID of the process running the `sleep` command:

```console
myuser@hostname:~$ ps -a
PID TTY          TIME CMD
46214 pts/1    00:00:00 sleep
46445 pts/3    00:00:00 ps
...
myuser@hostname:~$ kill -9 46214
```

Observe how the process running the `sleep` command is terminated.

---

In bash, you can use keyboard shortcuts to send signals to process:

| Shortcut | Signal  | Meaning                               |
|----------|---------|---------------------------------------|
| CTRL+C   | SIGINT  | Terminate the process in orderly way  |
| CTRL+Z   | SIGSTOP | Suspend the process in the background |


## Services


In Linux, services are **background processes** that run continuously and provide specific functions to the operating system or other applications. 
Here are some common Linux services:

1. `ssh`: A secure remote login protocol that allows users to log in to a remote computer securely.
2. `cron`: A service that executes scheduled tasks or commands at specified intervals.
3. `firewalld`: A dynamic firewall management tool.

The `systemctl` command is used to manage services in your system:

```console
myuser@hostname:~$ sudo systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2025-02-20 05:07:09 EST; 11min ago
     Docs: man:firewalld(1)
 Main PID: 817 (firewalld)
    Tasks: 2 (limit: 23205)
   Memory: 52.2M
   CGroup: /system.slice/firewalld.service
           └─817 /usr/libexec/platform-python -s /usr/sbin/firewalld --nofork --nopid

Feb 20 05:07:08 localhost.localdomain systemd[1]: Starting firewalld - dynamic firewall daemon...
Feb 20 05:07:09 localhost.localdomain systemd[1]: Started firewalld - dynamic firewall daemon.
Feb 20 05:07:10 localhost.localdomain firewalld[817]: WARNING: AllowZoneDrifting is enabled. This is considered an insecure configuration opt>

myuser@hostname:~$ sudo systemctl stop firewalld
myuser@hostname:~$ sudo systemctl start firewalld
myuser@hostname:~$ sudo systemctl restart firewalld
```

Who manages Linux services?  
As can be seen in the output of `pstree`, **systemd** is the first process in many Linux distributions, which is a system and service manager that provides a way to manage and control system services. 
Systemd reads **unit files**, which are configuration files used by systemd to define system services.

Unit files can be found in the `/etc/systemd/system` directory.

List all your system services:

```console
myuser@hostname:~$ systemctl list-units --type=service
UNIT                                              LOAD   ACTIVE SUB     DESCRIPTION                
accounts-daemon.service                          loaded active running Accounts Service           
acpid.service                                    loaded active running ACPI event daemon          
alsa-restore.service                             loaded active exited  Save/Restore Sound Card
....
```

In the above output, UNIT represents the unit name, LOAD indicates that the unit's configuration has been read by systemd, ACTIVE is the state of the unit.

## Environment variables

Global variables or **environment variables** are variables available for any process or application running in the same environment. Global variables are being transferred from parent process to child program. They are used to store system-wide settings and configuration information, such as the current user's preferences, system paths, and language settings. Environment variables are an essential part of the Unix and Linux operating systems and are used extensively by command-line utilities and scripts.

The `env` or `printenv` commands can be used to display environment variables.

### The `$PATH` environment variable

When you want the system to execute a command, you almost never have to give the full path to that command. For example, we know that the `ls` command is actually an executable file, located in the `/bin` directory (check with `which ls`), yet we don't have to enter the command `/bin/ls` for the computer to list the content of the current directory.

The `$PATH` environment variable is a list of directories separated by colons (`:`) that the shell searches when you enter a command. When you enter a command in the shell, the shell looks for an executable file with that name in each directory listed in the `$PATH` variable, in order. If it finds an executable file with that name, it runs it.

System commands are normal programs that exist in compiled form (e.g. `ls`, `mkdir` etc... ).

```console
myuser@hostname:~$ which ls
/bin/ls
myuser@hostname:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:.....
```

The above example shows that `ls` is actually an executable file located under `/bin/ls`. The `/bin` path is part of the PATH env var, thus we are able to type `ls` shortly.

### The `export` command

The `export` command is used to set environment variables in the current shell session or to export variables to **child processes**.

When a variable is exported using the `export` command, it becomes available to any child process that is spawned by the current shell. This is useful when you need to pass environment variables to programs or scripts that you run.

For example, let's say you want to add a directory called `mytools` to your `PATH` environment variable so that you can run executables stored in that directory. You can do this by running the following command:

```bash
export PATH=$PATH:/home/myuser/mytools
```

This command adds the directory `/home/myuser/mytools` to the existing PATH environment variable, which is a colon-separated list of directories that the shell searches for executable files.

If you only set the `PATH` variable without exporting it, it will only be available in the current shell session and will not be inherited by child processes.

```bash
PATH=$PATH:/home/myuser/mytools
```

# Exercises

### :pencil2: Deploy the Netflix service

In this exercise you'll run a dummy "Netflix" service.
The service is composed by 2 **microservices**:

- [NetflixFrontend][NetflixFrontend] - The frontend UI app that serves client requests, written in Node.js.
- [NetflixMovieCatalog][NetflixMovieCatalog] - The backend API that contains metadata on movies. The frontend app fetches movies metadata from the catalog, written in Python.

1. We will use the `git` command to clone the app's source code from GitHub website to your machine.
   Use `dnf` to install `git`.
2. Let's clone the source code of the NetflixMovieCatalog app from GitHub by:

   ```bash
   git clone https://github.com/exit-zero-academy/NetflixMovieCatalog.git
   ```
   The command creates a directory named `NetflixMovieCatalog` with all code files.
   The main file is `app.py`. 

3. You can try running the app by executing `python3 app.py` from the `NetflixMovieCatalog` dir.
   But as you may see, there an error of missing some package, since our app is dependent in other Python packages. 
   
   `pip` is the package manager for Python. It allows us to install and manage additional Python libraries (or dependencies) that are required for our app to run.
   You have to install `pip` by: `dnf install python3-pip`.
4. Now use `pip` to install the app dependencies by `pip3 install -r requirements.txt`.
5. Launch the app by running the main file: `python3 app.py`.
   The app is listening on port `8080`. 
6. You have 2 options to access the app:
   - (Recommended) From the local machine, visit: http://ip-192-168-122-251.ec2.internal.exit-zero.click:8080 (while changing `192-168-122-251` to your VM IP).
   - From the lab browser, visit the app by: `http://<your-vm-ip>:8080`.


> [!TIP]
> Did you check that your VM is accessible via port 8080? What are you waiting for? 
> 
> ```bash
> sudo firewall-cmd --add-port=8080/tcp --permanent
> ```
> 
> Don't forget to restart the `firewalld` service to apply the new configuration. 


Let's repeat the same process for the **NetflixFrontend**.

1. The NetflixFrontend app is built with **Node.js**, version **> 18.0**, which is not installed on a fresh AlmaLinux.
   Use `dnf` to install `nodejs`. **Make sure you install version at least 18.0**. The `dnf module list nodejs` might help. 
2. Clone the app source code: 
   ```bash
   git clone https://github.com/exit-zero-academy/NetflixFrontend.git
   ```
3. **Carefully** follow the instructions under [NetflixFrontend][NetflixFrontend] to launch the app. 

> [!NOTE]
> For the Netflix service to be fully operational, both the **NetflixFrontend** and the **NetflixMovieCatalog** should be running
> simultaneously. You can connect to multiple SSH sessions for that. 


### :pencil2: Deploy the Netflix app as a service

In this exercise you'll run the Netflix service (Frontend and Catalog apps) as a Linux service,
so it will be running in the background whenever the VM is up. 

To remind you, to create a Linux service, you have to create a **unit file**. 

You should create the following unit files:

- `/etc/systemd/system/netflix-frontend.service` for the NetflixFrontend service.
- `/etc/systemd/system/netflix-catalog.service` for the NetflixMovieCatalog service. 

Here is an example for a `.service` file to the NetflixFrontend:

```text
[Unit]
Description=Netflix Frontend

# ensures the service starts only after the network is up and available
After=network.target

[Service]
# absolute path you the app workdir
WorkingDirectory=/root/NetflixFrontend

# the command that starts your app, e.g. `python3 app.py` or `node /usr/bin/npm start` 
ExecStart=node /usr/bin/npm start

Environment=MOVIE_CATALOG_SERVICE=http://localhost:8080

# define the service to restart in case of failure in the execution
Restart=always

[Install]
# define the service to start automatically when the system reaches the multi-user runlevel
WantedBy=multi-user.target
```

Create similar unit file for the NetflixMovieCatalog app, and make sure the service works properly. 

**Note:**

- You can start the services by: `systemctl start <service-name>` (change `<service-name>` to either `netflix-frontend` or `netflix-catalog`).
- Always check the status: `systemctl status <service-name>`.
- Enable the service to start on boot: `sudo systemctl enable <service-name>`.
- You can always check the service logs (if needed): `journalctl -u <service-name>`.


### :pencil2: Work with services in the ESXi level 

In an ESXi host, just like in Linux, there are system services running in the background that manage various aspects of the host.
These services include networking, authentication, SSH access, and more.

1. In the vSphere web client inventory page, choose the host on which your VM is running.
2. Choose the **Configure** tab, and under **System**, choose **Services**.
3. Indentify "famous" services:
   - Which service is responsible to detect host failure and restart VM on HA cluster?
   - Which service is responsible for the communication between the host to the vCenter server? 
4. Let's say you have to access your host via SSH to perform some low level investigation. Start the SSH service. 
5. Try to connect to your ESXi host via SSH, by: `ssh root@<host-ip>`.
6. Perform simple administrative tasks:
   - Get information about the vSAN status in the host: `esxcli vsan cluster get`.
   - List your running VMs: `esxcli vm process list`.
   - Exit the ssh session by: `exit`.


### :pencil2: Run processes in the background terminal session

"Interactive processes" are processes that are initialized and controlled through a terminal session.
These processes can run in the foreground, occupying the terminal. 
Alternatively, they can run in the background. The terminal can accept new commands while the program is running.

1. Open a nes terminal session and execute `sleep 600` which initiates a process that "sleeps" 10 minutes and ends.
2. Stop the process and send into the background by `CTRL+Z`.
3. Bring the process to the foreground by `fg`.
4. Now run the same command while sending the process to the background by the `&` operator: `sleep 600 &`.
5. Bring the process to the foreground.
6. Kill the process by `CTRL+C`.



[linux_process_state]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/linux_process_state.png
[NetflixFrontend]: https://github.com/exit-zero-academy/NetflixFrontend.git
[NetflixMovieCatalog]: https://github.com/exit-zero-academy/NetflixMovieCatalog.git
