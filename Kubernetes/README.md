# Kubernetes
## Deployment
> bare metal: CentOS 7
### 1. prepare
> @ all nodes
#### 1.1 disable firewall and selinux
```bash
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
sed -i '/^SELINUX=/c SELINUX=disabled' /etc/selinux/config
```

#### 1.2
```bash
cat >> /etc/hosts << EOF
<master-ip> master
<minion-1-ip> minion-1
<minion-2-ip> minion-2
EOF
```

#### 1.3 install docker
```bash
yum install -y docker
systemctl restart docker
systemctl enable docker
# register local registry:
echo '{ "insecure-registries":["<ip>:<port>"] }' > /etc/docker/daemon.json
```

#### 1.4 kubeadm
```bash
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
EOF

yum install -y kubeadm
systemctl enable kubelet

images=$(kubeadm config images list)
# images=(
#       k8s.gcr.io/kube-apiserver:v1.17.0
#       k8s.gcr.io/kube-controller-manager:v1.17.0
#       k8s.gcr.io/kube-scheduler:v1.17.0
#       k8s.gcr.io/kube-proxy:v1.17.0
#       k8s.gcr.io/pause:3.1
#       k8s.gcr.io/etcd:3.4.3-0
#       k8s.gcr.io/coredns:1.6.5
# )
# registry=gcr.azk8s.cn/google_containers
registry=registry.cn-hangzhou.aliyuncs.com/google_containers
for image in ${images[@]} ; do
        echo "handling ${image}"
        imageName=${image#*/}
        docker pull ${registry}/${imageName}
        docker tag ${registry}/${imageName} k8s.gcr.io/${imageName}
        docker rmi ${registry}/${imageName}
done

```

### 2. kubeadm
> kubeadm will do some check before, you may need these tips:
```bash
# disable swap
swapoff -a
sed -i '/swap/s/^\//# \//g' /etc/fstab
free -m

# enable iptable bridge
echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
echo net.bridge.bridge-nf-call-iptables = 1 >> /etc/sysctl.conf

echo 1 > /proc/sys/net/ipv4/ip_forward
```

#### 2.1 kubeadm init
> @ master
```bash
hostnamectl set-hostname master

# you should specify --pod-network-cidr=10.244.0.0/16 in case failure of the following flannel installation
# --kubernetes-version is optional, default version is equal to the version of kubeadm
kubeadm init --apiserver-advertise-address=<master_ip> --kubernetes-version=v1.13.0 --pod-network-cidr=10.244.0.0/16

wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
# pull image from quay.azk8s.cn instead of quay.io
sed -i 's/quay.io\/coreos/quay.azk8s.cn\/coreos/' kube-flannel.yml
kubectl apply -f kube-flannel.yml
```

#### 2.2 kubeadm join
> @ minion
```bash
hostnamectl set-hostname minion-<id>
kubeadm join <master_ip>:6443 --token <token> --discovery-token-ca-cert-hash <sha256>
```

### 3. dashboard
#### 3.1 setup
> follow https://github.com/kubernetes/dashboard/

#### 3.2 access
> ref https://www.cnblogs.com/RainingNight/p/deploying-k8s-dashboard-ui.html

##### 3.2.1 prepare key & p12 certification, then import the p12 to web browser
```bash
grep 'client-certificate-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d > kubecfg.crt
grep 'client-key-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d > kubecfg.key
openssl pkcs12 -export -clcerts -inkey kubecfg.key -in kubecfg.crt -out kubecfg.p12 -name "kubernetes-client"
```
##### 3.2.2 get token for browser login
```bash
cat > admin-user.yaml << EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
EOF
kubectl create -f admin-user.yaml

cat > admin-user-role-binding.yaml << EOF
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: admin-user
    namespace: kube-system
    
EOF
kubectl create -f admin-user-role-binding.yaml

# get token
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```
## kubectl
### 1. basic
> ref: https://github.com/alswell/k8s/blob/master/doc/cmd/kubectl.md
### 2. scale
> ref: https://www.kubernetes.org.cn/deployment
```bash
kubectl apply -f spark-slave-deployment.yaml
kubectl scale deployment spark-slave-deployment --replicas 10

kubectl autoscale deployment spark232-slave-deployment --min=1 --max=5 --cpu-percent=80
kubectl get HorizontalPodAutoscaler  # check result

# edit json file, and then
kubectl apply -f spark232-slave-autoscale.json
kubectl get HorizontalPodAutoscaler

```
## RestAPI
### 1. ref
- https://blog.csdn.net/jinzhencs/article/details/51452208
- https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.12/

### 2. scale
```bash
curl -k --cert ./kubecfg.crt --key ./kubecfg.key -X PUT -d@spark232-slave-autoscale.json -H 'Content-Type: application/json' https://localhost:6443/apis/autoscaling/v1/namespaces/default/horizontalpodautoscalers/spark232-autoscale
kubectl get HorizontalPodAutoscaler
```

### 3. demo
> using python: https://github.com/alswell/k8s/k8s_client
```bash
# get deployment under default namespace
curl -k --cert ./kubecfg.crt --key ./kubecfg.key https://localhost:6443/apis/extensions/v1beta1/namespaces/default/deployments
# get detail info
curl -k --cert ./kubecfg.crt --key ./kubecfg.key https://localhost:6443/apis/extensions/v1beta1/namespaces/default/deployments/spark-slave-deployment
# create deployment under default namespace, equals to: kubectl apply -f spark-slave-deployment.json
curl -k --cert ./kubecfg.crt --key ./kubecfg.key -X POST -d@spark-slave-deployment.json -H 'Content-Type: application/json'  https://localhost:6443/apis/extensions/v1beta1/namespaces/default/deployments
# edit: e.g. enlarge 'replicas' in spark-slave-deployment.json
curl -k --cert ./kubecfg.crt --key ./kubecfg.key -X PUT -d@spark-slave-deployment.json -H 'Content-Type: application/json'  https://localhost:6443/apis/extensions/v1beta1/namespaces/default/deployments/spark-slave-deployment
# delete
curl -k --cert ./kubecfg.crt --key ./kubecfg.key -X DELETE https://localhost:6443/apis/extensions/v1beta1/namespaces/default/deployments/spark-slave-deployment

```
