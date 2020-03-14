# How to install Kubernetes on Centos7 with Kubeadm:

## Em todos os nós:

## Desabilitar o firewalld e colocar o selinux em modo permissive:

### Firewalld

```bash
systemctl stop firewalld
systemctl disable firewalld
```

### selinux

Desativando para a sessão atual:

```bash
setenforce 0
```

Desativando para as proximas reinicializações:

```bash
sed -i s/^SELINUX=.*$/SELINUX=permissive/ /etc/selinux/config
```

Verificar se está no modo permissive:

```bash
sestatus
```

## Instalando o docker ce:

[Instalação Docker](https://github.com/galenothiago/estudos-javascript/blob/master/fundamentos/organizacao.js)

### Habilitando roteamento de pacote:

```bash
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-iptables  = 1
EOF

sysctl --system
```

## Adicionando repositório do kubernetes:

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg \
https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

Com o repositório adicionado podemos instalar os itens a seguir:

```bash
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```

Ativamos o kubelet:

```bash
systemctl enable --now kubelet
```

### Liberar portar no firewall:

* Control-plane node(s)

| Protocol | Direction | Port Range |         Purpose         |        Used By       |
|:--------:|:---------:|:----------:|:-----------------------:|:--------------------:|
|    TCP   |  Inbound  |    6443*   |  Kubernetes API server  |          All         |
|    TCP   |  Inbound  |  2379-2380 |  etcd server client API | kube-apiserver, etcd |
|    TCP   |  Inbound  |    10250   |       Kubelet API       |  Self, Control plane |
|    TCP   |  Inbound  |    10251   |      kube-scheduler     |         Self         |
|    TCP   |  Inbound  |    10252   | kube-controller-manager |         Self         |

* Worker node(s)

| Protocol | Direction |  Port Range |      Purpose           |       Used By       |
|:--------:|:---------:|:-----------:|:----------------------:|:-------------------:|
|    TCP   |  Inbound  |    10250    |       Kubelet API      | Self, Control plane |
|    TCP   |  Inbound  | 30000-32767 |    NodePort Services   |         All         |


## Apenas no nó de controle: 

Dar o start como nó de controle:

```bash
kubeadm init
```

Instalar a rede Pod (usaremos a calico): 

```bash
kubectl apply -f https://docs.projectcalico.org/v3.11/manifests/calico.yaml
```
## Apenas nos nós workers:

Adicionar eles como workes:

```bash
kubeadm join <control-plane-host>:<control-plane-port> --token <token> \
    --discovery-token-ca-cert-hash sha256:<hash>
```
## Como remover um nó:

No nó de controle:

```bash
kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
kubectl delete node <node name>
```

No nó worker:

```bash
kubeadm reset
```

## References

1. https://kubernetes.io/docs/tasks/tools/install-kubectl/
1. https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
1. https://docs.docker.com/install/linux/docker-ce/centos/
1. https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
