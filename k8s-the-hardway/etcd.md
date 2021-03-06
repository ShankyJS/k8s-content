# What is etcd?

Etcd is a distributed key value store that provides a reliable way to store data accross a cluster of machines.

Etcd provides a way to store data across a distributed cluster of machines and make sure the data is synchronized across all machines.

## How is etcd used in Kubernetes?

Kubernetes uses etcd to store all of its internal data about cluster state.
THis data needs to be stored, but it also needs to be reliably syncrhonized accross all controller nodes i nthe cluster. etcd fulfills that purpose. 

We will need to install etcd on each of our kubernetes controller nodes and create an etcd cluster that includes all of those controller nodes

## Installation. 

Here are the commands used in the demo (note that these have to be run on both controller servers, with a few differences between them):

````
wget -q --show-progress --https-only --timestamping \
  "https://github.com/coreos/etcd/releases/download/v3.3.5/etcd-v3.3.5-linux-amd64.tar.gz"
tar -xvf etcd-v3.3.5-linux-amd64.tar.gz
sudo mv etcd-v3.3.5-linux-amd64/etcd* /usr/local/bin/
sudo mkdir -p /etc/etcd /var/lib/etcd
sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
````

Set up the following environment variables. Be sure you replace all of the <placeholder values> with their corresponding real values:

````
ETCD_NAME=<cloud server hostname>
INTERNAL_IP=$(curl http://169.254.169.254/latest/meta-data/local-ipv4)
INITIAL_CLUSTER=<controller 1 hostname>=https://<controller 1 private ip>:2380,<controller 2 hostname>=https://<controller 2 private ip>:2380
````

Create the systemd unit file for etcd using this command. Note that this command uses the environment variables that were set earlier:

````
cat << EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster ${INITIAL_CLUSTER} \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

````

Start and enable the etcd service:

````
sudo systemctl daemon-reload
sudo systemctl enable etcd
sudo systemctl start etcd
````
You can verify that the etcd service started up successfully like so:

````
sudo systemctl status etcd
````

Use this command to verify that etcd is working correctly. The output should list your two etcd nodes:

````
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
````
