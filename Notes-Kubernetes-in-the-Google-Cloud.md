# Kubernetes in the Google Cloud

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
    

