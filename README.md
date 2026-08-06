## Begin 
### Remove all containers
```text
docker rm -f $(docker ps -a -q)
```
### Initialize docker swarm
```text
docker swarm init
```

### Start portainer
```text
docker pull portainer/portainer-ce:2.0.1
docker pull portainer/agent
docker stack deploy -c portainer.yml por
```

### Create the overlay network 
```text
docker network create --scope=swarm --driver=overlay --attachable my_swarm_network
```
### Deploy MySQL database
```text 
docker stack deploy --with-registry-auth -c mysql.yml mysql
```

### Quit swarm 
```text
docker swarm leave --force
``
