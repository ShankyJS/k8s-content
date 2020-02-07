# k8s-content
Kubernetes Deep Dive and Essentials (Linux Academy)

# Install and prevent update on Docker (First Step to K8s setup)

````
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

sudo apt-get update

sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu

sudo apt-mark hold docker-ce
````

Check it using:

````
sudo docker version
````

# Installing Kubernetes Components (Kubectl, Kubelet and Kubeadm)

***Kubeadmin:*** Automates the process of setting up a K8s Cluster with some useful tooling.
***Kubelet:*** Controles the essential components of K8s to handle the running containers on every node.
***Kubectl:*** Interact with the cluster by a CLI.

````
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update

sudo apt-get install -y kubelet=1.12.7-00 kubeadm=1.12.7-00 kubectl=1.12.7-00

sudo apt-mark hold kubelet kubeadm kubectl
````

```
kubeadm version
```

# Bootstrap the K8s Cluster

On the Kube master node, initialize the cluster

````
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
````

Now we can set it up our local kubectl in the master node to interact with all the nodes.

````
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
````

You can verify your installation with this: 

````
kubectl version
````

The last step is to join all our nodes to the cluster with this command (similar)

````
sudo kubeadm join $some_ip:6443 --token $some_token --discovery-token-ca-cert-hash $some_hash
````

Now if we get the nodes we are going to see something like this:

````
NAME                      STATUS     ROLES    AGE     VERSION
wboyd1c.mylabserver.com   NotReady   master   5m17s   v1.12.2
wboyd2c.mylabserver.com   NotReady   <none>   53s     v1.12.2
wboyd3c.mylabserver.com   NotReady   <none>   31s     v1.12.2
````

Setting Up Flannel (Networking)

Kubernetes supports a lot of Networking Solutions but in this case we are going to use Flannel.
First we have to turn on the bridge of the ip tables between our nodes.

````
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
````

Now we can apply flannel in our master node (just in our master node cause he applys our config in the other 2 nodes).

````
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
````

Now our nodes are up and running.

````
NAME                         STATUS   ROLES    AGE   VERSION
jrab66921c.mylabserver.com   Ready    master   18m   v1.12.7
jrab66922c.mylabserver.com   Ready    <none>   18m   v1.12.7
jrab66923c.mylabserver.com   Ready    <none>   17m   v1.12.7
````

## Containers and Pods

Pods are the smaller and most basic building blocks on K8s, a pod consists of one or more container per pod, storage resources with uniques IP addresses on K8s Cluster Network 

In order to run containers K8s schedules pods to run on servers in the cluster, when that node is Scheduled the server will run the containers that are part of the pod.


## Creating a Simple Nginx Container

````
cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
EOF
````

Now we can do ```kubectl get pods``` to see our running pod (on our default namespace).

And we can describe our pod with ```kubectl describe pod nginx``` or even delete it with ```kubectl delete pod nginx ```

# Custering and Nodes

Kubernetes implements a clustered architecture. In a typical production environment, you will have multiple servers that are able to run your workloads (containers).

These servers which actually run the containers are called nodes or workers.

A Kubernetes cluster has one or more control servers which manage and control the cluster and host by the Kubernetes API. 

These control servers are usually separate from worker nodes, which run applications within the cluster.

````
kubectl get nodes
kubectl describe "node_name"
````
<img src="https://i.imgur.com/nwphQOS.png"/>

# Networking on K8s 
 
When using K8s it's important to know how networking works, in this course I used the K8s flannel plugin to create my VCN on my cluster.
The kubernetes networking model involve create a VN across the cluster, so every pod has a unique IP address and can communicate with any other pod in the cluster.  that's why my pods can communicate each other without be on the same node. 

<img src="https://i.imgur.com/0daZJAu.png" />

````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.4
        ports:
        - containerPort: 80
````

````
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - name: busybox
    image: radial/busyboxplus:curl
    args:
    - sleep
    - "1000"
````

## Kubernetes Architecture and Components.

Kubernetes includes multiple components that work together to provide the funcionability of a K8s cluster.

The control plane components manage and control the cluster: 

- etcd: Provides distributed synchronized data sotrage for the cluster state.

- kube-apiserver: Serves the K8s API, the primary interface for the cluster.

- Kube-controller-manager: Bundles several components into one package.

- Kube-scheduler: Schedules pods to run on individual nodes.

In additionn to the control plane, each node also has:

***kubelet:*** Agent that executes containers on each node.
***Kube-proxy:*** Handles network communcation between nodes by adding firewall routing rules.

With kubeadm, many of these components are run as pods within the cluster itself.


````
kubectl get pods -n kube-system
````

Kubelet is not a pod, is a service running on systemd.

## Deployments: 

Pods are great way to organize and manage containers, but what if I want to spin up and automate multiple pods?

***Deployments:*** Are great way to automate the management of your pods. A deployment allows you to specify a desired state for a set of pods. The cluster will then constantly work to maintain that desired state.

***Scaling:*** With a deployment, you can specify the number of replicas you want, and the deployment will create (or remove) pods to meet that number of replicas.

***Rolling updates:*** With a deployment you can change the deployment container version to a new version of the image but the deployment will gradually replace existing containers with the new version.

***Self healing***: If one of the pods in the deployment is accidentally destroyed, the deployment will inmediately spin up a new one to replace it.


## Services in Kubernetes 

Services are another important component of deploying apps with K8s. 

Services allow you to dynamically access a group of replica pods. Replica pods are often being created and destroyed, so what happens to other pods or external entities which need to access those pods?

A Service creates an ***abstraction layer*** on top of a set of replica pods. You can access the service rather than accessing the pods directly, so as pods come and go, you get uninterrupted, dynamic access to whatever replicas are up at the time.


Cluster:

POD <---------------Service 
POD <----------^
POD <-------^

````
kind: Service
apiVersion: v1
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
  type: NodePort
````
list services.
````
kubectl get svc
````

# Deploy a simple Microservices Application on K8s

````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: store-products
  labels:
    app: store-products
spec:
  replicas: 4
  selector:
    matchLabels:
      app: store-products
  template:
    metadata:
      labels:
        app: store-products
    spec:
      containers:
      - name: store-products
        image: linuxacademycontent/store-products:1.0.0
        ports:
        - containerPort: 80
````

Service

````
kind: Service
apiVersion: v1
metadata:
  name: store-products
spec:
  selector:
    app: store-products
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
````

Executing from busybox pod

````
kubectl exec busybox -- curl -s store-products
````

## What are Microservices?

Microservices are one of those areas where the benefits and features of Kubernetes really shines. Managing microservices applications is not the only use case for Kubernetes, but it's definitely a very powerful use case. So in this lesson, we're going to talk about what our microservices.

Microservices is application architecture, Many applications are designed with a monolithic architecture, meaning that all parts of the application are combined in one large executable.

Monolith Application:
------------------------|
| Auth    Customer Data |
| Product Search        |
| ----------------------|


Microservice Application

Microservice architectures break the application up into several small services, can be made on completely different technologies, languages, can be scalated separately.

|-------------------------|
| Auth |
|-------------------------|
|      | Customer| Product|
|      | Data    | Product|
--------------------------|

Here are few advantages of microservices:

Scalability: Individual microservices are independiently scalable. If your search service is under a large amount of load, you can scale that service by itself, without scaling the whole application.

Cleaner code: When services are relatively independent, it is easier to make a change in one area of the application without breaking things in other areas.

Reliability: Problems in one area of the application are less likely to affect other areas.

Variety of Tools: Different parts of the application can be built using different tools, languages, and frameworks. THis means that the right tool can be used for every job.

Implementing microservices means deploying, scaling and managing a lot of individual components, ***Kubernetes*** is a great tool for accomplishing all of this. In the world of microservices, the benefits of Kubernetes really shine.

## Deploying the Robot Shop App

Kubernetes is a powerful tool for managing and deploying microservice applications. In this lesson, we will deploy a microservice application consisting of a multiple varied components to our cluster. We will also explore the application briefly in order to get a hand on gimpse of what a microservice application might look like, and how it might run in a K8s cluster.


***Clone the Git Repository.***

````
cd ~/
git clone https://github.com/linuxacademy/robot-shop.git
````

*** Create a namespace and deploy the application objects to the namespace using the deployment descriptors from the Git Repository.***

````
kubectl create namespace robot-shop
kubectl -n robot-shop create -f ~/robot-shop/K8s/descriptors/
````

***Get a list of the application's pods and wait for all of them to finish starting up:***

````
kubectl get pods -n robot-shop -w
````

Once all the pods are up, you can access the application in a browser using the public IP of a one of your Kubernetes servers and port 30080:

````
http://$kube_server_public_ip:30080
````

Edit a deployment added:

````
kubectl edit deployment <name> -n <namespace>

````
