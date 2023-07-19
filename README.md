# Django_poll_App_on_Docker_swarm
in this repository we Deploy Django Poll App on Docker Swarm 
# what is swarm ?
is the cluster solution based on docker containter to stablish HA For Containers
![swarm-diagram](https://github.com/amirajoodani/Django_poll_on_Docker_swarm/assets/42912741/b06bfcd5-61d0-4e44-bd0a-7f5f2f8fa8f4)
the senario that we assume is : <br>
![swarm cluster](https://github.com/amirajoodani/Django_poll_App_on_Docker_swarm/assets/42912741/1d71480f-b131-4d72-884b-174c53b5ceab)

there is Poll App Created with Django that you can create poll and see result .this app lunch on port 9000 and path is /poll . <br>
prepare 3 vm and every one have two network interface . one of them is for private connection between cluster node . 
# swarm command :
see swarm nodes : <br>
```
docker node ls
```
initialaize docker cluster (you get error because you have two have ): <br>
```
docker swarm init
```
```
(Master node)#docker swarm init --advertise-addr PRIVATE_IP
```
(For adding other nodes to manager as worker node do this command on other nodes): <br>
```
docker swarm join --token SWMTKN-1-0ug3tns725f08jre3a8kl9o48j6xu4wagi0rmqdxrstc2hbwdt-8j1e1yi08u4hvf9u4w4gjh1li 192.168.56.102:2377
```
(For leaving node fron orchastrator):<br>
```
docker swarm leave --force
```
(to add all node as manager):<br>
```
docker node promote swarm-2
docker node promote swarm-3
```
(for logout one node from manager):<br>
```
docker node demote swarm-2
```
# Install NFS For Sharing Database Between nodes:<br>
replica 1 for app and db <br>
(Master node)<br>
```
# apt-get install nfs-kernel-server
# apt-get install nfs-common
# mkdir -p /var/nfs/db
# sudo chown nobody:nogroup /var/nfs/db
# vi /etc/exports
/var/nfs/db 192.168.56.0/24(rw,sync,no_subtree_check,no_root_squash)
(open port on firewall)
# sudo ufw allow from 192.168.56.0/24 to any port nfs
# sudo systemctl restart nfs-kernel-server
#exportfs -a
```
# Deploy Django App on Swarm <br>
(create registery)<br>
```
docker service create --name registery --publish published=5000,target=5000 registery:2
```
push used image in docker compose to registery that all node have access to them :<br>
```
docker-compose push
```
bring up docker compose to have project on master node : <br>
```
docker-compose up -d
```
deploy django app on swarm cluster : <br>
```
docker stack deploy --compose-file docker-compose.yml pollingservice
```
right now , we can see our app on all node : <br>
![1 (2)](https://github.com/amirajoodani/Django_poll_App_on_Docker_swarm/assets/42912741/036caecb-dc16-4403-9d9e-830a14896100)
![2 (1)](https://github.com/amirajoodani/Django_poll_App_on_Docker_swarm/assets/42912741/fcffe530-188d-45b1-b314-27c12b423017)
![3](https://github.com/amirajoodani/Django_poll_App_on_Docker_swarm/assets/42912741/ab146ab8-206e-4348-93fa-7542e6ae9182)
![5](https://github.com/amirajoodani/Django_poll_App_on_Docker_swarm/assets/42912741/67e138a0-9cb8-4723-8b50-d971767e6f17)

see replicated service on cluster : <br>
![4](https://github.com/amirajoodani/Django_poll_App_on_Docker_swarm/assets/42912741/2fb8abdf-5c76-4af2-8c5e-9c3b49953f0d)
see proccess of service : <br>
```
docker service ps pollingservice
```
(for scale service): <br>
```
docker service scale pollingservice=2
```
(for down scale): <br>
```
docker service scale test-swarm=1
```
for removing service its recomended to set replicas to 0 <br>
```
docker service scale test-swarm=0
```
or <br>
```
docker service rm test-swarm
```
# route mesh
```
docker service create --name pollingservice --replicas 2 --publish published=8080,target=80 pollingservice
```
# Adminstration
for maintenance server : <br>
```
docker node update --availability drain docker-swarm-1
```
monitor swarm health: <br>
```
docker node inspect manager1 --format "{{ .ManagerStatus.Reachability }}"
docker node inspect manager1 --format "{{ .Status.State }}"
```
(backup and resotre) : <br>
/var/lib/docker/swarm <br>

# (rebalance): <br>
```
docker service update SERVICE_NAME
```




















