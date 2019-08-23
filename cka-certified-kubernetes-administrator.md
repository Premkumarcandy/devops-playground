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
