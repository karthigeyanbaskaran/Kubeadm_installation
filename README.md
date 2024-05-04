## Prerequisites:

1. Ubuntu instance with 4 GB RAM - Master Node - (with ports open to all traffic)
2. Ubuntu instance with at least 2 GB RAM - Worker Node - (with ports open to all traffic)

## Kubernetes Setup using Kubeadm

## Start - Execute the below commands in both Master/worker nodes

Login to both instances execute the below commands:

```bash
sudo apt-get update -y  && sudo apt-get install apt-transport-https -y
```
```bash
Change to root user
sudo su -
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
```
```bash
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
#Disable swap memory for better performance
swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
#Enable IP tables
#We need to enable IT tables for pod to pod communication.
modprobe br_netfilter
sysctl -p
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```
```bash
#Install Docker on both Master and Worker nodes
apt-get install docker.io -y

#Add ubuntu user to Docker group
usermod -aG docker ubuntu
systemctl restart docker
systemctl enable docker.service

#Type exit to come out of root user.
#Install Kubernetes Modules
sudo apt-get install -y kubelet kubeadm kubectl kubernetes-cni

sudo systemctl daemon-reload
sudo systemctl start kubelet
sudo systemctl enable kubelet.service
sudo systemctl status docker
```
## Initialize Kubeadm on Master Node(only on Master Node)

#Execute the below command as root user to initialize Kubernetes Master node.
```bash
sudo su -
kubeadm init
```

## Now you face error:-
```bash
rm /etc/containerd/config.toml
systemctl restart containerd
kubeadm init

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Now we need weave network to enable dns connections:
```bash
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

#Now execute the below command to see the pods.

kubectl get pods  --all-namespaces
```

## Now login to Worker Node

## Join worker node to Master Node
# The below command will join worker node to master node, execute this a normal user by putting sudo before:
```bash
sudo kubeadm join <master_node_ip>:6443 --token xrvked.s0n9771cd9x8a9oc \
    --discovery-token-ca-cert-hash sha256:288084720b5aad132787665cb73b9c530763cd1cba10e12574b4e97452137b4a
```

## Go to Master and type the below command
```bash
kubectl get nodes
##the above command should display both Master and worker nodes.
```

Links:
```bash
https://www.coachdevops.com/2020/06/how-to-setup-kubernetes-cluster-in.html
https://stackoverflow.com/questions/72504257/i-encountered-when-executing-kubeadm-init-error-issue
https://www.weave.works/docs/net/latest/kubernetes/kube-addon/
```
