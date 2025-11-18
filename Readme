# Docker-Installation-Documentation
## Installation of Docker
- For the installation of Docker, I used the following url to guide in installing Docker. 
  - https://docs.docker.com/engine/install/ubuntu/
### Uninstall old versions
- Uninstall all unofficial Docker packages, which may conflict with the official packages provided by Docker.
````
sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc podman-docker containerd runc | cut -f1)
````
### Install using the apt repository
1. Set up Docker **apt** repository:
````
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
````
2. Install the Docker packages:
- To install the latest version of Docker.
````
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
````
- Verify if Docker is running by seeing if it is enabled
````
sudo systemctl status docker
````
- Start Docker if the system is disabled
````
sudo systemctl start docker
````
- Verify that the installation is successful by running the **hello-world** image:
````
sudo docker run hello-word
````

## Install Docker Compose
````
sudo apt update
sudo apt install docker-compose-plugin # Install Docker Compose
docker compose version # Test
````
## Choose your own adventure: GitLab
- If followed the instruction from https://docs.gitlab.com/install/docker/installation/ to complete the installation of GitLab using Docker Compose. 
- Installing GitLab
````
sudo mkdir -p /srv/gitlab/config /srv/gitlab/logs /srv/gitlab/data # Creates the Directories
cd /srv/gitlab # Change to the GitLab directory
sudo nano docker-compose.yml # Below shows what was used
sudo docker compose up -d # Install GitLab
sudo docker ps # Check the container
````
- In docker-compose.yml use
````
version: '3.8'
services:
  web:
    image: 'gitlab/gitlab-ee:latest'
    container_nameL my-gitlab
    restart: always
    hostname: ''
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://localhost'
    ports:
      - '80:80'
      - '443:443'
      - '22:22'
    volumes:
      - './config:/etc/gitlab'
      - './logs:/var/log/gitlab'
      - './data:/var/opt/gitlab'
````
## Docker Picture
- In the document attached to the respository has the picture of sucessfully loging into the GitLab directory and succesfully running it.
- Giving out the following information; Container ID, Image, Command, when it was Created, Status, If it is Running, Ports, Port mapping and its Name

## Logging into the the GitLab browser:
- Using the command of **ip addr** verify the host IP
- Go into the web browswer and type **http://localhost** #http or https depending on the port


## Problem
- When trying to exectue the command of writing docker-compose.yml an error was showing whenever I was trying to execte the command. The error was that I was writing it on my user account and not in root. I added **sudo** on the command and that resolved it.  
