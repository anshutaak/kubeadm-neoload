# Getting started

### Installing Helm on Ubuntu

```
wget https://get.helm.sh/helm-v3.9.3-linux-amd64.tar.gz

tar xvf helm-v3.9.3-linux-amd64.tar.gz

sudo mv linux-amd64/helm /usr/local/bin

````
# Kubedm cluster setup on one node

### Step 1 : run the below commands

``` 
sudo su 

apt-get update 

```

##### Disable Firewall
```
ufw disable
```
##### Disable swap

```
swapoff -a; sed -i '/swap/d' /etc/fstab
```
##### Update sysctl settings for Kubernetes networking
```
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

```sysctl --system```


### Step 2 : Install Docker

``` apt-get install -y docker.io ```

### Step 3 : Install kubeadm, Kubelet And Kubectl .


kubeadm: the command to bootstrap the cluster.
kubelet: the component that runs on all of the machines in your cluster and does things like starting pods and containers.
kubectl: the command line utility to communicate with your cluster.

``` 
apt-get update && apt-get install -y apt-transport-https curl 

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - 

```

```
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
``` 

```
apt-get update 

apt-get install -y kubelet kubeadm kubectl 

apt-mark hold kubelet kubeadm kubectl 
```


### Step 4 : Let’s start Kubernetes cluster . Before running the required command, note the private ip address of the master virtual machine.

``` kubeadm init --apiserver-advertise-address=<internal-ip-address-of-master-node> --pod-network-cidr=192.168.0.0/16  ```

### step 5: Untaint node

``` 
kubectl taint nodes --all node-role.kubernetes.io/master- 

kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

### step 5: Install Calico

```  
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.2/manifests/tigera-operator.yaml 

kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.2/manifests/custom-resources.yaml  

kubectl get pods -n calico-system 
```

### step 6: Install a CSI driver

```
helm repo add openebs https://openebs.github.io/charts

kubectl create namespace openebs

helm --namespace=openebs install openebs openebs/openebs

```


# Deploy Nginx Ingress Controller in Kubernetes

1. Download Nginx controller deployment for Baremetal:

```
controller_tag=$(curl -s https://api.github.com/repos/kubernetes/ingress-nginx/releases/latest | grep tag_name | cut -d '"' -f 4)

wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/${controller_tag}/deploy/static/provider/baremetal/deploy.yaml

```

2. Rename deployment file:

```
mv deploy.yaml nginx-ingress-controller-deploy.yaml
```

3.Apply Nginx ingress controller manifest deployment file:

```
kubectl apply -f nginx-ingress-controller-deploy.yaml
```


# NeoLoad Web

1. Add the Neotys chart repository or update it if you already had it registered

```
helm repo add neotys https://helm.prod.neotys.com/stable/

helm repo update

```

2. Download and set up your values-custom.yaml file

```
wget https://raw.githubusercontent.com/Neotys-Labs/helm-neoload-web/master/values-custom.yaml

```
3. Create a dedicated namespace

```
kubectl create namespace neoload
```

4. Install with the following command

```
Note: When file is downloaded, we would require to make changes under the file, where we need to add mongodb Hostname, username and password values under values.yaml file . Either we can use mongodb or mongodb atlas
```

```
helm install my-release neotys/nlweb -n neoload -f ./values-custom.yaml

```

