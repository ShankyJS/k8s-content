# KubeCOnfigs

A Kubernetes configuration file, or kubeconfig, is a file that stores information about clusters, users, ns, and authentication mechanisms. It contains the configuration data needed to connect to and interact with one or more K8s clusters.

More info: (k8s.io)[https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/]

Kubeconfig contains information such as:

- THe location of the cluster you want to connect to.
- What user you want to authenticate as.
- Data needed in order to authenticate, such as tokens or client certificates.

You can define multiple contexts in a kubeconfig file, allowing you to easily switch between multiple clusters.

## Why do we need Kubeconfigs?

We use kubeconfigs to store the configuration data that will allow the many components of K8s to connect to and interact with the K8s cluster.

How will t he kubelet service on one of our worker nodes know how to locate the Kubernetes API and authenticate with it? ***It willl use a kubeconfig.***

### Generating a K8s Kubeconfig for the Cluster.

The first step is to generate kubeconfigs, for this we need to set up a environment variable to store the address of the K8s API. 

````
KUBERNETES_ADDRESS=<LOAD_BALANCER_PRIVATE IP>
````

Now we can use the command ```kubectl config set-cluster``` to set up the configuration for the location of the cluster.

- Use ```kubectl config set-credentials``` to set the username and client certificate that will be used to authenticate.
- Use ```kubectl config set-context default``` to set up the default context.
- Use ```kubectl config use-context default``` to set the current context to the configuration we provided.

### What Kubeconfigs Do We need to generate?

We will need several kubeconfig files for various components of the Kubernetes cluster:

- Kubelet (one for each worker node)
- Kube-proxy.
- Kube-controller-manager
- Kube-scheduler
- Admin

Create an environment variable to store the address of the Kubernetes API, and set it to the private IP of your load balancer cloud server:

KUBERNETES_ADDRESS=<load balancer private ip>

Generate a kubelet kubeconfig for each worker node:

````
for instance in jrab66924c.mylabserver.com jrab66923c.mylabserver.com ; do
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_ADDRESS}:6443 \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-credentials system:node:${instance} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:${instance} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=${instance}.kubeconfig
done
````

Generate a kube-proxy kubeconfig:

````
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_ADDRESS}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
}
````

Generate a kube-controller-manager kubeconfig:

````
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
}
````

Generate a kube-scheduler kubeconfig:

````
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem \
    --client-key=kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
}

````

Generate an admin kubeconfig:

````
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default --kubeconfig=admin.kubeconfig
}
````

### Distribuiting the Kubeconfig files. 

Here are the commands used in the demo. Be sure to replace the placeholders with the actual values from your cloud servers.

Move kubeconfig files to the worker nodes:

````
scp <worker 1 hostname>.kubeconfig kube-proxy.kubeconfig user@<worker 1 public IP>:~/
scp <worker 2 hostname>.kubeconfig kube-proxy.kubeconfig user@<worker 2 public IP>:~/
````

Move kubeconfig files to the controller nodes:

````
scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig user@<controller 1 public IP>:~/
scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig user@<controller 2 public IP>:~/
````
