# kubernetes-notes

From [a-kubernetes-quick-start-for-people-who-know-just-enough-about-docker-to-get-by](https://blog.sourcerer.io/a-kubernetes-quick-start-for-people-who-know-just-enough-about-docker-to-get-by-71c5933b4633)

 Configuration file: /var/lib/kubelet/config.yaml

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

# Deploy nginx

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/controllers/nginx-deployment.yaml
```

# Install Nginx ingress controller

To load balance nginxes:

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml
```

```
kubectl apply -f nginx-service.yml
```

```
kind: Service
apiVersion: v1
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  externalTrafficPolicy: Local
  type: LoadBalancer
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: http
    - name: https
      port: 443
      protocol: TCP
      targetPort: https
```

Install ingress

```
kubectl apply -f nginx-ingress.yml
```

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-ubuntu
  namespace: ingress-nginx
  annotations:
    # use the shared ingress-nginx
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: ubuntu.sam
    http:
      paths:
      - path: /
        backend:
          serviceName: ingress-nginx
          servicePort: 80
---
```

Open up the nginx deployments:
```
kubectl expose deployment nginx-deployment --external-ip 10.0.2.15 --type LoadBalancer --port 8081 --target-port 80
```

# Configure haproxy ingress

```
kubectl apply -f https://raw.githubusercontent.com/haproxytech/kubernetes-ingress/master/deploy/haproxy-ingress.yaml
```

Service
```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: web
  annotations:
    haproxy.org/load-balance: "leastconn"
    haproxy.org/pod-maxconn: "50"
spec:
  selector:
    app: nginx
  ports:
  - name: port-1
    port: 9092
    protocol: TCP
    targetPort: 80
```

Ingress

```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: web-ingress
  namespace: default
spec:
  rules:
  - host: ubuntu.sam
    http:
      paths:
      - path: /
        backend:
          serviceName: web
          servicePort: 80   
```


# Help my docker containers have disappeared after a restart

Try turning swap off again if you haven't done it permanently.

```
sudo swapoff -a
```

# Remember to hold back your versions of kubectl

https://stackoverflow.com/questions/59291108/worker-start-to-fail-csinodeifo-error-updating-csinode-annotation

```
sudo apt-mark hold kubelet kubeadm kubectl
```

https://github.com/kubernetes/kubernetes/issues/86094
