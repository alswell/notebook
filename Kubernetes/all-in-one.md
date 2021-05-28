# Kubernetes部署与使用
## 一、部署

### 1. 安装部署工具 kubeadm
[master和minion相同操作]
#### 1.1 添加yum repo
```bash
vim /etc/yum.repos.d/kubernetes.repo
```
```text
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
```
#### 1.2 执行安装
```bash
yum install -y kubeadm
```
#### 1.3 设置 kubelet 服务开机自动启动
```bash
systemctl enable kubelet
```

### 2. 安装 docker
[master和minion相同操作]
#### 2.1 执行安装
```bash
yum install -y docker
```
#### 2.2 注册本地docker镜像服务
- 已在69服务器上搭建好docker本地镜像服务器 172.12.78.69:5000
- 参考docker本地镜像服务器搭建 https://www.cnblogs.com/Javame/p/7389093.html
```bash
echo '{ "insecure-registries":["172.12.78.69:5000"] }' > /etc/docker/daemon.json
```
#### 2.3 启动docker并开机自动启动
```bash
systemctl restart docker
systemctl enable docker
```
#### 2.4 准备k8s组件镜像
> 国内无法访问谷歌服务器，从国内 registry 服务器拉取必要镜像
```bash
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

### 3. 安装前设置
[master和minion相同操作]
#### 关闭防火墙和selinux
```bash
# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld
# 关闭selinux
setenforce 0 
sed -i '/^SELINUX=/c SELINUX=disabled' /etc/selinux/config
```
#### 在/etc/hosts添加
```text
<master-ip> master
<minion-1-ip> minion-1
<minion-2-ip> minion-2
```

### 4. kubeadm init
[仅master节点]
#### 4.1 执行命令（会自动检查宿主机环境，根据提示按需执行第6节中Tips的操作）
- --pod-network-cidr选项必须为10.244.0.0/16，否则后面安装flannel会失败
```bash
hostnamectl set-hostname master
kubeadm init --apiserver-advertise-address=<master_ip> --kubernetes-version=v1.13.0 --pod-network-cidr=10.244.0.0/16
```
#### 4.2 若安装成功会输出提示，根据提示完成剩余操作，比如：
```bash
cp /etc/kubernetes/admin.conf $HOME/.kube/config
```
#### 4.3 也会提示如何添加minion节点到本集群
```bash
kubeadm join <master_ip>:6443 --token <token> --discovery-token-ca-cert-hash <sha256>
```
#### 4.4 安装flannel
```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
sed -i 's/quay.io\/coreos/172.12.78.69:5000/' kube-flannel.yml  # 修改yml文件，使用本地镜像，原版镜像可能拉不到
kubectl apply -f kube-flannel.yml
```

### 5. kubeadm join
[仅minion节点]
#### 5.1 执行命令（会自动检查宿主机环境，根据提示按需执行第6节中Tips的操作）
```bash
hostnamectl set-hostname minion-01
kubeadm join <master_ip>:6443 --token <token> --discovery-token-ca-cert-hash <sha256>  # 同4.3节
```

### 6. Tips
[master和minion相同操作]
- 在kubeadm init和kubeadm join中，会自动检查宿主机环境，根据提示按需执行以下操作
#### 6.1 close swap
```bash
swapoff -a
sed -i '/swap/s/^/# /' /etc/fstab
free -m  # 查看是否生效
```
#### 6.2 enable iptable bridge:
```bash
echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
echo net.bridge.bridge-nf-call-iptables = 1 >> /etc/sysctl.conf
```

## 二、dashboard
### 1. 安装
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```
### 2. 访问
参考 https://www.cnblogs.com/RainingNight/p/deploying-k8s-dashboard-ui.html
#### 2.1 准备key，然后把p12证书文件拷到本机，并在浏览器中导入证书
```bash
grep 'client-certificate-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d > kubecfg.crt
grep 'client-key-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d > kubecfg.key
openssl pkcs12 -export -clcerts -inkey kubecfg.key -in kubecfg.crt -out kubecfg.p12 -name "kubernetes-client"
```
#### 2.2 获取管理员token（登录dashboard时需要）
##### 2.2.1 创建服务账号
```bash
vim admin-user.yaml
```
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
```
```bash
kubectl create -f admin-user.yaml
```
##### 2.2.2 绑定角色
```bash
vim admin-user-role-binding.yaml
```
```yaml
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
```
```bash
kubectl create -f admin-user-role-binding.yaml
```
##### 2.2.3 获取token
```bash
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```
## 三、使用
### 3.1 kubectl
#### 3.1.1 基本命令参考: k8s/doc/cmd/kubectl.md
#### 3.1.2 手动扩容/缩容 (环境172.12.78.31:/root/k8s)
- 只需修改deployment yaml文件中的replicas字段（本例修改文件spark-slave-deployment.yaml），然后
```bash
kubectl apply -f spark-slave-deployment.yaml
```
- 或者使用命令：(参考 https://www.kubernetes.org.cn/deployment)
```bash
kubectl scale deployment spark-slave-deployment --replicas 10
```
### 3.2 RestAPI
- https://blog.csdn.net/jinzhencs/article/details/51452208 (入门版)
- https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.12/ (官方详细版)
- kubectl和RestAPI的使用都是对应的，请看下面的例子：spark232-slave-autoscale.json
#### 3.2.1 自动扩容 (环境172.12.78.31:/root/k8s)
- 使用命令：(参考 https://www.kubernetes.org.cn/deployment)
```bash
kubectl autoscale deployment spark232-slave-deployment --min=1 --max=5 --cpu-percent=80
kubectl get HorizontalPodAutoscaler  # 查看效果
```
- 修改yaml/json文件配置，然后用kubectl apply使之生效
```bash
kubectl apply -f spark232-slave-autoscale.json
kubectl get HorizontalPodAutoscaler

```
- 修改yaml/json文件配置，然后用相应的RestAPI使之生效
```bash
curl -k --cert ./kubecfg.crt --key ./kubecfg.key -X PUT -d@spark232-slave-autoscale.json -H 'Content-Type: application/json' https://localhost:6443/apis/autoscaling/v1/namespaces/default/horizontalpodautoscalers/spark232-autoscale
kubectl get HorizontalPodAutoscaler
```
python请求RestAPI的代码请参考k8s/k8s_client
#### 3.2.2 deployment(手动扩容的RestAPI方式)
```bash
# 获取default namespace下所有deployment
curl -k --cert ./kubecfg.crt --key ./kubecfg.key https://localhost:6443/apis/extensions/v1beta1/namespaces/default/deployments
# 获取详情
curl -k --cert ./kubecfg.crt --key ./kubecfg.key https://localhost:6443/apis/extensions/v1beta1/namespaces/default/deployments/spark-slave-deployment
# 在default namespace下创建deployment 效果等于：kubectl apply -f spark-slave-deployment.json
curl -k --cert ./kubecfg.crt --key ./kubecfg.key -X POST -d@spark-slave-deployment.json -H 'Content-Type: application/json'  https://localhost:6443/apis/extensions/v1beta1/namespaces/default/deployments
# 修改：比如把spark-slave-deployment.json中的replicas调大（手动扩容）然后执行下一行命令
curl -k --cert ./kubecfg.crt --key ./kubecfg.key -X PUT -d@spark-slave-deployment.json -H 'Content-Type: application/json'  https://localhost:6443/apis/extensions/v1beta1/namespaces/default/deployments/spark-slave-deployment
# 删除
curl -k --cert ./kubecfg.crt --key ./kubecfg.key -X DELETE https://localhost:6443/apis/extensions/v1beta1/namespaces/default/deployments/spark-slave-deployment

```
