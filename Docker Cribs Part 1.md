# Docker Cribs


**Start a new Docker/Build a Container**

First we need to download the image, this can be done locally, from git or by specifying the remote location.

	Docker build –t <tag> location or path
	
	Docker build –t cxe cxxea_docker

**_Run Docker commands_**

	docker run -it cxe nmap -sS 10.168.50.1

	docker run -it cxe hping3 --scan 1-30,70-90 -S 10.168.50.201

**Additional Commands**

	Remove the container after running use –rm
	The second two flags -ti allocates a pseudo-TTY (-t) and keeps an interactive session (-i), allowing us to interact with the shell /bin/bash.

**Map a local directory to the docker container whilst running a command**

The directory listed after the colon: is the path of the container, this will vary depending on the container. Run the interactive container if you need to establish the path. In the following example I am running nmap from a container called cxe and sending the output to a local file called jon.txt.

	docker run -v /Users/jonhumph/nmap:/home/run -dit cxe nmap -sS 10.168.50.1 -oG jon.txt


The Above command maps local dir with docker directory then runs commands within docker, here in this example using ansible syntax. Alternatively use the following command to execute the linux PWD (current Dir command) in bash (use $ to denote bash command) first to get current directory and map the current directory.  

	docker run --rm -v ${PWD}:/home/run \cxe \ansible-playbook CONSOLE.yaml -i lions.yml
	
**List all containers**

	Docker ps -a

**Stop all containers**

	docker stop $(docker ps -aq)
**Remove all containers**

	docker rm $(docker ps -aq)

**Interactive Bash – Helps with troubleshooting**
	
	docker run -it cxe bash or docker run -it cxe 
	docker run -v ${PWD}:/home/run -it cxe bash
	


	

