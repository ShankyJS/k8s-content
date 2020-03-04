# Why do we Need to Enable HTTP Health Checks?

In kelsey Hightower's K8s the hard way he uses a GCP load balancer. And that needs to be able to perform health checks against the KUbernetes API to measure the health status of API Nodes.

The GCP load balancer cannot easily perform health checks over https, so the guide instruct us to set up a proxy server to allow these health checks to be performed over HTTP.

Since we are using NGINX as our load balancer, we don't actually need to do this, but it will be a good practice for us.

```
sudo apt-get install -y nginx
```

Create an nginx configuration for the health check proxy:

````
cat > kubernetes.default.svc.cluster.local << EOF
server {
  listen      80;
  server_name kubernetes.default.svc.cluster.local;

  location /healthz {
     proxy_pass                    https://127.0.0.1:6443/healthz;
     proxy_ssl_trusted_certificate /var/lib/kubernetes/ca.pem;
  }
}
EOF
````
Set up the proxy configuration so that it is loaded by nginx:

````
sudo mv kubernetes.default.svc.cluster.local /etc/nginx/sites-available/kubernetes.default.svc.cluster.local
sudo ln -s /etc/nginx/sites-available/kubernetes.default.svc.cluster.local /etc/nginx/sites-enabled/
sudo systemctl restart nginx
sudo systemctl enable nginx
````

You can verify that everything is working like so:

```
curl -H "Host: kubernetes.default.svc.cluster.local" -i http://127.0.0.1/healthz
```
You should receive a 200 OK response.


## Set up a RBAC for Kubelet Authorization.

Why do we need it?

RBAC: Role based access control.

We need to make sure that the K8s API has permission to access the Kubelet API on each node and perform certain common tasks. Without this, some functionality will not work.

We will create a ClusterROle with the necessary permissions and assign that role to the Kubernetes user with a ClusterRoleBinding

````
cat << EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF
````

````
cat << EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF

````

### Load Balancing NGINX.

Here are the commands you can use to set up the nginx load balancer. Run these on the server that you have designated as your load balancer server:

````
sudo apt-get install -y nginx
sudo systemctl enable nginx
sudo mkdir -p /etc/nginx/tcpconf.d
sudo vi /etc/nginx/nginx.conf
````

Add the following to the end of nginx.conf:

````
include /etc/nginx/tcpconf.d/*;
````

Set up some environment variables for the lead balancer config file:

````
CONTROLLER0_IP=<controller 0 private ip>
CONTROLLER1_IP=<controller 1 private ip>
````

Create the load balancer nginx config file:

````
cat << EOF | sudo tee /etc/nginx/tcpconf.d/kubernetes.conf
stream {
    upstream kubernetes {
        server $CONTROLLER0_IP:6443;
        server $CONTROLLER1_IP:6443;
    }

    server {
        listen 6443;
        listen 443;
        proxy_pass kubernetes;
    }
}
EOF

````

Reload the nginx configuration:

````
sudo nginx -s reload
````

You can verify that the load balancer is working like so:

````
curl -k https://localhost:6443/version
````