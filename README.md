# K3S-DEMO

## Quick start to set up your environment

Here we will set up environment using k3d to create a cluster in docker.

### Prerequisite

* Docker
* wget

### Install k3d

k3d is a lightweight wrapper to run k3s (Rancher Lab’s minimal Kubernetes distribution) in docker.

k3d makes it very easy to create single- and multi-node k3s clusters in docker, e.g. for local development on Kubernetes.

```sh
# https://k3d.io/#quick-start
wget -q -O - https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash
```

### Create a cluster

```sh
k3d cluster create demo \
  --servers 1 \
  --agents 1

# evaluate your cluster
kubectl get nodes
```

## Deploy a container image for a python microservice

### Build the container

```sh
cd app

docker build -t flask-app:0.1.0 .
```

### Run the container

```sh
docker run --name flask-app -p 5000:5000 flask-app:0.1.0
```

### Invoke the service

```sh
curl http://127.0.0.1:5000
```

### Stop the container

```sh
Control + C followed by

docker rm -f flask-app

```

### Login Docker Hub and push your image to the registry

```sh
docker login --username <username>
# Paste the Access Token as your password
docker build -t docker.io/<username>/flask-app:0.1.1 .
docker push <username>/flask-app:0.1.1

```

## Deploy Real Workloads with the kubernetes API

### Deploy a deployment

```sh
cd kubernetes
kubectl apply -f deployment.yaml 
```

### Deploy a service

```sh
kubectl apply -f service.yaml
```

### Deploy a ingress
Loadbalancing and hostname are provided.

```sh
kubectl apply -f ingress-host.yaml
```

### Access the application

```sh
kubectl port-forward -n kube-system service/traefik 8080:80

# on a new terminal
curl http://127.0.0.1:8080 --header "Host: flask-app.example.com" ;echo
# Hello from K3S
```


## Function at the Edge


### Install OpenFaaS using the arkade tool

arkade provides a portable marketplace for downloading your favourite devops CLIs and installing helm charts, with a single command.

```sh
# https://github.com/alexellis/arkade
curl -sLS https://dl.get-arkade.dev | sh
```

### Install the CLI for OpenFaaS

```sh
arkade get faas-cli

export PATH=$PATH:$HOME/.arkade/bin/

arkade install openfaas
```

Response:
```
To verify that openfaas has started, run:

  kubectl -n openfaas get deployments -l "release=openfaas, app=openfaas"
=======================================================================
= OpenFaaS has been installed.                                        =
=======================================================================

# Get the faas-cli
curl -SLsf https://cli.openfaas.com | sudo sh

# Forward the gateway to your machine
kubectl rollout status -n openfaas deploy/gateway
kubectl port-forward -n openfaas svc/gateway 8080:8080 &

# If basic auth is enabled, you can now log into your gateway:
PASSWORD=$(kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode; echo)
echo -n $PASSWORD | faas-cli login --username admin --password-stdin

faas-cli store deploy figlet
faas-cli list

# For Raspberry Pi
faas-cli store list \
 --platform armhf

faas-cli store deploy figlet \
 --platform armhf

# Find out more at:
# https://github.com/openfaas/faas

Thanks for using arkade!
```

### Clone the example and follow the instructions

* Set up Minio
* Deploy the functions

```sh
git clone https://github.com/alexellis/mqtt-s3-example
```

### Setup Minio

```sh
arkade install minio
arkade get mc

# Forward the minio port to your machine
kubectl port-forward -n default svc/minio 9000:9000 &

# Get the access and secret key to gain access to minio
ACCESSKEY=$(kubectl get secret -n default minio -o jsonpath="{.data.accesskey}" | base64 --decode; echo)
SECRETKEY=$(kubectl get secret -n default minio -o jsonpath="{.data.secretkey}" | base64 --decode; echo)

# Get the Minio Client
arkade get mc

# Add a host
mc config host add minio http://127.0.0.1:9000 $ACCESSKEY $SECRETKEY

# List buckets
mc ls minio

# Find out more at: https://min.io

Thanks for using arkade!
```

### Deploy the functions

Follow the instructions in `mqtt-s3-example` 