# single-node-k8s
单节点 Kubernetes 环境

```
vagrant init ubuntu/xenial64
# 编辑 Vagrantfile，安装 docker kubeadm kubelet kubectl，执行 kubeadm init
vagrant up
```