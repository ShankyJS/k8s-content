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
