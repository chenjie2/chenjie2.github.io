---
layout: post
title: etcd集群配置 
categories: [kubernetes]
description: etcd集群配置 
keywords: kubernetes, etcd
---

etcd集群配置

## 启动
nohup etcd -name infra0 -data-dir /home/infra0.etcd -initial-advertise-peer-urls http://20.12.16.90:7001 \
  -listen-peer-urls http://20.12.16.90:7001 \
  -listen-client-urls http://20.12.16.90:4001,http://127.0.0.1:4001 \
  -advertise-client-urls http://20.12.16.90:4001 \
  -initial-cluster-token kube-cluster \
  -initial-cluster infra0=http://20.12.16.90:7001,infra1=http://20.12.16.91:7001,infra2=http://20.12.16.92:7001 \
  -initial-cluster-state new > /var/log/etcd.log 2>&1 &

nohup etcd -name infra1 -data-dir /home/infra1.etcd -initial-advertise-peer-urls http://20.12.16.91:7001 \
  -listen-peer-urls http://20.12.16.91:7001 \
  -listen-client-urls http://20.12.16.91:4001,http://127.0.0.1:4001 \
  -advertise-client-urls http://20.12.16.91:4001 \
  -initial-cluster-token kube-cluster \
  -initial-cluster infra0=http://20.12.16.90:7001,infra1=http://20.12.16.91:7001,infra2=http://20.12.16.92:7001 \
  -initial-cluster-state new > /var/log/etcd.log 2>&1 &

nohup etcd -name infra2 -data-dir /home/infra2.etcd -initial-advertise-peer-urls http://20.12.16.92:7001 \
  -listen-peer-urls http://20.12.16.92:7001 \
  -listen-client-urls http://20.12.16.92:4001,http://127.0.0.1:4001 \
  -advertise-client-urls http://20.12.16.92:4001 \
  -initial-cluster-token kube-cluster \
  -initial-cluster infra0=http://20.12.16.90:7001,infra1=http://20.12.16.91:7001,infra2=http://20.12.16.92:7001 \
  -initial-cluster-state new > /var/log/etcd.log 2>&1 &

## 常见操作
etcdctl ls / --recursive


Remove a Member
 
Let us say the member ID we want to remove is a8266ecf031671f3. We then use the remove command to perform the removal:

$ etcdctl member remove a8266ecf031671f3
Removed member a8266ecf031671f3 from cluster

The target member will stop itself at this point and print out the removal in the log:

etcd: this member has been permanently removed from the cluster. Exiting.

It is safe to remove the leader, however the cluster will be inactive while a new leader is elected. This duration is normally the period of election timeout plus the voting process.
Add a New Member

Adding a member is a two step process:

    Add the new member to the cluster via the members API or the etcdctl member add command.
    Start the new member with the new cluster configuration, including a list of the updated members (existing members + the new member).

Using etcdctl let's add the new member to the cluster by specifying its name and advertised peer URLs:

$ etcdctl member add infra3 http://10.0.1.13:2380
added member 9bf1b35fc7761a23 to cluster

ETCD_NAME="infra3"
ETCD_INITIAL_CLUSTER="infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra2=http://10.0.1.12:2380,infra3=http://10.0.1.13:2380"
ETCD_INITIAL_CLUSTER_STATE=existing

etcdctl has informed the cluster about the new member and printed out the environment variables needed to successfully start it. Now start the new etcd process with the relevant flags for the new member:

$ export ETCD_NAME="infra3"
$ export ETCD_INITIAL_CLUSTER="infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra2=http://10.0.1.12:2380,infra3=http://10.0.1.13:2380"
$ export ETCD_INITIAL_CLUSTER_STATE=existing
$ etcd -listen-client-urls http://10.0.1.13:2379 -advertise-client-urls http://10.0.1.13:2379  -listen-peer-urls http://10.0.1.13:2380 -initial-advertise-peer-urls http://10.0.1.13:2380 -data-dir %data_dir%

The new member will run as a part of the cluster and immediately begin catching up with the rest of the cluster.

If you are adding multiple members the best practice is to configure a single member at a time and verify it starts correctly before adding more new members. If you add a new member to a 1-node cluster, the cluster cannot make progress before the new member starts because it needs two members as majority to agree on the consensus. You will only see this behavior between the time etcdctl member add informs the cluster about the new member and the new member successfully establishing a connection to the existing one.

