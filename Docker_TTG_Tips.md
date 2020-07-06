# Exploring Docker

---

Ensure that you have cloned this repository to your local machine:

```
git clone https://gitlab-sjc.cisco.com/CX-EMEAR-TTG/ttg-automation-ws.git
```
---

The first thing we can do is check that Docker is installed and the version we are running;

```
docker -v
```

Docker provides an ecosystem of tools around containers including a free public container register on [hub.dockerhub.com](https://hub.docker.com/).

Ok, let's run a linux container with the standard Ubuntu distribution;

```
docker run --rm -ti ubuntu:latest
```

You can exit and terminate the container by simply typing ```exit``` in the shell.

A docker container name is made up of ```REGISTRY[:PORT]/USER/REPO[:TAG]```.  In the above command there is no REGISTRY specified so we default to ```docker.io```.  Because the Ubuntu container is the 'official' one there is no user associated with it.  The ```:TAG``` at the end is where can specify a specific image or version that we want.

We could also try;

```
docker run --rm -ti docker.io/ubuntu:latest
```

You can search the registry directly from you docker CLI using ```docker search```

```
docker search ubuntu
```

or

```
docker search cunningr
```

Taking a closer look at the ```docker run``` command above, the first option ```--rm``` basically tells the docker daemon to clean up (delete) our container instance when it stops.  This is quite common as containers are often stateless and don't hold any data that needs to be preserved (more on this later).

The second two flags ```-ti``` allocates a pseudo-TTY (```-t```) and keeps an interactive session (```-i```), allowing us to interact with the shell ```/bin/bash```.

Often a container runs only a single process, process 1, and when this process exists the container is stopped!  Try running the Ubuntu container again and doing a;

```
ps -aux
```

You should should see something similar to the below;

```
root@20563116de32:/# ps -aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  1.2  0.0  18504  3084 pts/0    Ss   16:04   0:00 /bin/bash
root        10  0.0  0.0  34396  2932 pts/0    R+   16:04   0:00 ps -aux
```

Notice that the shell process ```/bin/bash``` is PID 1.  Keep the container shell open and in another terminal do a ```docker ps```.  You should see your container running in the output.

Now if you exit the shell and do ```docker ps``` you container should not be listed anymore.  Further more, if you do a ```docker ps --all``` to see even the stopped containers, your container should be gone since we started it with the ```--rm``` option!

## Customising Our Container

Start your ubuntu container again but this time without the ```--rm``` option.  Try to ```curl www.google.com```.  Result?

Use apt-get to install some new software in your container such as curl or wget;

```
apt-get update
apt-get install curl
```

Have fun watching all the installation fly by and then try the curl command again. This time you should be able to see the HTML version of www.google.com!

So now you customised your container with a little software, maybe we want to keep it running.  

You can detach from the pseudo-TTY with ```CTRL+P+Q``` and re-attach with ```docker attach <container-id>``` (use ```docker ps``` to get a list of running containers).

Once re-attached to your container, you can check that ```curl www.google.com``` does indeed still work, however this time, redirect the output to file in the root folder ```curl www.google.com > /google.com```.  Quickly check that this file exists by doing a ```cat /google.com``` the detach from your container again (not exit!)

Now you are back in your local OS environment, can you still do ```cat /google.com```?  Of course not, that file exists somewhere else, buried deep on your HDD where the container file system is mounted.

## Pre-built Containers

Customising containers can be fun but often you can find a container built with just the application you are looking for to save you a lot of time and trouble.  For example ...

 * HTTP servers such NGINX, Apache
 * Database servers such as Cassandra, MongoDB or MySQL
 * TACACS or LDAP servers such as TAC_PLUS or OpenLDAP

Let's quickly start a new Web server and have it listen locally on port 8181;

NOTE: for Windows users, you might need to run the CMD / Powershell with admin rights before launching the container with a port mapping.

```
docker run --rm -p 8181:80 nginx
```

Once the finishes downloading and runs, open a browser to [http://127.0.0.1:8181/](http://127.0.0.1:8181/).  Do you see the default NGINX web page?  If you did, check back on the console and you should see the NGINX access logs.

NOTE: By default ```docker run``` will attach to the container process and most containers will log to console.  Since we didn't specify ```-ti``` also, we can't use ```CTRL+P+Q``` here.

When we started the container above, we opened up the local port ```8181``` and mapped it port ```80``` in the NGINX container, the standard HTTP port.  You can use ```CTRL+C``` to exit your container.

So how can we use a web server in a container to serve custom content?

There are two basic ways;

 1. ```docker cp``` our web files into the container and put them in the web root
 2. Use the ```docker run -v <local-dir>:<container-dir>``` option to mount a local file system directory into our container (notice the colon, ':', to separate the local dir and the contaier dir where it's mounted) 

Remember that one of the goals of containers is that they are stateless so we shouldn't care if a container dies, restarts or gets deleted and launched again from a newer version of the container image.  If we copy files into the container, we are adding data that we might lose or need to copy back to a new container.

If we take the second option, we no longer care what happens to our container.  We can just start a new one and mount our directory again (did you see how we just upgraded our web server software in seconds?!!).

For the command below, you need to find your full path to the git ttg-automation-ws repo and then mount that inside the conatiner at ```/usr/share/nginx/html```, the default site webroot. For Windows users, the full path includes "c:" or whatever disk device on which the git repo has been copied.

(You will notice that there is an ```index.html``` file in this folder)

```
docker run --rm -p 8181:80 -v /Users/cunningr/git-projects/ttg-automation-ws/01_docker/webroot:/usr/share/nginx/html nginx
```

This time when you browse to [http://127.0.0.1:8181/](http://127.0.0.1:8181/) you should see your custom web page.

Try editing the ```index.html``` file locally on your machine and refreshing your browser.  Does the web page update?

## Running Containers in Detached mode

Till now, we have been running containers and keeping our terminal attached to the container process.  But what if we just want to run our software in the background and connect to the service using tcp/ip only?  Enter the ```--detach``` or ```-d``` option.  Let's rerun the NGINX container using this option;

```
docker run -d --rm -p 8181:80 -v /Users/cunningr/git-projects/ttg-automation-ws/01_docker/webroot:/usr/share/nginx/html nginx
```

This time the ```docker run``` command should return you to your local shell.  Now doing a ```docker ps``` we should be able to see our new container running.  Again connect to [http://127.0.0.1:8181/](http://127.0.0.1:8181/) and check that everything is still working as expected.

But where did those logs go?  Weren't they useful?  The docker daemon manages logs from all the running containers and you can see them using the ```docker logs <container-id>``` command.  Add in the ```-f``` for follow and you can follow the container logs just like with the linux command ```tail -f```.

E.g.

```
docker logs -f 479c36a87dd2
```

Execute the above command for your container-id and try refreshing your browser page.  You should see the new logs scrolling by.

Exit the logs trail with ```CTRL+C``` and stop the container with ```docker stop <container-id>```.

Using the container-id to work with our containers is ok, but it is not very efficient.  Kill this container using the ```docker stop 479c36a87dd2``` (example) command (again, as we started with ```--rm``` docker should clean up the container also) and let's re-launch, this time giving the container a friendly name by adding the ```--name``` option;

(NOTE: in the "-v" option, replace the local path for the ttg-automation-ws by the valid one in your PC).

```
docker run -d \
--rm \
-p 8181:80 \
-v /Users/cunningr/git-projects/ttg-automation-ws/01_docker/webroot:/usr/share/nginx/html \
--name www \
nginx
```

(notice we started to split our command over multiple lines using ```\``` which helps make the command more human readable).

Now we should be able to refer to our container by the name ```www```.  So we have our container running in the background and all is well, but what if we need to debug, or customise or container like we did early with the ```/bin/bash``` shell?

We can start a second (or third) process in the _same_ container namespace(s) using the ```docker exec``` command.  So let's start a new shell in this ```www``` namespace so that we can see everything that our NGINX process can see (filesystem, network etc.).  This time we are going to use the new name gave it, ```www```;

```
docker exec -ti www /bin/bash
```

Once you are attached, first do, ```apt-get update && apt-get install procps net-tools``` then do ```ps -aux```.  You should see something like;

```
root@9eb4cc92f3ad:/# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  10628  5412 ?        Ss   17:30   0:00 nginx: master process nginx -g daemon off;
nginx        7  0.0  0.0  11084  2584 ?        S    17:30   0:00 nginx: worker process
root         8  0.0  0.0   3864  3128 pts/0    Ss   17:31   0:00 /bin/bash
root       342  0.0  0.0   7636  2712 pts/0    R+   17:34   0:00 ps aux
```

take a look at the mounted file system;

```
root@9eb4cc92f3ad:/# df
Filesystem     1K-blocks      Used Available Use% Mounted on
overlay         61255492  12920024  45194144  23% /
tmpfs              65536         0     65536   0% /dev
tmpfs            4082264         0   4082264   0% /sys/fs/cgroup
shm                65536         0     65536   0% /dev/shm
/dev/sda1       61255492  12920024  45194144  23% /etc/hosts
osxfs          488245288 116013440 356851320  25% /usr/share/nginx/html
tmpfs            4082264         0   4082264   0% /proc/acpi
tmpfs            4082264         0   4082264   0% /sys/firmware
```

and the local network interface/ip;

```
root@9eb4cc92f3ad:/# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.3  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:03  txqueuelen 0  (Ethernet)
        RX packets 7524  bytes 9572764 (9.1 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3957  bytes 216116 (211.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

Finally, exit the shell and do a ```docker ps```.  Is you NGINX container still running?  Why didn't it exit?

## Docker Build

Sharing is caring! The real power of containers is packing up and configuring software so that it can run in any environment. So long as Docker is installed and there is a linux kernel, our well-built containers should just run ... anywhere.  Let's explore how to build our own containers using a ```Dockerfile```.

First of all we should understand that a container image is built up of one or more _immutable_ file layers.  When we run a container, docker creates a new Copy on Write (CoW) layer where local file modifications are stored. The actual file layers that were pulled down from dockerhub never change.

In fact, each filesystem layer is named using a SHA hash of the layer itself.  This is how docker daemon knows if it already has a copy of a particular file layer locally before downloading the same thing over and over again.

Now consider this file layering concept as we look at the Dockerfile below;

```
FROM ubuntu:latest
RUN apt-get update && apt-get install python3 python3-pip python3-flask -y
COPY flask_hello_world.py /
CMD python3 /flask_hello_world.py
```

The first line ```FROM ubuntu:latest``` takes our initial container layer(s) and then starts 'building' on top, each new line creating a new layer of the container filesystem.

The second line ```RUN``` an apt install for python3 and some python packages, and then the third line ```COPY``` a small python Flask program from the local build directory into the container.  Finally we set the command ```CMD``` that should be executed when the container is run.

Next we run the ```docker build``` command which will look for the Dockerfile in the current directory and try to build the new container.  So that we can identify our container, we can also give a tag;

```
docker build . --tag pythonflask:1.0
```
Above command will require to find in the local directory both 'Dockfile' and 'flask_hello_world.py' files. They are in <path>\ttg-automation-ws\01_docker folder.

When the build is finished, we can run our new container like so, detaching and exposing port 5000;

```
docker run -d -p 5000:5000 pythonflask:1.0
```

Now, either from a browser or using curl, do a http GET to [http://127.0.0.1:5000/](http://127.0.0.1:5000/)

```
curl http://127.0.0.1:5000/
```

Your new web service should simply return the string "Hello World!".

Congratulations, you just built your first python based web API server!

If you have time, take a look at the very small API server code in ```flask_hello_world.py``` and maybe try to modify it to return instead the sum of 2*2 or PI to 10 decimal places!

If you have some time, create an account on Dockerhub (free) and push your new application container for others to run!

Alternatively, you can use the Cisco IT [Enterprise Container Hub](https://containers.cisco.com/) to create and maintain your container images. The Enterprise Container Hub is part of the RunOn service, and the documentation about container hub is available on this [link](https://runon.cisco.com/c/r/runon/Services/CAE/howto/cae_build_image.html).

## A few other bits ...

 * List local images: ```docker images```
 * Inspect a container: ```docker inspect <container-id>```
 * Remove a (stopped) container: ```docker rm <container-id>```

Command references;

```
docker –v
docker search cunningr
docker run
docker build
docker run <ports>
docker run <volumes>
docker run <daemon>
docker exec
docker inspect
docker images
docker logs
docker cp
```

