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
### Create secrets 
```text
printf 'mypassword' | docker secret create mysql_root_password -
printf 'mypassword' | docker secret create mysql_user_password -
```

### Deploy MySQL database
```text 
docker stack deploy --with-registry-auth -c mysql.yml mysql
```

### Web UI
```text
DOCKER_IMAGE_VERSION=0.0.6 docker stack deploy --with-registry-auth -c app.yml app
```

### Reverse proxy 
#### Configuration
```text
docker config create caddy_config Caddyfile
```
#### Deployment
```text
docker stack deploy --with-registry-auth -c caddy.yml proxy
```
### Quit swarm 
```text
docker swarm leave --force
```
