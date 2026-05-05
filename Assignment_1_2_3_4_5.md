Assignment1
Tasks To Be Performed:
1. Deploy a Kubernetes cluster for 3 nodes
2. Create a NGINX deployment of 3 replicas

Assignment2
Tasks To Be Performed:
1. Use the previous deployment
2. Create a service of type NodePort for NGINX deployment
3. Check the NodePort service on a browser to verify

Assignment3
Tasks To Be Performed:
1. Use the previous deployment
2. Change the replicas to 5 for the deployment

Assignment4
Tasks To Be Performed:
1. Use the previous deployment
2. Change the service type to ClusterIP

Assignment5
Tasks To Be Performed:
1. Use the previous deployment
2. Deploy an NGINX deployment of 3 replicas
3. Create an NGINX service of type ClusterIP
4. Create an ingress service/ Apache to Apache service/ NGINX to NGINX
service
   
### Assignment1 - Solution

## Architecture
- 1 Master Node (EC2 instance => k8-m)
- 2 Worker Nodes (EC2 instances => k8-s1, k8-s2)
  
## 1. Deploy a Kubernetes cluster for 3 nodes

### 1. Launch 3 EC2 Instances and connect 
- Ubuntu 22.04
- t2.medium
- Security Group (IMPORTANT)
- Allow:
    - All Traffic (for simplicity in learning)OR at least:
    - 22 (SSH)
    - 6443 (K8s API)
    - 30000–32767 (NodePort)

### 2. ON MASTER NODE and On slave nodes execute following commands
- Create installation script file => installation.sh
```
sudo apt-get update
sudo apt install apt-transport-https curl -y

## Install containerd

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install containerd.io -y

## Configure containerd

sudo mkdir -p /etc/containerd

sudo containerd config default | sudo tee /etc/containerd/config.toml

sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

sudo systemctl restart containerd

## Install Kubernetes

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update

sudo apt-get install -y kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl

sudo systemctl enable --now kubelet

## Disable Swap
sudo swapoff -a
sudo modprobe br_netfilter
sudo sysctl -w net.ipv4.ip_forward=1

### 3. Execute this file

sudo bash installation.sh
```
### 4. Run following commands only on master node one by one
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

## Configure kubectl
(you will get following commands from output of above commands, execute them simultaneously)

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config


## Install Network Plugin (Flannel)
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

## check which machine is inside k8s cluster
kubectl get nodes (only single node will appear)
```
### 5. Execute on ALL of your Worker Node's
```
## Perform pre-fight checks
sudo kubeadm reset pre-fight checks

## work as root user
sudo su

## Paste the join command you got from the master node, append '--v=5' at the end and replace x values with original one's from master node. avoid using sudo your-token

kubeadm join <MASTER-IP>:6443 --token xxxx --discovery-token-ca-cert-hash sha256:xxxx --v=5
```
### 6. Check on master node if 3 nodes are created or not 
```
kubectl get nodes
```

## 2. Create a NGINX deployment of 3 replicas

### 1. on master machine 
- create deploy.yaml file
  ```
  sudo nano deploy.yaml
  ```
- go to k8s official website for kubernetes deployment, copy the deployment file
   ```
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
      labels:
        app: nginx
    spec:
      replicas: 3
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
            image: nginx:latest
            ports:
            - containerPort: 80

  ```
- apply the changes
  ```
  kubectl apply -f deploy.yaml
  ``` 
- get the deployment details
  ```
  kubectl get deployment
  ```
- get the pods
  ```
  kubectl get pods
  ```
- to check on which slave which pod is running
  ```
  kubectl get pods -o wide 
  ```

### Assignment2 - Solution

## on Master Node
```
sudo nano service.yaml
```
- go to kubernetes official website
- https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport
- paste this code in service.yaml
- use label mentioned inside deployment.yaml to connect deployment to service
  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: my-service
  spec:
    type: NodePort
    selector:
      app: nginx
    ports:
      - port: 80
        # By default and for convenience, the `targetPort` is set to
        # the same value as the `port` field.
        targetPort: 80
        # Optional field
        # By default and for convenience, the Kubernetes control plane
        # will allocate a port from a range (default: 30000-32767)
        nodePort: 30007
  ```
- apply the service
  ```
  kubectl apply -f service.yaml
  ```
- check the service
  ```
  kubectl get services
  ```
- copy the public ip of master machine and paste it on browser with nodeport (http://<Node-IP>:<NodePort>)
- if we did same thing for slave1 and slave2 then same output would appear on browser

### Assignment3 - Solution

## on master node
```
kubectl get deployment
kubectl edit deployment
```
- edit the replicas from 3 to 5 , save the changes and check the changes
```
kubectl get deployment
kubectl get pods
```

### Assignment4 - Solution
## on master node
- check the status of the service
  ```
  kubectl get service
  ```
- edit the service and 
  ```
  kubectl edit service
  ```
- look for
- service having name: my-service(our service name)
- type: Nodeport => change this to ClusterIP
- save the changes
 ```
  kubectl get service
  ```
- to check the updates(cluster exposes the deployment within cluster only)
  ```
  curl <CLUSTER-IP>
  ```



