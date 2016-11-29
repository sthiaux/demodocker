# 0. Introduction

### Docker Container
 - cgroups and namespaces
 - Union file systems
 - go programming language

### Vm vs container
![](images/1.jpg)

### Build Ship Run
![](images/2.png)

### Advantages of running docker
- Rapid application deployment + testing
- Portability across machines
- Version control and component reuse
- Sharing
- Lightweight footprint and minimal overhead
- Simplified maintenance


### Drawbacks

- ...

# 1. Installing docker
```
apt-get update
apt-get install -y apt-transport-https ca-certificates linux-image-extra-$(uname -r) linux-image-extra-virtual
apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
echo "deb https://apt.dockerproject.org/repo ubuntu-xenial main" | sudo tee /etc/apt/sources.list.d/docker.list
apt-get update
apt-get install -y docker-engine
service docker start
usermod -aG docker ubuntu
```
# 2. Introduction to images
```
differences bw images and containers

hub docker +
docker search

official images

tags
```
# 3. Managing containers
```
--help

docker run hello-world
docker --version
pull
run
exec
inspect
rm
ps
stats
logs
```
# 4. Building images

### Images and layers
![](images/3.jpg)

### Dockerfile
```
https://github.com/sth35web/demodocker
http://192.168.48.54/
```
# 5. Distributing images
```
docker import /export
public/private registry
docker tag
docker push / pull
docker rmi
```
# 6. Volumes 
```
docker run -d -p 80:80 -v /home/ubuntu/demo/demo:/usr/share/nginx/html/demo  --name nginxsth nginxsth:latest

docker volume create --name nginxhtml

run -d -p 80:80 -v nginxhtml:/usr/share/nginx/html/demo  --name nginxsth nginxsth:latest

echo toto > /var/lib/docker/volumes/nginxhtml/_data/index.html
```
https://docs.docker.com/engine/userguide/storagedriver/images/driver-pros-cons.png

# 7. Networking

### Port mapping

Default:
- Containers can make connections to the outside world
- Outside world cannot connect to containers

Mapping:
- To accept incoming connections, specify option -P or -p IP:host_port:container_port in 'run' command
- iptables -t nat -L -n

### DNS

Embedded DNS
- Docker uses embedded DNS to provide service discovery for containers (127.0.0.11:53)
- Key/value store in Docker Engine
- Embedded DNS is network-scoped (Containers not on the same network cannot resolve each other's addresses)

### Bridging
```
docker network ls
docker network inspect bridge

docker exec -ti nginxsth bash
  ping 172.17.0.4 => ok
```  
![](images/bridge1.png)

```
docker network create --driver bridge isolated_nw
docker run -d -P --network=isolated_nw nginxsth:latest
docker network inspect isolated_nw
docker exec -ti fb82aad80f58 bash
  ping 172.18.0.2 => ok
  ping reverent_ramanujan => KO


docker run -d -P --network=isolated_nw --name c1  nginxsth:latest
docker run -d -P --network=isolated_nw --name c2  nginxsth:latest
docker exec -ti c1 bash
  ping c2 => ok
```  
![](images/bridge2.png)

```
#exemple d'isolation :

docker network create --driver bridge front
docker network create --driver bridge back

docker run -d --net=front --name web  nginxsth
docker run -d --net=front --name proxy  nginxsth
docker network connect back proxy
docker run -d --net=back  --name app  nginxsth

```
# 8. Registry

```
docker run -d -p 5000:5000 \
 --restart=always \
 --name registry  \ 
 -e REGISTRY_STORAGE=swift \ 
 -e REGISTRY_STORAGE_SWIFT_USERNAME=xxxxxxxxxxxx \  
 -e REGISTRY_STORAGE_SWIFT_PASSWORD=xxxxxxxx \  
 -e REGISTRY_STORAGE_SWIFT_AUTHURL=http://xxxxxxxxxxxxxxxxxx \  
 -e REGISTRY_STORAGE_SWIFT_CONTAINER=docker \  
 -v  /home/ubuntu/registry/ca:/etc/ssl/certs/ \ 
 registry:2

docker push 192.168.48.54:5000/google/cadvisor

api
authentification
notifications

portus
```
# 9. Continuous Integration

### Github -> dockerhub
https://github.com/sth35web/formationdocker/blob/master/Dockerfile
https://hub.docker.com/r/sth35web/formationdocker/builds/

### Gitlab CI
GitLab CI in conjunction with GitLab Runner can use Docker Engine to test and build any application.

# 10. Multihost networking


### Overlay outside swarm mode

- Requires a valid key-value store service
- Consul, Etcd, and ZooKeeper
- Configure docker engines to use the key-value store

![](images/6.png)

### Overlay in swarm mode

- Embedded key-value store

![](images/7.png)

### Plugins
Nuage, Contrail, Midokura, etc...


# 11. Docker machine

Docker Machine enables you to 
- Provision and manage multiple remote Docker hosts
- Provision Swarm clusters

![](images/8.png)

Docker machine
- Automatically creates hosts
- Installs Docker Engine on them
- Configures the docker clients (~/.docker)

Driver:
- AWS (ok) 
- Openstack (ok)
- Virtualbox (ok)
- Azure
- Google Compute Engine
- VMware
- ...

Demo !

```
curl -L https://github.com/docker/machine/releases/download/v0.8.2/docker-machine-`uname -s`-`uname -m` >/usr/local/bin/docker-machine && \
chmod +x /usr/local/bin/docker-machine

```
```
userkeyname=dockerkey
myname=sth
 
for instance in 1 2 3; do
 docker-machine create --driver openstack \
  --openstack-username xxxx \
  --openstack-password xxxx \
  --openstack-tenant-name docker \
  --openstack-auth-url https://xxx:xxx/vxx \
  --openstack-flavor-name m1.large \
  --openstack-image-name Ubuntu-16.04 \
  --openstack-ssh-user ubuntu \
  --openstack-sec-groups default,docker \
  --openstack-keypair-name $userkeyname \
  --openstack-private-key-file .ssh/id_rsa \
  docker$myname-node-$instance
done
```


# 12. Docker compose

Compose:
- Tool for defining and running multi-container Docker applications
- Create and start all the services from your configuration

Intallation
```
curl -L "https://github.com/docker/compose/releases/download/1.8.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

```
version: '2'

services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: wordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "80:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_PASSWORD: wordpress
volumes:
    db_data:
```

docker-compose up
docker-compose ps
docker-compose scale db=2
docker-compose down
...


# 13. Docker Swarm

# 14. Security



