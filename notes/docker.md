# Docker

## Docker Linux Install
``` bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install -y docker.io

systemctl start docker
systemctl enable docker

sudo apt-get install -y python python-pip
pip install docker-compose
```

## Docker Hub Publish
``` bash
docker login
docker tag image username/repo:tag
docker push username/repo:tag
docker run -p 5000:80 username/repo:tag
```

## Docker Machine
* Create a droplet with docker properly setup
``` bash
docker-machine create --driver digitalocean --digitalocean-access-token PERSONAL_ACCESS_TOKEN DROPLET_NAME
```
* List running machines
``` bash
docker-machine ls
```
* Activate docker for specified container
``` bash
eval $(docker-machine env DROPLET_NAME)
```
* Deactivate docker for a remote container
``` bash
eval $(docker-machine env -u)
```
* Remove a droplet
``` bash
docker-machine rm DROPLET_NAME
```

## Useful Images
* Run swagger editor locally on "http://locahost:80"
``` bash
docker run -d -p 80:8080 swaggerapi/swagger-editor
```
