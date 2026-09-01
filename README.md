## Begin 
### Remove all containers
```text
docker rm -f $(docker ps -a -q)
```
### Initialize docker swarm
```text
docker swarm init
```

### Label the node
`app.yml` and `caddy.yml` schedule their services with the constraint
`node.labels.app==true`. No node carries that label by default, so until
you add it the `api`, `frontend`, and `caddy` services stay stuck in
`Pending` and never start — which is what shows up externally as a 502.
```text
docker node update --label-add app=true $(docker node ls -q --filter role=manager)
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

## Troubleshooting a 502 from the API

A 502 means Caddy (or whatever sits in front of it, e.g. a GCP load
balancer) couldn't get a valid response from the `api` upstream. Work
through these in order:

1. **Service actually scheduled?**
   ```text
   docker service ps app_api --no-trunc
   ```
   If it shows `Pending` with a message like "no suitable node", the
   `app=true` node label is missing — see the labeling step above. Do the
   same for `proxy_caddy` and `app_frontend`.

2. **Container crash-looping?**
   ```text
   docker service logs app_api --tail 100
   ```
   A common cause is the `api` container starting before MySQL is ready
   to accept connections, or `/run/secrets/mysql_user_password` not
   existing because the secret wasn't created before `app.yml` was
   deployed (see "Create secrets" above).

3. **Networks lined up?** `caddy.yml` and `app.yml` must both join the
   same external `my_swarm_network` (created once via `docker network
   create`) — if it was recreated after one of the stacks was deployed,
   redeploy that stack so it re-attaches.

4. **Multi-node swarm on GCP?** GCP's default VPC firewall does not open
   the ports Swarm's overlay network needs between VM instances. If
   `caddy` and `api` land on different nodes, add firewall rules
   (typically for your instances' network tags) allowing, between swarm
   nodes: TCP/UDP 7946 (gossip), UDP 4789 (VXLAN data), and TCP 2377
   (cluster management, manager nodes only).
