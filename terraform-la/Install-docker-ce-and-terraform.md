# Installing Docker on the Swarm Manager and Worker

These actions will be executed on both the Swarm manager and worker nodes. So we need two servers, a master and a worker node.

Update the operating system

```
sudo yum update -y
```

## Prerequisites

Uninstall old versions:

````
sudo yum remove -y docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
````

# Install Docker CE

***Install Utils:***

````
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
````

Add the Docker repository:

````
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
````

Install Docker CE:

````
sudo yum -y install docker-ce
````

Start Docker and enable it:

````
sudo systemctl start docker && sudo systemctl enable docker
````

Add cloud_user to the docker group:

````
sudo usermod -aG docker cloud_user
````

***Test the Docker installation:***

````
docker --version
````

## Configuring Swarm Manager node
On the manager node, initialize the manager:

````
docker swarm init \
--advertise-addr [PRIVATE_IP]
````

## Configure the Swarm Worker node

On the worker node, add the worker to the cluster:

````
docker swarm join --token [TOKEN] \
[PRIVATE_IP]:2377
````

***Verify the Swarm cluster***

List Swarm nodes:

````
docker node ls
````

## Install Terraform

Install Terraform 0.11.13 on the Swarm manager:

````
sudo curl -O https://releases.hashicorp.com/terraform/0.11.13/terraform_0.11.13_linux_amd64.zip
sudo yum install -y unzip
sudo unzip terraform_0.11.13_linux_amd64.zip -d /usr/local/bin/
````

***Test the Terraform installation:***

````
terraform version
````