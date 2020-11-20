---
title: "Docker Setup"
date: 2020-11-19T14:19:58+01:00
authors: ["Jon Brookes"]
draft: false
---

# docker on to Ubuntu 1804 

The following should work for Ubuntu versions 16 18 and 20 and is taken directly from the docker documentation pages ( see References below ) but for completeness this procedure is test run on a new Ubuntu 1804 server installation.

First, the server is updated to ensure the latest packages are present on the system. Then docker is installed using the docker repositories and finally docker-compose is downloaded and added to the path, ready to use compose files in the normal running of docker and the life-cycle of stand alone containers upon this service. 

It is not at this stage intended for clustered multi-node orchestration using things like kubernetes however can be used as a base for these platforms to be deployed to when using docker as a container management system. 


Ensure the system is up to date:

```
apt update -y
apt upgrade -y
```

install the repository

```
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

add docker GPG key

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

add docker repository

```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

```

install docker

```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

check if docker is working

```
sudo docker run hello-world
```

if everything is working the following will be displayed in part :

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
....
```

add `docker-compose` to your system with :

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

to enable your user to run `docker` commands without having to `sudo` all of the time add ***your user name*** to the docker group with :

```
sudo usermod -aG docker <your-user-name>
```

logout / login again or start a new session to effect the change to your access and run the above `docker run hello-world` to test



and we're done

# docker on raspberry pi 400

The following installs Docker and Docker Compose onto a Raspberry Pi 400:

```
sudo apt update
sudo curl -sSL https://get.docker.com | sh
sudo usermod -aG docker pi
sudo apt install -y python3-pip libffi-dev
sudo pip3 install docker-compose
```



# References

https://docs.docker.com/engine/install/ubuntu/

https://docs.docker.com/compose/install/

https://www.raspberrypi.org/blog/docker-comes-to-raspberry-pi/

https://withblue.ink/2020/06/24/docker-and-docker-compose-on-raspberry-pi-os.html

