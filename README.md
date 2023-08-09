# Kubernetes Anywhere
Automated Deployment of Lightweight HA Kubernetes on any cloud provider.

Set up Kubernetes clusters on any major cloud service provider. Leveraging the efficiency of k3s, the flexibility of Ansible automation, and the reliability of Longhorn distributed block storage, this repository offers a unified and lightweight Kubernetes environment.

🚀 Key Features:

- Multi-Cloud Freedom: Deploy Kubernetes clusters on AWS EC2, Google Cloud, Azure, and more, without cloud-specific complexities.
- Lightweight Kubernetes (k3s): Enjoy a streamlined Kubernetes distribution designed for efficiency and ease of use.
- Effortless Automation: Utilize Ansible for smooth and reliable cluster provisioning and management.
- Distributed Block Storage: Integrate Longhorn for resilient and scalable block storage tailored for Kubernetes.

## Get started
In this below docs we are using Aws Ec2 (ubuntu 20.04), K3s, Longhorn, Ansible and mac m1 for deployment.

### **Step 1 : Install Ansible and helm on your local machine:-**

```bash
# install ansible locally
python3 -m pip install --user ansible

# install helm3 locally
brew install helm
```

### **Step 2 : Create Load Balancer :-**
Create Ec2 machine as External Load Balancer and optional assign elastic ip and update inbound security group to allow `6443,80,443` ports.

```bash
#install docker
curl -sSL https://get.docker.com |sh
sudo usermod -aG docker $(whoami)
sudo service docker start

# update master node ip's in nginx-lb/nginx.conf and then copy nginx-lb/nginx.conf to /etc/nginx/nginx.conf
touch /etc/nginx/nginx.conf

# run nginx container
docker run -d --restart=unless-stopped  -p 80:80 -p 443:443 -p 6443:6443 -v /etc/nginx.conf:/etc/nginx/nginx.conf  nginx:1.14
```

### **Step 3 : Update ip addresses :-**
Edit master and agent node ipv4 addresses in `inventory/my-cluster/hosts.ini` For example:

```bash
[master]
192.16.35.12

[node]
192.16.35.10
192.16.35.11
```

### **Step 4: Update loadBalancer ip :-**
Update loadBalancer ip address in `inventory/my-cluster/group_vars/all.yml` For example:

```bash
# loadBalancerIp is elastic ip or ipv4 of ec2(that we get from step-2)
apiserver_endpoint: <loadBalancerIp>

```

### **Step 5 : Update your .pem(private key file) path  :-**
Edit path of private_key_file in `ansible.cfg`. For Example :-

```bash
private_key_file = ~/Downloads/secret.pem
```

### **Step 6 : Provisioning of the cluster :-**
Start provisioning of the cluster using the following command:

```bash
sudo ansible-playbook site.yml -i inventory/my-cluster/hosts.ini
```

### **Step 7 : Access kubernetes cluster by downloading config file :-**
Get access to your Kubernetes cluster
```bash
$ scp -i ./secret.pem ubuntu@{master_node_ip}:~/.kube/config ~/.kube/config

# Next edit server url in  ~/.kube/config file and point to loadBalancerIp eg: server: https://{loadBalancerIp}:6443)
```

### **Step 8 :-**
Install longhorn
```bash
$ helm repo add longhorn https://charts.longhorn.io

$ helm repo update

$ helm install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace --version 1.3.0

# next make sure every pod is in healthy state
$ kubectl get po -w -n longhorn-system
```
To get longhorn ui (optional) 

```bash
# ui will be available on http://localhost:8080
kubectl port-forward svc/longhorn-frontend 8080:80 -n longhorn-system
```

###  Remove k3s cluster

```bash
sudo ansible-playbook reset.yml -i inventory/my-cluster/hosts.ini
```

## **Troubleshooting**
if you get a `113: No route to host` in loadbalancer logs, restart node by running below command
```bash
# ssh to specific node
sudo reboot
```

