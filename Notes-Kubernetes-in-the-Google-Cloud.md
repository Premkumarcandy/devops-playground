# Kubernetes in the Google Cloud
  * [Introduction to Docker](#introduction-to-docker)
  * [Kubernetes Engine: Qwik Start](#kubernetes-engine--qwik-start)
    + [Setting a default compute zone](#setting-a-default-compute-zone)
    + [Create cluster](#create-cluster)
    + [Get authentication credentials for the cluster](#get-authentication-credentials-for-the-cluster)
    + [Deploying an application to the cluster](#deploying-an-application-to-the-cluster)
  * [Orchestrating the Cloud with Kubernetes](#orchestrating-the-cloud-with-kubernetes)
    + [Pods](#pods)
    + [Interacting with Pods](#interacting-with-pods)
    + [Services](#services)
    + [Adding Labels to Pods](#adding-labels-to-pods)
    + [Deploying Applications with Kubernetes](#deploying-applications-with-kubernetes)
  * [Managing Deployments Using Kubernetes Engine](#managing-deployments-using-kubernetes-engine)
    + [Scale a Deployment](#scale-a-deployment)
    + [Rolling update](#rolling-update)
    + [Rollback an update](#rollback-an-update)
    + [Canary deployments](#canary-deployments)
    + [Canary deployments in production - session affinity](#canary-deployments-in-production---session-affinity)
    + [Blue-green deployments](#blue-green-deployments)
      - [Blue-Green Rollback](#blue-green-rollback)
  * [Continuous Delivery with Jenkins in Kubernetes Engine](#continuous-delivery-with-jenkins-in-kubernetes-engine)
    + [What is Kubernetes Engine?](#what-is-kubernetes-engine-)
    + [What is Jenkins?](#what-is-jenkins-)
    + [Install Helm](#install-helm)
    + [Configure and Install Jenkins](#configure-and-install-jenkins)
      - [Connect to Jenkins](#connect-to-jenkins)
      - [Understanding the Application](#understanding-the-application)
      - [Deploying the Application](#deploying-the-application)
    + [Creating the Jenkins Pipeline](#creating-the-jenkins-pipeline)
      - [Adding your service account credentials](#adding-your-service-account-credentials)
      - [Creating the Jenkins job](#creating-the-jenkins-job)
    + [Creating the Development Environment](#creating-the-development-environment)
      - [Creating a development branch](#creating-a-development-branch)
      - [Modify the site](#modify-the-site)
    + [Deploying to production](#deploying-to-production)


## Introduction to Docker

docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
9db2ca6ccae0: Pull complete
Digest: sha256:4b8ff392a12ed9ea17784bd3c9a8b1fa3299cac44aca35a85c90c5e3c7afacdc
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
...

$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              fce289e99eb9        6 months ago        1.84kB

$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS                          PORTS               NA
MES
d2ac9eaf5345        hello-world         "/hello"            19 seconds ago       Exited (0) 18 seconds ago                           sa
d_yonath
d4166a5d66b2        hello-world         "/hello"            About a minute ago   Exited (0) About a minute ago                       di
stracted_knuth

```
$ mkdir test && cd test
$ cat > Dockerfile <<EOF
# Use an official Node runtime as the parent image
FROM node:6

# Set the working directory in the container to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Make the container's port 80 available to the outside world
EXPOSE 80

# Run app.js using node when the container launches
CMD ["node", "app.js"]
```

```
cat > app.js <<EOF
const http = require('http');

const hostname = '0.0.0.0';
const port = 80;

const server = http.createServer((req, res) => {
    res.statusCode = 200;
      res.setHeader('Content-Type', 'text/plain');
        res.end('Hello World\n');
});

server.listen(port, hostname, () => {
    console.log('Server running at http://%s:%s/', hostname, port);
});

process.on('SIGINT', function() {
    console.log('Caught interrupt signal and will exit');
    process.exit();
});
EOF
EOF
```

Build it
```
docker build -t node-app:0.1 .
```
It might take a couple of minutes for this command to finish executing.

List it
```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
node-app            0.1                 9abd86ddd1eb        6 seconds ago       884MB
node                6                   ab290b853066        2 months ago        884MB
hello-world         latest              fce289e99eb9        6 months ago        1.84kB
```

```
$ docker run -p 4000:80 --name my-app node-app:0.1
Server running at http://0.0.0.0:80/
```

On another terminal:
$ curl http://localhost:4000
Hello World


The container will run as long as the initial terminal is running. If you want the container to run in the background (not tied to the terminal's session), you need to specify the -d flag.

$ docker run -p 4000:80 --name my-app -d node-app:0.1
3ef42db03f10920c8fa2645e4d3d3e58f03e4d28892bd66730885dcf62911d81

$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                  NAMES
3ef42db03f10        node-app:0.1        "node app.js"       19 seconds ago      Up 18 seconds       0.0.0.0:4000->80/tcp   my-app

Edit app.js
```
....
const server = http.createServer((req, res) => {
    res.statusCode = 200;
      res.setHeader('Content-Type', 'text/plain');
        res.end('Welcome to Cloud\n');
});
....
```
and Docker build

Tag project and push
$ docker tag node-app:0.2 gcr.io/qwiklabs-gcp-gcpd-5a0d22780ec3/node-
app:0.2

$ docker images
REPOSITORY                                       TAG                 IMAGE ID            CREATED             SIZE
node-app                                         0.2                 a459ec7b7ffa        2 minutes ago       884MB
gcr.io/qwiklabs-gcp-gcpd-5a0d22780ec3/node-app   0.2                 a459ec7b7ffa        2 minutes ago       884MB
node-app                                         0.1                 9abd86ddd1eb        6 minutes ago       884MB
node                                             6                   ab290b853066        2 months ago        884MB
hello-world                                      latest              fce289e99eb9        6 months ago        1.84kB

$ docker push gcr.io/qwiklabs-gcp-gcpd-5a0d22780ec3/node-app:0.2


Test it by removing all items from current space
docker stop $(docker ps -q)
docker rm $(docker ps -aq)

docker rmi node-app:0.2 gcr.io/[project-id]/node-app node-app:0.1
docker rmi node:6
docker rmi $(docker images -aq) # remove remaining images
docker images

Now pull from registry
```
docker pull gcr.io/[project-id]/node-app:0.2
docker run -p 4000:80 -d gcr.io/[project-id]/node-app:0.2
curl http://localhost:4000
```


## Kubernetes Engine: Qwik Start
```
$ gcloud config list project
[core]
project = qwiklabs-gcp-gcpd-5a0d22780ec3
Your active configuration is: [cloudshell-5386]
```

### Setting a default compute zone
```
gcloud config set compute/zone us-central1-a
```

### Create cluster
```
gcloud container clusters create mykube01
```

### Get authentication credentials for the cluster
```
$ gcloud container clusters get-credentials mykube01
Fetching cluster endpoint and auth data.
kubeconfig entry generated for mykube01.
```

### Deploying an application to the cluster
```
$ kubectl run hello-server --image=gcr.io/google-samples/hello-app:1.0 --port 8080
```
Expose it
kubectl expose deployment hello-server --type="LoadBalancer"

get details
kubectl get service hello-server
NAME           TYPE           CLUSTER-IP   EXTERNAL-IP      PORT(S)          AGE
hello-server   LoadBalancer   10.0.5.52    35.239.160.169   8080:31523/TCP   82s

Test with IP


## Orchestrating the Cloud with Kubernetes
Set Region
```
gcloud config set compute/zone us-central1-b
```
create a cluster
```
gcloud container clusters create io
```

Get the sample code
```
git clone https://github.com/googlecodelabs/orchestrate-with-kubernetes.git
cd orchestrate-with-kubernetes/kubernetes
ls
```

Create Deployment
```
kubectl create deployment nginx --image=nginx:1.10.0
```

Check pod status
```
kubectl get pods
```
Expose the service
```
kubectl expose deployment nginx --port 80 --type LoadBalancer
```
Behind the scenes Kubernetes created an external Load Balancer with a public IP address attached to it. Any client who hits that public IP address will be routed to the pods behind the service. In this case that would be the nginx pod.

Check services
```
$ kubectl get services
NAME         TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP      10.0.0.1     <none>        443/TCP        8m41s
nginx        LoadBalancer   10.0.11.62   <pending>     80:32594/TCP   40s
```

Check access using curl

### Pods
At the core of Kubernetes is the Pod.

Pods represent and hold a collection of one or more containers. Generally, if you have multiple containers with a hard dependency on each other, you package the containers inside a single pod.
```
$ cat pods/monolith.yaml
apiVersion: v1
kind: Pod
metadata:
  name: monolith
  labels:
    app: monolith
spec:
  containers:
    - name: monolith
      image: kelseyhightower/monolith:1.0.0
      args:
        - "-http=0.0.0.0:80"
        - "-health=0.0.0.0:81"
        - "-secret=secret"
      ports:
        - name: http
          containerPort: 80
        - name: health
          containerPort: 81
      resources:
        limits:
          cpu: 0.2
          memory: "10Mi"
```
Create a pod
```
kubectl create -f pods/monolith.yaml
```
Describe pod
```
kubectl describe pods monolith
```

### Interacting with Pods
By default, pods are allocated a private IP address and cannot be reached outside of the cluster. Use the kubectl port-forward command to map a local port to a port inside the monolith pod.

on the 2nd terminal, run this command to set up port-forwarding:
```
kubectl port-forward monolith 10080:80
```

Now in the 1st terminal start talking to your pod using curl:
```
curl http://127.0.0.1:10080
```

curl -u user http://127.0.0.1:10080/login

At the login prompt, use the super-secret password "password" to login.
TOKEN=$(curl http://127.0.0.1:10080/login -u user|jq -r '.token')

then
curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:10080/secure

See logs
```
$ kubectl logs monolith
2019/07/23 09:41:21 Starting server...
2019/07/23 09:41:21 Health service listening on 0.0.0.0:81
2019/07/23 09:41:21 HTTP service listening on 0.0.0.0:80
127.0.0.1:38478 - - [Tue, 23 Jul 2019 09:43:21 UTC] "GET / HTTP/1.1" curl/7.52.1
127.0.0.1:38512 - - [Tue, 23 Jul 2019 09:43:52 UTC] "GET /secure HTTP/1.1" curl/7.52.1
127.0.0.1:38526 - - [Tue, 23 Jul 2019 09:44:07 UTC] "GET /login HTTP/1.1" curl/7.52.1
127.0.0.1:38556 - - [Tue, 23 Jul 2019 09:44:33 UTC] "GET /login HTTP/1.1" curl/7.52.1
127.0.0.1:38562 - - [Tue, 23 Jul 2019 09:44:45 UTC] "GET /login HTTP/1.1" curl/7.52.1
127.0.0.1:38572 - - [Tue, 23 Jul 2019 09:45:04 UTC] "GET /secure HTTP/1.1" curl/7.52.1
```
To see logs 
kubectl logs -f monolith 

Interactive Shell
```
kubectl exec monolith --stdin --tty -c monolith /bin/sh
```

### Services
Pods aren't meant to be persistent. Services provide stable endpoints for Pods.
The level of access a service provides to a set of pods depends on the Service's type. Currently there are three types:
ClusterIP (internal) -- the default type means that this Service is only visible inside of the cluster,
NodePort gives each node in the cluster an externally accessible IP and
LoadBalancer adds a load balancer from the cloud provider which forwards traffic from the service to Nodes within it.

Create Service
```
cd ~/orchestrate-with-kubernetes/kubernetes
kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-proxy-conf --from-file nginx/proxy.conf
kubectl create -f pods/secure-monolith.yaml

cat services/monolith.yaml
kind: Service
apiVersion: v1
metadata:
  name: "monolith"
spec:
  selector:
    app: "monolith"
    secure: "enabled"
  ports:
    - protocol: "TCP"
      port: 443
      targetPort: 443
      nodePort: 31000
  type: NodePort
 ```
 
 Enable port in GCP
 ```
 gcloud compute firewall-rules create allow-monolith-nodeport \
  --allow=tcp:31000
  ```
  
First, get an external IP address for one of the nodes.
  ```
  $ gcloud compute instances list
NAME                               ZONE           MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
gke-io-default-pool-b58e5577-3vs4  us-central1-b  n1-standard-1               10.128.0.4   35.188.35.189   RUNNING
gke-io-default-pool-b58e5577-rkmj  us-central1-b  n1-standard-1               10.128.0.2   35.225.241.22   RUNNING
```
And try curl.

### Adding Labels to Pods

```
kubectl get pods -l "app=monolith"
kubectl get pods -l "app=monolith,secure=enabled"
kubectl label pods secure-monolith 'secure=enabled'
kubectl get pods secure-monolith --show-labels
kubectl describe services monolith | grep Endpoints

gcloud compute instances list
```

### Deploying Applications with Kubernetes
We're going to break the monolith app into three separate pieces:

auth - Generates JWT tokens for authenticated users.
hello - Greet authenticated users.
frontend - Routes traffic to the auth and hello services.

```
$ cat deployments/auth.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: auth
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: auth
        track: stable
    spec:
      containers:
        - name: auth
          image: "kelseyhightower/auth:1.0.0"
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81

```

Create Deployment
```
kubectl create -f deployments/auth.yaml

Create service
kubectl create -f services/auth.yaml
```
Now do the same thing to create and expose the hello deployment:
```
kubectl create -f deployments/hello.yaml
kubectl create -f services/hello.yaml
```
And one more time to create and expose the frontend Deployment.

kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
kubectl create -f deployments/frontend.yaml
kubectl create -f deployments/frontend.yaml

kubectl get services frontend
curl -k https://<EXTERNAL-IP>
    

## Managing Deployments Using Kubernetes Engine
Set zone
gcloud config set compute/zone us-central1-a

clone repo
```
git clone https://github.com/googlecodelabs/orchestrate-with-kubernetes.git
cd orchestrate-with-kubernetes/kubernetes
```
Create Cluster
```
gcloud container clusters create bootcamp --num-nodes 5 --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"
```

Learn about deployment
```
kubectl explain deployment
kubectl explain deployment.metadata.name
```

Edit vi deployments/auth.yaml
...
containers:
- name: auth
  image: kelseyhightower/auth:1.0.0
...

Create deployment
```
$ kubectl create -f deployments/auth.yaml
$ kubectl get deployments
NAME   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
auth   1         1         1            1           2m41s
```
Get replica sets
```
$ kubectl get replicasets
NAME              DESIRED   CURRENT   READY   AGE
auth-6bb8dcd7bd   1         1         1       3m13s

$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
auth-6bb8dcd7bd-js66t   1/1     Running   0          3m50s
```

```
kubectl create -f services/auth.yaml
kubectl create -f deployments/hello.yaml
kubectl create -f services/hello.yaml

kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
kubectl create -f deployments/frontend.yaml
kubectl create -f services/frontend.yaml

$ kubectl get services frontend
NAME       TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
frontend   LoadBalancer   10.113.12.176   <pending>     443:31784/TCP   5s

curl -ks https://<EXTERNAL-IP>
```

### Scale a Deployment
```
kubectl explain deployment.spec.replicas
kubectl scale deployment hello --replicas=5
kubectl get pods | grep hello- | wc -l
kubectl scale deployment hello --replicas=3

$ kubectl get pods | grep hello- 
hello-5cbf94fc49-744qk      0/1     Terminating   0          25s
hello-5cbf94fc49-9t7wg      1/1     Running       0          3m12s
hello-5cbf94fc49-bvbkj      0/1     Terminating   0          25s
hello-5cbf94fc49-dpp5d      1/1     Running       0          3m12s
```

### Rolling update
https://cdn.qwiklabs.com/uc6D9jQ5Blkv8wf%2FccEcT35LyfKDHz7kFpsI4oHUmb0%3D
```
kubectl edit deployment hello
```
and make,
```
...
containers:
- name: hello
  image: kelseyhightower/hello:2.0.0
...
```
and see
```
$ kubectl get replicaset
NAME                  DESIRED   CURRENT   READY   AGE
auth-6bb8dcd7bd       1         1         1       11m
frontend-77f46bf858   1         1         1       6m25s
hello-5cbf94fc49      2         2         2       6m44s
hello-677685c76       2         2         0       6s

$ kubectl rollout history deployment/hello
deployment.extensions/hello
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

```
Pause it
```
kubectl rollout pause deployment/hello
$ kubectl rollout status deployment/hello
deployment "hello" successfully rolled out

$ kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
auth-6bb8dcd7bd-js66t           kelseyhightower/auth:1.0.0
frontend-77f46bf858-mh9rl               nginx:1.9.14
hello-677685c76-9b4c8           kelseyhightower/hello:2.0.0
hello-677685c76-nd879           kelseyhightower/hello:2.0.0
hello-677685c76-rpn95           kelseyhightower/hello:2.0.0
```
Resume,
```
kubectl rollout resume deployment/hello
```

### Rollback an update
```
kubectl rollout undo deployment/hello
$ kubectl rollout history deployment/hello
deployment.extensions/hello
REVISION  CHANGE-CAUSE
2         <none>
3         <none>

$ kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
```
### Canary deployments
edit version to 1.0.0
```
cat deployments/hello-canary.yaml
kubectl create -f deployments/hello-canary.yaml
$kubectl get deployments
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
auth           1         1         1            1           15m
frontend       1         1         1            1           11m
hello          3         3         3            3           11m
hello-canary   1         1         1            1           29s
```
Verify 
```
$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
{"version":"1.0.0"}
```
### Canary deployments in production - session affinity

### Blue-green deployments
https://cdn.qwiklabs.com/POW8Q247ZKNY%2ByHIartCsoEu8MAih7k4u1twusCx6pw%3D
```
kubectl apply -f services/hello-blue.yaml
```
Updating using Blue-Green Deployment
```
$ kubectl create -f deployments/hello-green.yaml
$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
{"version":"1.0.0"}
```
Now, update the service to point to the new version:
```
$ kubectl apply -f services/hello-green.yaml
service/hello configured
```
With the service is updated, the "green" deployment will be used immediately. You can now verify that the new version is always being used.
```
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
```
#### Blue-Green Rollback
```
kubectl apply -f services/hello-blue.yaml
```
Once you have updated the service, your rollback will have been successful. Again, verify that the right version is now being used:
```
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
```


## Continuous Delivery with Jenkins in Kubernetes Engine
https://cdn.qwiklabs.com/1b%2B9D20QnfRjAF8c6xlXmexot7TDcOsYzsRwp%2FH4ErE%3D
https://cloud.google.com/solutions/jenkins-on-container-engine

### What is Kubernetes Engine?
Kubernetes Engine is GCP's hosted version of Kubernetes - a powerful cluster manager and orchestration system for containers. Kubernetes is an open source project that can run on many different environments—from laptops to high-availability multi-node clusters; from virtual machines to bare metal. As mentioned before, Kubernetes apps are built on containers - these are lightweight applications bundled with all the necessary dependencies and libraries to run them. This underlying structure makes Kubernetes applications highly available, secure, and quick to deploy—an ideal framework for cloud developers.

### What is Jenkins?
Jenkins is an open-source automation server that lets you flexibly orchestrate your build, test, and deployment pipelines. Jenkins allows developers to iterate quickly on projects without worrying about overhead issues that can stem from continuous delivery.


Set location 
gcloud config set compute/zone us-central1-f

Then clone the lab's sample code into your Cloud Shell:

git clone https://github.com/GoogleCloudPlatform/continuous-deployment-on-kubernetes.git

Now change to the correct directory:

cd continuous-deployment-on-kubernetes

Create a Kubernetes Cluster 
```
gcloud container clusters create jenkins-cd \
--num-nodes 2 \
--machine-type n1-standard-2 \
--scopes "https://www.googleapis.com/auth/projecthosting,cloud-platform"
```
Check cluter
```
$ gcloud container clusters list
NAME        LOCATION       MASTER_VERSION  MASTER_IP      MACHINE_TYPE   NODE_VERSION   NUM_NODES  STATUS
jenkins-cd  us-central1-f  1.12.8-gke.10   35.226.20.249  n1-standard-2  1.12.8-gke.10  2          RUNNING
```

Get Credentials
```
$ gcloud container clusters get-credentials jenkins-cd

Fetching cluster endpoint and auth data.
kubeconfig entry generated for jenkins-cd.

$ kubectl cluster-info
Kubernetes master is running at https://35.226.20.249
GLBCDefaultBackend is running at https://35.226.20.249/api/v1/namespaces/kube-system/services/default-http-backend:http/proxy
Heapster is running at https://35.226.20.249/api/v1/namespaces/kube-system/services/heapster/proxy
KubeDNS is running at https://35.226.20.249/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://35.226.20.249/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

```


### Install Helm
```
wget https://storage.googleapis.com/kubernetes-helm/helm-v2.14.1-linux-amd64.tar.gz
tar zxfv helm-v2.14.1-linux-amd64.tar.gz
cp linux-amd64/helm .
```

Add yourself as a cluster administrator in the cluster's RBAC so that you can give Jenkins permissions in the cluster:
```
kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value account)
```
Grant Tiller, the server side of Helm, the cluster-admin role in your cluster:
```
kubectl create serviceaccount tiller --namespace kube-system
kubectl create clusterrolebinding tiller-admin-binding --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
```
Initialize Helm. This ensures that the server side of Helm (Tiller) is properly installed in your cluster.
```
./helm init --service-account=tiller
./helm update

./helm version
```

### Configure and Install Jenkins
Use the Helm CLI to deploy the chart with your configuration settings.
```
./helm install -n cd stable/jenkins -f jenkins/values.yaml --version 1.2.2 --wait
```
Check pods
```
$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
cd-jenkins-546f5559b4-cxb4c   1/1     Running   0          2m14s
```
Configure the Jenkins service account to be able to deploy to the cluster.
```
kubectl create clusterrolebinding jenkins-deploy --clusterrole=cluster-admin --serviceaccount=default:cd-jenkins
```
Setup port forwarding
```
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=cd" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &
```
Check services
```
$ kubectl get svc
NAME               TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)     AGE
cd-jenkins         ClusterIP   10.0.14.213   <none>        8080/TCP    3m51s
cd-jenkins-agent   ClusterIP   10.0.9.81     <none>        50000/TCP   3m51s
kubernetes         ClusterIP   10.0.0.1      <none>        443/TCP     24m
```
*Notice that this service exposes ports 8080 and 50000 for any pods that match the selector. This will expose the Jenkins web UI and builder/agent registration ports within the Kubernetes cluster. Additionally, the jenkins-ui services is exposed using a ClusterIP so that it is not accessible from outside the cluster.*

#### Connect to Jenkins
Get the token
```
printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
```

To get to the Jenkins user interface, click on the Web Preview button in cloud shell, then click “Preview on port 8080”:

#### Understanding the Application
https://cdn.qwiklabs.com/P1T5JBWWprA4iLf%2FB5%2BO6as7otLE25YBde57gzZwSz4%3D

#### Deploying the Application
- Production: The live site that your users access.
- Canary: A smaller-capacity site that receives only a percentage of your user traffic. Use this environment to validate your software with live traffic before it's released to all of your users.


```
$ cd sample-app
$ kubectl create ns production
$ kubectl apply -f k8s/production -n production
$ kubectl apply -f k8s/canary -n production
$ kubectl apply -f k8s/services -n production
```

Scale up the production environment frontends by running the following command:
```
$ kubectl scale deployment gceme-frontend-production -n production --replicas 4
$ kubectl get pods -n production -l app=gceme -l role=frontend
NAME                                         READY   STATUS    RESTARTS   AGE
gceme-frontend-canary-84cc88cccf-sx6lw       1/1     Running   0          64s
gceme-frontend-production-5df96c664d-ldqgd   1/1     Running   0          10s
gceme-frontend-production-5df96c664d-r4mfl   1/1     Running   0          10s
gceme-frontend-production-5df96c664d-vp7g6   1/1     Running   0          77s
gceme-frontend-production-5df96c664d-xv55j   1/1     Running   0          10s

$ kubectl get pods -n production -l app=gceme -l role=backend
NAME                                       READY   STATUS    RESTARTS   AGE
gceme-backend-canary-688b9c69d9-rc6m6      1/1     Running   0          84s
gceme-backend-production-d6559978d-cpkvg   1/1     Running   0          97s

$ kubectl get service gceme-frontend -n production
NAME             TYPE           CLUSTER-IP   EXTERNAL-IP     PORT(S)        AGE
gceme-frontend   LoadBalancer   10.0.10.69   35.238.66.129   80:31024/TCP   74s
```
Paste External IP into a browser to see the info card displayed on a card—you should get a similar page:
https://cdn.qwiklabs.com/dfXfZfxRk0UMSTZm8FyIaLFsHla7AlUmxHO70UTOFjk%3D

Now, store the frontend service load balancer IP in an environment variable for use later:
```
export FRONTEND_SERVICE_IP=$(kubectl get -o jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)

curl http://$FRONTEND_SERVICE_IP/version
```

### Creating the Jenkins Pipeline
Create a copy of the gceme sample app and push it to a Cloud Source Repository:
```
gcloud source repos create default
git init
git config credential.helper gcloud.sh
git remote add origin https://source.developers.google.com/p/$DEVSHELL_PROJECT_ID/r/default

git config --global user.email "[EMAIL_ADDRESS]"
git config --global user.name "[USERNAME]"

git add .
git commit -m "Initial commit"
git push origin master
```
#### Adding your service account credentials
Step 1: In the Jenkins user interface, click Credentials in the left navigation.

Step 2: Click Jenkins stored_scoped_cred.png

Step 3: Click Global credentials (unrestricted).

Step 4: Click Add Credentials in the left navigation.

Step 5: Select Google Service Account from metadata from the Kind drop-down and click OK.

The global credentials has been added. The name of the credential is the GCP Project ID found in the CONNECTION DETAILS section of the lab.

global_cred.png
#### Creating the Jenkins job

Navigate to your Jenkins user interface and follow these steps to configure a Pipeline job.

Step 1: Click Jenkins > New Item in the left navigation:

86e92964a4599231.png

Step 2: Name the project sample-app, then choose the Multibranch Pipeline option and click OK.

Step 3: On the next page, in the Branch Sources section, click Add Source and select git.

Step 4: Paste the HTTPS clone URL of your sample-app repo in Cloud Source Repositories into the Project Repository field. Replace [PROJECT_ID] with your GCP Project ID:

https://source.developers.google.com/p/[PROJECT_ID]/r/default

Step 5: From the Credentials drop-down, select the name of the credentials you created when adding your service account in the previous steps.

Step 6: Under Scan Multibranch Pipeline Triggers section, check the Periodically if not otherwise run box and set the Interval value to 1 minute.

Step 7: Your job configuration should look like this:

general_tab_jenkins.png

branch_source_jenkins.png

build_conf_jenkins.png

Step 8: Click Save leaving all other options with their defaults

After you complete these steps, a job named "Branch indexing" runs. This meta-job identifies the branches in your repository and ensures changes haven't occurred in existing branches. If you click sample-app in the top left, the master job should be seen.

Note: The first run of the master job might fail until you make a few code changes in the next step.

You have successfully created a Jenkins pipeline! Next, you'll create the development environment for continuous integration.

### Creating the Development Environment

#### Creating a development branch
```
git checkout -b new-feature
```
Edit Jenkinsfile and add project ID.

#### Modify the site
o demonstrate changing the application, you will change the gceme cards from blue to orange.

Open html.go:
```
vi html.go
```
Change the two instances of <div class="card blue"> with following:
```
<div class="card orange">
```
Open main.go and change version
```
### Kick off Deployment
Commit and push your changes:
```
git add Jenkinsfile html.go main.go

git commit -m "Version 2.0.0"

git push origin new-feature
```
kubectl proxy &
curl \
http://localhost:8001/api/v1/namespaces/new-feature/services/gceme-frontend:80/proxy/version

### Deploying a Canary Release
Create a canary branch and push it to the Git server:

git checkout -b canary

git push origin canary

In Jenkins, you should see the canary pipeline has kicked off. Once complete, you can check the service URL to ensure that some of the traffic is being served by your new version. You should see about 1 in 5 requests (in no particular order) returning version 2.0.0.

export FRONTEND_SERVICE_IP=$(kubectl get -o \
jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)
```
while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1; done

### Deploying to production
Create a canary branch and push it to the Git server:
```
git checkout master

git merge canary

git push origin master
```
In Jenkins, you should see the master pipeline has kicked off. Once complete, you can check the service URL to ensure that all of the traffic is being served by your new version, 2.0.0.
```
export FRONTEND_SERVICE_IP=$(kubectl get -o \
jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)

while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1; done
```
```
$ kubectl get service gceme-frontend -n production
NAME             TYPE           CLUSTER-IP   EXTERNAL-IP     PORT(S)        AGE
gceme-frontend   LoadBalancer   10.0.10.69   35.238.66.129   80:31024/TCP   31m

```

