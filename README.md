# Overview

Q: So, Ben, what's going on here?

A: Great question. We've got a number of resources here that we're using to build a basic flask app using Docker, and deploying it using Kubernetes. The core application logic is stored in `app.py`, which contains a greedy change-making algorithm, as well as the Flask routing logic. We also have a test file, `test_app.py` that is designed to test the logic in `app.py`.

Before building a new Docker image, we'll want to set up our environment using `venv`. Once that's setup, we can leverage the `Makefile`'s helpful functions by running `make all`. This will install the requirements from `requirements.txt`, lint the Dockerfile (with `hadolint`) as well as the application code, and test `app.py` using `pytest`. 

Great! Now we know our application code is working, so we can build a Docker image and run it in a container. Luckily, this logic is also built into the `Makefile`. But let's write it out since we're trying to learn a thing or two about Docker and k8s. The following code will create a new docker image based on the `Dockerfile` defined within the repo:

```bash
docker build -t flask-change:latest .
```

The stuff after `-t` tells docker that we want to name the image `flask-change` and tag it as `latest`, and we want to build the image from the current directory (`.`). We can now use this new image to run our application logic in a container to make sure it's working, before deploying it into a k8s cluster:

```bash
docker run -p 8080:8080 flask-change
```

The `-p` stuff above tells docker to bind the container's port 8080 to the (local)host's port 8080. If all is working corectly, you should be able to test this by curling the localhost and trying it out. The following code will call the app running in the container and ask it to make change for $1.56. 

```
curl 127.0.0.1:8080/change/1/56
```

Great, everything is running. Now we can move onto the fun stuff - kubernetes. First make sure k8s is running locally by running

```
kubectl get nodes
```

You should see the `docker-desktop` node running. Now, all of the hard stuff has been figured out for you in the `kube-hello-change.yaml` file in the repository, which defines how the cluster should be set up. One of the key takeawawys is that the build spec is going to set up 3 pods within the `docker-desktop` node, and it's going to expose the application runinng in parallel across those pods through the (local)host's port 8080. To get everything up and running, you can run `make run-kube` (that helpful `Makefile` again!), or you can act like you know and run

```
kubectl apply -f kube-hello-change.yaml
```

You should get a note that both the service and the deployment are running. To see the pods running, you can run

```
kubectl get pods
```

You should see three bright and shiny pods smiling back at you. To get more information about the service that's running you can run 

```
kubectl describe services hello-flask-change-service
```

This will confirm what you expected: that you can now access the application yet again through `localhost:8080`. The difference is that now you have your application running across three different pods in a dedicated k8s cluster, which is being dynamically load-balances by the k8s controller! Pretty. Cool. Stuff.

# Kubernetes Hello World
A Kubernetes Hello World Project for Python Flask.  This project uses [a simple Flask app that returns correct change](https://github.com/noahgift/flask-change-microservice) as the base project and converts it to Kubernetes.
![kubernetes-load-balanced-cluster](https://user-images.githubusercontent.com/58792/111511557-3f45a280-8725-11eb-8e4a-5f5ef787796d.png)

This recipe is in the book Practical MLOps.

![9781098103002](https://user-images.githubusercontent.com/58792/111000927-eb1b7680-8350-11eb-8e24-d41064590fc1.jpeg)


## Assets in Repo

* `Makefile`:  [Builds project](https://github.com/noahgift/kubernetes-hello-world-python-flask/blob/main/Makefile)
* `Dockerfile`:  [Container configuration](https://github.com/noahgift/kubernetes-hello-world-python-flask/blob/main/Dockerfile)
* `app.py`:  [Flask app](https://github.com/noahgift/kubernetes-hello-world-python-flask/blob/main/app.py)
* `kube-hello-change.yaml`: [Kubernetes YAML Config](https://github.com/noahgift/kubernetes-hello-world-python-flask/blob/main/kube-hello-change.yaml)

## Get Started

* Create Python virtual environment `python3 -m venv ~/.kube-hello && source ~/.kube-hello/bin/activate`
* Run `make all` to install python libraries, lint project, including `Dockerfile` and run tests

## Build and Run Docker Container

* Install [Docker Desktop](https://www.docker.com/products/docker-desktop)

* To build the image locally do the following.

`docker build -t flask-change:latest .` or run `make build` which has the same command.

* To verify container run `docker image ls`

* To run do the following:  `docker run -p 8080:8080 flask-change` or run `make run` which has the same command

* In a separate terminal invoke the web service via curl, or run `make invoke` which has the same command 

`curl http://127.0.0.1:8080/change/1/34`

```bash
[
  {
    "5": "quarters"
  }, 
  {
    "1": "nickels"
  }, 
  {
    "4": "pennies"
  }
]
```

* Stop the running docker container by using `control-c` command

## Running Kubernetes Locally

* Verify Kubernetes is working via docker-desktop context

```bash
(.kube-hello) ➜  kubernetes-hello-world-python-flask git:(main) kubectl get nodes
NAME             STATUS   ROLES    AGE   VERSION
docker-desktop   Ready    master   30d   v1.19.3
```

* Run the application in Kubernetes using the following command which tells Kubernetes to setup the load balanced service and run it:  

`kubectl apply -f kube-hello-change.yaml` or run `make run-kube` which has the same command

You can see from the config file that a load-balancer along with three nodes is the configured application.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-flask-change-service
spec:
  selector:
    app: hello-python
  ports:
  - protocol: "TCP"
    port: 8080
    targetPort: 8080
  type: LoadBalancer

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-python
spec:
  selector:
    matchLabels:
      app: hello-python
  replicas: 3
  template:
    metadata:
      labels:
        app: hello-python
    spec:
      containers:
      - name: flask-change
        image: flask-change:latest
        imagePullPolicy: Never
        ports:
        - containerPort: 8080
```

* Verify the container is running

`kubectl get pods`

Here is the output:

```bash
NAME                            READY   STATUS    RESTARTS   AGE
flask-change-7b7d7f467b-26htf   1/1     Running   0          8s
flask-change-7b7d7f467b-fh6df   1/1     Running   0          7s
flask-change-7b7d7f467b-fpsxr   1/1     Running   0          6s
```

* Describe the load balanced service:

`kubectl describe services hello-python-service`

You should see output similar to this:

```bash
Name:                     hello-python-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=hello-python
Type:                     LoadBalancer
IP Families:              <none>
IP:                       10.101.140.123
IPs:                      <none>
LoadBalancer Ingress:     localhost
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30301/TCP
Endpoints:                10.1.0.27:8080,10.1.0.28:8080,10.1.0.29:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

Invoke the endpoint to curl it:  

`make invoke`

```bash
curl http://127.0.0.1:8080/change/1/34
[
  {
    "5": "quarters"
  }, 
  {
    "1": "nickels"
  }, 
  {
    "4": "pennies"
  }
]
```

To cleanup the deployment do the following: `kubectl delete deployment hello-python`

## References

* Azure [Kubernetes deployment strategy](https://azure.microsoft.com/en-us/overview/kubernetes-deployment-strategy/)
* Service [Cluster Config](https://kubernetes.io/docs/tasks/access-application-cluster/service-access-application-cluster/) YAML file
* [Kubernetes.io Hello World](https://kubernetes.io/blog/2019/07/23/get-started-with-kubernetes-using-python/)
