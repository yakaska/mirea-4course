```bash
docker service create \
--name portainer \
--replicas=1 \
--publish 9000:9000 \
--constraint 'node.role == manager' \
--mount type=volume,source=portainer_data,target=/data \
--mount
type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock \
portainer/portainer-ce:latest \
-H unix:///var/run/docker.sock
```
