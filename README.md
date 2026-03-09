# kubernetes-minikube

Minikube is a tool that lets you run Kubernetes locally. 
minikube runs a single-node Kubernetes cluster on your personal computer (including Windows, macOS and Linux PCs) so that you can try out Kubernetes, or for daily development work.

## Docker installation

### installation for Mac, Windows 10 Pro, Enterprise, or Education

https://www.docker.com/get-started

Choose Docker Desktop

### installation for Windows home

https://docs.docker.com/docker-for-windows/install-windows-home/

## Kuberntes Minikube installation

https://minikube.sigs.k8s.io/docs/start/

Minikube provides a dashboard (web portal). Access the dashboard using the following command:

```
minikube dashboard
```

## Download this project

This project contains a web service coded in Java, but the language doesn't matter. This project has already been built and the binary version is there:

First of all, download and uncompress the project: https://github.com/lilriina/kubernetes-minikube

You can also use git: `git clone https://github.com/lilriina/kubernetes-minikube`

Then move to the sud directory with `cd kubernetes-minikube/myservice` where a DockerFile is.

## Test this project using Docker

Build the docker image:
```
docker build -t myservice .
```

Check the image:
```
docker images
```

Start the container:
```
docker run -p 4000:8080 -t myservice
```

8080 is the port of the web service, while 4000 is the port for accessing the container. Test the web service using a web browser: http://localhost:4000 It displays hello.

Ctrl-C to stop the Web Service.

Check the containerID:
```
docker ps
```

Stop the container:
```
docker stop containerID
```
Exemple: `docker stop 4144cfc6dcbb`

## Publish the image to the Docker Hub

Retreive the image ID:
```
docker images
```

Tag the docker image: 
```
docker tag imageID yourDockerHubName/imageName:version
```
Example: `docker tag fb122716c08c gimenezm/myservice:1`

Login to docker hub: 
```
docker login
```

Push the image to the docker hub:
```
docker push yourDockerHubName/imageName:version
```
Example: `docker push gimenezm/myservice:1`

## Create a kubernetes deployment from a Docker image

```
kubectl get nodes
```
```
kubectl create deployment myservice --image=gimenezm/myservice:1
```

The image used comes from the Docker hub: https://hub.docker.com/r/gimenezm/myservice/tags

Check the pod:
```
kubectl get pods
```

Check if the state is running.

Get complete logs for a pods: 
```
kubectl describe pods
```

Retreive the IP address but notice that this IP address is ephemeral since a pods can be deleted and replaced by a new one.

Then retrieve the deployment in the minikube dashboard. 
Actually the Docker container is runnung inside a Kubernetes pods (look at the pod in the dashboard).
  
You can also enter inside the container in a interactive mode with:
```
kubectl exec -it podname -- /bin/bash
```
Exemple: `kubectl exec -it myservice-746766f779-drpwm -- /bin/bash`

where podname is the name of the pods obtained with:
```
kubectl get pods
```

List the containt of the container with:
```
ls
```

Don't forget to exit the container with:
```
exit
```

## Expose the Deployment through a service

A Kubernetes Service is an abstraction which defines a logical set of Pods running somewhere in the cluster, 
that all provide the same functionality. 
When created, each Service is assigned a unique IP address (also called clusterIP). 
This address is tied to the lifespan of the Service, and will not change while the Service is alive.

## Expose HTTP and HTTPS routes from outside the cluster to services within the cluster

For some parts of your application (for example, frontends) you may want to expose a Service onto an external IP address, that’s outside of your cluster.

Kubernetes ServiceTypes allow you to specify what kind of Service you want. The default is ClusterIP.

Type values and their behaviors are:

* ClusterIP: Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default ServiceType.
* NodePort: Exposes the Service on each Node’s IP at a static port (the NodePort). A ClusterIP Service, to which the NodePort Service routes, is automatically created. You’ll be able to contact the NodePort Service, from outside the cluster, by requesting NodeIP:NodePort.
* LoadBalancer: Exposes the Service externally using a cloud provider’s load balancer. NodePort and ClusterIP Services, to which the external load balancer routes, are automatically created.
* ExternalName: Maps the Service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record

## Expose HTTP and HTTPS route using NodePort

```
kubectl expose deployment myservice --type=NodePort --port=8080
```

Retrieve the service address:
```
minikube service myservice --url
```

This format of this address is `NodeIP:NodePort` (`http://127.0.0.1:58875/`).

Test this address inside your browser. It should display hello again.

Look from the NodeIP and the NodePort in the minikube dashboard.

## Scaling and load balancing

Check if the myservice deployment is running:

```
kubectl get deployments
```

How many instance are actually running:

```
kubectl get pods
```
We can see that for myservice there is 1 deployment ready and there is 1 pods (myservice-746766f779-drpwm).

Start a second instance:

```
kubectl scale --replicas=2 deployment/myservice
```
```
kubectl get deployments
```

and 

```
kubectl get pods
```

again.

We can now see that for myservice there are 2 deployments ready and there are 2 pods (myservice-746766f779-drpwm and myservice-746766f779-hd5kf).

## Creating a Service of type LoadBalancer

Check if the myservice deployment is running:

```
kubectl get deployments
```

If a service is running in front of the deployment you must delete this service first in ordre to create a new one of kind LoadBalancer. So retreive the service using:

```
kubectl get services
```
And delete it:
```
kubectl delete service serviceName
```
Exemple: `kubectl delete service myservice`

```
kubectl expose deployment myservice --type=LoadBalancer --port=8080
```
```
minikube service myservice --url
```
`http://127.0.0.1:63428/`

Test in your web browser


## Creating version 2

```docker build -t myservice:2 .```
```docker images```
```docker tag 100cc3d951db gimenezm/myservice:2```
```docker push gimenezm/myservice:2```


## Rolling updates

Rolling updates allow Deployments' update to take place with zero downtime by incrementally updating Pods instances with new ones.

To update the image of the application to version 2, use the set image subcommand, followed by the deployment name and the new image version:
```
kubectl set image deployments/my-deployment my-deployment=dockerHudId/my-image:v2
```
Exemple: `kubectl set image deployments/myservice myservice=gimenezm/myservice:2`

You can also confirm the update by running the rollout status subcommand:
```
kubectl rollout status deployments/my-deployment
```
Exemple: `kubectl rollout status deployments/myservice`

To roll back the deployment to your last working version, use the rollout undo subcommand:
```
kubectl rollout undo deployments/my-deployment
```

## Create a deployment and a service using a yaml file

Yaml files can be used instead of using the command `kubectl create deployment` and `kubectl expose deployment`

The yaml file for the deployment: https://github.com/charroux/kubernetes-minikube/blob/main/myservice-deployment.yml

The yaml file for the node port service: https://github.com/charroux/kubernetes-minikube/blob/main/myservice-service.yml

The yaml file for the node port service: https://github.com/charroux/kubernetes-minikube/blob/main/myservice-loadbalancing-service.yml

Apply the deployment:
```
kubectl apply -f myservice-deployment.yml
```

Apply the node port service: 
```
kubectl apply -f myservice-service.yml
```

or 

Apply the service of type loadbalancer:
```
kubectl apply -f myservice-loadbalancing-service.yml
```
Then test if it works as expected.


# Routing rule to a service using Ingress

You can use Ingress to expose your Service. 
Ingress is not a Service type, but it acts as the entry point for your cluster. 
It lets you consolidate your routing rules into a single resource as it can expose multiple services under the same IP address.
Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. 
An Ingress may be configured to give Services externally-reachable URLs, load balance traffic, terminate SSL / TLS, and offer name-based virtual hosting.

## Set up Ingress on Minikube with the NGINX Ingress Controller

Enable the NGINX Ingress controller: 

```
minikube addons enable ingress
```
Verify that the NGINX Ingress controller is running:
```
kubectl get pods -n ingress-nginx
```

Create a Deployment and expose it as a NodePort (not a loadbalancer).

```kubectl scale --replicas=2 deployment/myservice```
```kubectl get deployments```
```kubectl get services```
```kubectl delete service myservice```
```kubectl expose deployment myservice --type=NodePort --port=8080```
```minikube service myservice --url```
http://127.0.0.1:49678/

Check if it works.

A yaml file for ingress: https://github.com/charroux/kubernetes-minikube/blob/main/ingress.yml

```
kubectl apply -f ingress.yml
```

Retrieve the IP address of Ingress: 

```
kubectl get ingress
```

Exemple:
```
NAME                 CLASS    HOSTS                  ADDRESS        PORTS   AGE

example-ingress      nginx   myservice.info         192.168.49.2   80      2m21s
```

On Windows : edit the `c:\windows\system32\drivers\etc\hosts` file, add 

`127.0.0.1 myservice.info`	

Enable a tunnel for Minikube:

```
minikube addons enable ingress-dns
```
```
minikube tunnel
```

Then check in your Web browser: 

http://myservice.info/


Create a second deployment and its service, then add a new route to the ingress.yml file.

```docker build -t newservice .```
```docker tag 14df09db94f5 gimenezm/myservice:3```
```docker push gimenezm/myservice:3```
```kubectl create deployment newservice --image=gimenezm/myservice:3```
```kubectl scale --replicas=2 deployment/newservice```
```kubectl expose deployment newservice --type=NodePort --port=8081```
```minikube service newservice --url```
http://127.0.0.1:62347/
```kubectl apply -f ingress.yml```
```kubectl get ingress```
```minikube addons enable ingress-dns```
```minikube tunnel```

## Delete resources

```
kubectl delete services myservice
```
```
kubectl delete deployment myservice
```
