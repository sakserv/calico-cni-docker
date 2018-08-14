Running Calico CNI with Docker
------------------------------

# Start two VMs
```
cd calico-cni-docker && vagrant up
```

# Start calico/node on each VM
```
docker logs -f $(docker run --net=host --privileged --name=calico-node -d --restart=always -e NODENAME=`hostname | cut -d. -f1` -e CALICO_NETWORKING_BACKEND=bird -e CALICO_LIBNETWORK_ENABLED=false -e ETCD_ENDPOINTS=http://172.17.8.101:2379 -e WAIT_FOR_DATASTORE=true -e DATASTORE_TYPE=etcdv3 -v /var/log/calico:/var/log/calico -v /var/run/calico:/var/run/calico -v /var/lib/calico:/var/lib/calico -v /lib/modules:/lib/modules -v /run:/run -v /run/docker/plugins:/run/docker/plugins -v /var/run/docker.sock:/var/run/docker.sock quay.io/calico/node:release-v3.2)
```
