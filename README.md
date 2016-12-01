# Docker compose V2 file example for setting up an Artifactory HA cluster
This docker-compose file will demonstrate the auto sync of nginx configurations from artifactory HA cluster.
For the moment, the docker-compose file only starts a HA cluster with a shared volume instead of NFS. 

## Prerequisites

Docker 1.10, either native or with docker-machine (Toolbox on Windows or Mac OS X : https://www.docker.com/docker-toolbox)

### 1. (Optional) Create a big docker-machine on local vmware
For the first time, create a boot2docker VM with the following command line :

    docker-machine create --driver vmwarefusion --vmwarefusion-cpu-count 2 --vmwarefusion-disk-size 80000 --vmwarefusion-memory-size 4096 fusion

    # Tell the docker command line to use this machine
    eval "$(docker-machine env fusion)"

    # Should work
    docker ps

### 2. Setup env variables
2 env variables are mandatory :
- ART_LICENSES : coma separated licenses to be used by Artifactory nodes
    
    export ART_LICENSES="{art_license_1},{art_license_2},{art_license_n}"

2 env variables are optional :
- ART_LOGIN (optional, default=admin)
- ART_PASSWORD (optional, default=password)

### 3. Aliasing the ip machine

    echo "$(docker-machine ip fusion) artifactory-cluster" | sudo tee -a /etc/hosts

## Launching    

    docker-compose up

Without logs
    
    docker-compose up -d

And then, your own artifactory cluster should be available :

https://artifactory-cluster/artifactory

## Usefull commands
Ssh into container 

    docker exec -ti container_id /bin/bash

Print logs:   

    docker logs container_id

## Managing the lifecycle

### Restarting 
Without any data loss, you can safely restart the cluster with

    docker-compose restart

or by CTRL+C and then

    docker-compose up

### Docker volumes
2 named docker volumes (for mysql data and cluster home) are created and persisted, even when containers are deleted.

To list volumes :

    docker volume ls

To delete this 2 volumes :
    
    docker volume rm nginxproxy_clusterhome
    docker volume rm nginxproxy_mysqldata

To copy a file into this volume, for example adding a plugin

    docker cp checksums/checksums.groovy nginxproxy_artifactory_1_1:/var/opt/jfrog/cluster/ha-etc/plugins/.

### Restarting from scratch
Rebuild and restart with fresh data :
    
    docker-compose stop
    docker-compose rm
    docker volume rm nginxproxy_clusterhome
    docker volume rm nginxproxy_mysqldata
    docker-compose build
    docker-compose up
