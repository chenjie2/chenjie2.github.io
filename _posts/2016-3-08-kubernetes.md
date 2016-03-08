---
title: kubernetes
layout: post
category : kubernetes
tags : [kubernetes]
---
{% include JB/setup %}

##etcd
./etcd --listen-client-urls=http://0.0.0.0:4001 --listen-peer-urls=http://0.0.0.0:7001 -advertise-client-urls=http://0.0.0.0:4001 > /var/log/etcd.log 2>&1 &

##master上运行

kube-apiserver
kube-apiserver --address=0.0.0.0 --insecure-port=8080 --service-cluster-ip-range='10.254.0.0/16' --log-dir=/var/log/kube --kubelet-port=10250 --v=0 --logtostderr=false --etcd-servers=http://20.10.93.215:4001 --allow-privileged=true
kube-apiserver --address=0.0.0.0 --insecure-port=8080 --service-cluster-ip-range='10.254.0.0/16' --log-dir=/var/log/kube --kubelet-port=10250 --v=0 --logtostderr=false --etcd-servers=http://20.12.16.90:4001,http://20.12.16.91:4001,http://20.12.16.92:4001 --allow-privileged=true
kube-controller-manager
kube-controller-manager  --v=0 --logtostderr=false --log-dir=/var/log/kube  --master=127.0.0.1:8080 --cloud-provider=""
kube-controller-manager  --v=0 --logtostderr=false --log-dir=/var/log/kube  --master=20.12.16.90:8080 --cloud-provider=""

kube-scheduler
kube-scheduler  --master='20.10.93.215:8080' --v=0 --log-dir=/var/log/kube
kube-scheduler  --master='20.12.16.90:8080' --v=0 --log-dir=/var/log/kube

##slave上运行：

kube-proxy
nohup kube-proxy  --logtostderr=false --v=0 --log-dir=/var/log/kube --master=http://20.10.93.215:8080 &
nohup kube-proxy  --logtostderr=false --v=0 --log-dir=/var/log/kube --master=http://20.12.16.90:8080 &

kubelet 
nohup kubelet --logtostderr=false --v=0 --allow-privileged=true --log-dir=/var/log/kube --address=0.0.0.0  --port=10250 --hostname-override=20.12.16.91 --api-servers=http://20.12.16.90:8080 &
kubelet --logtostderr=false --v=0 --allow-privileged=true --log-dir=/var/log/kube --address=0.0.0.0  --port=10250 --hostname-override=20.10.93.215 --api-servers=http://20.10.93.215:8080
kubelet --logtostderr=false --v=0 --allow-privileged=true --log-dir=/var/log/kube --address=0.0.0.0  --port=10250 --hostname-override=20.10.93.216 --api-servers=http://20.10.93.215:8080






##启动flannel

tar -zxvf etcd-v2.0.9-linux-amd64.tar.gz
cd etcd-v2.0.9-linux-amd64/
./etcd --listen-client-urls=http://0.0.0.0:4001 --listen-peer-urls=http://0.0.0.0:7001 &
./etcdctl mk /coreos.com/network/config '{"Network":"172.18.0.0/16"}'
./etcdctl mk /coreos.com/network/config '{"Network":"172.17.0.0/16"}'

host1:
ip link set dev docker0 down;brctl delbr docker0;
cd flannel/bin/
./flanneld -iface=ens32 > /var/log/flanneld.log 2>&1 &
source /run/flannel/subnet.env
docker -d --bip=${FLANNEL_SUBNET} --mtu=${FLANNEL_MTU} > /var/log/docker.log 2>&1 &



host2:
ip link set dev docker0 down;brctl delbr docker0
cd flannel/bin/
./flanneld -etcd-endpoints=http://20.10.93.215:4001 -iface=ens32 > /var/log/flanneld.log 2>&1 &
source /run/flannel/subnet.env
docker -d --bip=${FLANNEL_SUBNET} --mtu=${FLANNEL_MTU} > /var/log/docker.log 2>&1 &
nohup docker daemon --log-level=error --bip=${FLANNEL_SUBNET} --mtu=${FLANNEL_MTU} > /var/log/docker.log 2>&1 &
docker daemon --storage-opt dm.thinpooldev --bip=${FLANNEL_SUBNET} --mtu=${FLANNEL_MTU}

host2:
ip link set dev docker0 down;brctl delbr docker0
cd flannel/bin/
./flanneld -etcd-endpoints=http://20.12.16.90:4001,http://20.12.16.91:4001,http://20.12.16.92:4001 -iface=ens160 > /var/log/flanneld.log 2>&1 &
source /run/flannel/subnet.env
docker -d --storage-opt dm.basesize=15G --bip=${FLANNEL_SUBNET} --mtu=${FLANNEL_MTU} > /var/log/docker.log 2>&1 &



socat TCP4-LISTEN:90,reuseaddr,fork TCP4:10.254.38.245:80


##master：

    运行etcd：

etcd > /var/log/etcd.log 2>&1 &

    运行kube-apiserver：

./kube-apiserver --address=0.0.0.0 --insecure-port=8080 --service-cluster-ip-range='10.254.0.0/16' --log_dir=/var/log/kube --kubelet_port=10250 --v=0 --logtostderr=false --etcd_servers=http://127.0.0.1:4001 --allow_privileged=false

kube-apiserver --address=0.0.0.0 --insecure-port=8080 --service-cluster-ip-range='10.254.0.0/16' --log-dir=/var/log/kube --kubelet-port=10250 --v=0 --logtostderr=false --etcd-servers=http://127.0.0.1:4001 --allow-privileged=false

kube-apiserver 的参数说明：https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/man...

    运行kube-controller-manager：

./kube-controller-manager  --v=0 --logtostderr=false --log_dir=/var/log/kube  --master=127.0.0.1:8080

kube-controller-manager  --v=0 --logtostderr=false --log-dir=/var/log/kube  --master=127.0.0.1:8080 --cloud-provider=""

kube-controller-manager的参数说明：https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/man...

    运行kube-scheduler：

./kube-scheduler  --master='127.0.0.1:8080' --v=0 --log-dir=/var/log/kube

kube-scheduler  --master='127.0.0.1:8080' --v=0 --log-dir=/var/log/kube

kube-scheduler的参数说明：https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/man...


##slave：

    运行kube-proxy：

./kube-proxy  --logtostderr=false --v=0 --log_dir=/var/log/kube --master=http://20.10.93.215:8080 

kube-proxy的参数说明：https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/man...

    运行kubelet:

./kubelet  --logtostderr=false --v=0 --allow_privileged=false --log_dir=/var/log/kube --address=0.0.0.0 --port=10250 --hostname_override=20.10.93.215 --api_servers=http://20.10.93.215:8080

./kubelet  --logtostderr=false --v=0 --allow_privileged=true --log_dir=/var/log/kube --address=0.0.0.0 --port=10250 --hostname_override=20.10.93.216 --api_servers=http://20.10.93.215:8080

kubelet的参数说明：https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/man...




## Running your first containers in Kubernetes

Ok, you've run one of the [getting started guides](../docs/getting-started-guides/) and you have
successfully turned up a Kubernetes cluster.  Now what?  This guide will help you get oriented
to Kubernetes and running your first containers on the cluster.

### Running a container (simple version)

From this point onwards, it is assumed that `kubectl` is on your path from one of the getting started guides.

The [`kubectl run`](/docs/kubectl_run.md) line below will create two [nginx](https://registry.hub.docker.com/_/nginx/) [pods](/docs/pods.md) listening on port 80. It will also create a [replication controller](/docs/replication-controller.md) named `my-nginx` to ensure that there are always two pods running.

```bash
kubectl run my-nginx --image=nginx --replicas=2 --port=80
kubectl run my-nginx --image=soma-build-docker:8080/httpd --replicas=3 --port=80
```

Once the pods are created, you can list them to see what is up and running:
```bash
kubectl get pods
```

You can also see the replication controller that was created:
```bash
kubectl get rc
```

To stop the two replicated containers, stop the replication controller:
```bash
kubectl stop rc my-nginx
```

### Exposing your pods to the internet.
On some platforms (for example Google Compute Engine) the kubectl command can integrate with your cloud provider to add a [public IP address](/docs/services.md#external-services) for the pods,
to do this run:

```bash
kubectl expose rc my-nginx --port=80 --type=LoadBalancer
```

This should print the service that has been created, and map an external IP address to the service. Where to find this external IP address will depend on the environment you run in.  For instance, for Google Compute Engine the external IP address is listed as part of the newly created service and can be retrieved by running

```bash
kubectl get services
```

In order to access your nginx landing page, you also have to make sure that traffic from external IPs is allowed. Do this by opening a firewall to allow traffic on port 80.

### Next: Configuration files
Most people will eventually want to use declarative configuration files for creating/modifying their applications.  A [simplified introduction](simple-yaml.md)
is given in a different document.



kubectl run httpd --image=soma-build-docker:8080/httpd --replicas=3 --port=80
kubectl expose rc httpd  --port=85 --target-port=80 --type=LoadBalancer
kubectl expose rc httpdnode  --port=80 --target-port=80 --type=NodePort

kubectl expose rc 23e1cdc9  --port=6379 --target-port=6379 --type=NodePort --name=servicenodeport

kubectl scale --replicas=5 replicationcontrollers httpd



