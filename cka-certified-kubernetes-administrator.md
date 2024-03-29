# Certified Kubernetes Administrator (CKA) 

Certified Kubernetes Administrator: https://www.cncf.io/certification/cka/

Exam Curriculum (Topics): https://github.com/cncf/curriculum

Candidate Handbook: https://www.cncf.io/certification/candidate-handbook

Exam Tips: http://training.linuxfoundation.org/go//Important-Tips-CKA-CKAD

## What is ETCD
Distributed reliable key-value store that is simple, secure and fast

*Learn how to install etcd : https://computingforgeeks.com/how-to-install-etcd-on-rhel-centos-8/*
- etcd will start a service and listen on port 2379
- etcdctl 
    ./etcdctl set key1 value1

### etcd in k8s
- if you install k8s manually, etcd has to install manually
- if k8s installation is by using kubeadm, then kubeadm will install and configure etcd

- etcd will be running under kube-system namespace
```
kubectl get pods -n kube-system 
    # you will see a pod running with name 'etcd-master'
kubecrl exec etcd-master -n kube-system etcdctl get / --prefix -keys-only
```
- in a HA system, there will be multiple master nodes and in the cluster and each master node will have an etcd pod running for HA.

## kube-api server
- kubectl command is hitting on kube-api server 
- kube-api server will authenticate and validate the request and send back the result.
- It also assist to retrive information, update etcd etc

Config file ```/etc/kubernetes/manifests/kube-apiserver.yaml```

## kube controller manager
- continuous monitoring of cluster and components
- remediation

There are other controllers under kube controller manager like deployment controller, job controller, name-space controller etc.

Check controller manager status by ```kubectl get pods -n kube-system```.
### Node Controllder
**Node monitoring**
- will monitor status of every nodes in the cluster every 5 sec; if not reachable then it will wait for 40 sec (grace period) before marking as unreachable
- if unreachable for 5 min, it will evict the pods from that node and deploy to other available nodes

### Replication Controller
- Monitor the status of replicasets and make sure desired number of replicas are running on nodes.

- Replication controller and Replica Sets -> for same purpose but not same
    - Replication controller is the old technology
    - Replica set is the new recommended method

**Sample rc-def.yml**
```
apiVerion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp
      label:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-controller
        image: nginx
  replicas: 3
```

**Sample replicationset-def.yml**

*See the difference and options from replicationcontrollder like ```selector``` etc.*

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp
      label:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-controller
        image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```

**Sample pod-defenition.yml**
```
apiVerion: v1
kind: pod
```

#### Load balancing and Scaling
You can scale application by below methods.
- changing the ```replicas``` option in replicaset defenition. And then run ```kubectl replace -f replicaset-defenition.yml```
- run ```kubectl scale --replicas=6 -f replicaset-defenition.yml```
- run ```kubectl scale --replicas=6 replicaset myapp-replicaset``` (Note: this one will not update replica details in replicaset defenition file.)

 
## Kube Scheduler
- kube scheduler doesnt place pods on nodes, instead it will decide where to place the pods.
- placing pods to nodes is the job of kubelet

**How to see kube-scheduler options**
- ```/etc/kubernetes/manifests/kube-scheduler.yaml```
- ```systemctl status kube-scheduler``` - pod defenition file
- ```ps -aux|grep kube-scheduler```

## Kubelet
- like a captain of ship
- enquire status and report back to master node 
- **kubeadm install will not install kubelet by default**; you must always need to manually install it.

## Kube Proxy
- pod network
- pods are accessible by service IP instead of pod IP
- kube-proxy is running on all nodes
- everytime a new service is created, kube-proxy will coordinate to create the same rule (iptables) on other podes so that pods behind the service will be accessible on all nodes.
- kubeadm will deploy kube-proxy under kube-system namespace
- kube-proxy is deployed a daemonset and will be deployed on all nodes in the cluster - ```kubectl get daemonset -n kube-system```

## pods
- k8s doesn't deploy containers directly to nodes, instead it will encapsulated in an object called pod. 
- smallest object in kubernetes


##Practice Test
```
# create an nginx pod
kubectl run nginx --image=nginx --generator=run-pod/v1

# check pods in current project
kubectl get pods

# delete a pod
kubectl delete pod webapp

# Create a new pod with the name 'redis' and with the image 'redis123'
kubectl run redis --image=redis123 --generator=run-pod/v1
```
 