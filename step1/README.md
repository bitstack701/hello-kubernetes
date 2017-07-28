# Getting started

Install and start a node app by following this tutorial

https://kubernetes.io/docs/tutorials/stateless-application/hello-minikube/

## Tutorial Summary

### Installation

Install Minikube, the xhyve driver and kubectrl. I don't know what any of these this are yet. 

#### Install Minikube
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64 && \
  chmod +x minikube && \
  sudo mv minikube /usr/local/bin/
```

#### Use Homebrew to install the xhyve driver and set its permissions
```
brew install docker-machine-driver-xhyve
sudo chown root:wheel $(brew --prefix)/opt/docker-machine-driver-xhyve/bin/docker-machine-driver-xhyve
sudo chmod u+s $(brew --prefix)/opt/docker-machine-driver-xhyve/bin/docker-machine-driver-xhyve
```

#### Install kubectl
```
curl -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/v1.6.4/bin/darwin/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

#### Start miniKube
```
minikube start --vm-driver=xhyve
```

### Create a node app

```
mkdir hellonode
touch server.js
```

```
# put this into the hellonode/server.js
const http = require('http');

const handleRequest = function(request, response) {
  console.log('Received request for URL: ' + request.url);
  response.writeHead(200);
  response.end('Hello World!');
};

const www = http.createServer(handleRequest);
www.listen(8080);
```

Test the node server ```node hellonode/server.js``` then browse to http://localhost:8080


### Create a docker container image

Inside the hellonode directory create a file called Dockerfile with the following contents
```
# contents of the Dockerfile
FROM node:7.9.0
EXPOSE 8080
COPY server.js .
CMD node server.js
```

Make sure that minikube vm is running ```minikube start --vm-driver=xhyve```
And then set docker registry to minikube ```eval $(minikube docker-env)```

To undo this run later ```eval $(minikube docker-env -u)```

Now we are ready to build our docker image, from the hellonode directory run  
```docker build -t hello-node:v1 .```


Now the Minikube VM can run the image you built.

### Create a deployment

```kubectl run hello-node --image=hello-node:v1 --port=8080```

view the deployments with ```kubectl get deployments```
view the pods with ```kubectl get pods```
view the events with ```kubectl get events```

[docs for kubectl commands](https://kubernetes.io/docs/user-guide/kubectl-overview/)

### Create a service

By default, the Pod is only accessible by its internal IP address within the Kubernetes cluster. To make the hello-node Container accessible from outside the Kubernetes virtual network, you have to expose the Pod as a Kubernetes Service.

From your development machine, you can expose the Pod to the public internet using the kubectl expose command:

```kubectl expose deployment hello-node --type=LoadBalancer```

To open a browser to your service run ```minikube service hello-node```

### Update the app

Make a code change

Rebuild the image ```docker build -t hello-node:v2 .```

Update the image of your deployment ```kubectl set image deployment/hello-node hello-node=hello-node:v2```

### Clean up

```
kubectl delete service hello-node
kubectl delete deployment hello-node
minikube stop 
```

