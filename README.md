# kubernetes-notes

From [a-kubernetes-quick-start-for-people-who-know-just-enough-about-docker-to-get-by](https://blog.sourcerer.io/a-kubernetes-quick-start-for-people-who-know-just-enough-about-docker-to-get-by-71c5933b4633)

# Install kubectl

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

# Install docker

```
sudo apt-get update
sudo apt-get install -y docker.io
```

```
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl

```

Get local interface IP address
```
ip a
export MASTER_IP=<local ip address>
```


Kubeadm detects 127.0.0.1 and changes it to a local interface IP.

```
kubeadm init --apiserver-advertise-address ${MASTER_IP}
```

Local configuration of Kubectl:

```
cp /etc/kubernetes/admin.conf $HOME/
chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf
```

# Setting up networking

Need a CNI compliant network interface, using weave:

```
sysctl net.bridge.bridge-nf-call-iptables=1
export kubever=$(kubectl version | base64 | tr -d '\n')
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$kubever"
```

For local testing of scheduling pods on master node: You're really not meant to do this.

```
kubectl taint nodes $(hostname) node-role.kubernetes.io/master:NoSchedule-
```
