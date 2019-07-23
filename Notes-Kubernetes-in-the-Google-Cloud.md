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


