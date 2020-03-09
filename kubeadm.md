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

### Instalando o docker ce:

Para que o kubernetes funcione corretamente precisamos instalar um virtualizador ou o docker:

Para instalar o docker podemos usar dois métodos:

* Método 1:

Instalando os pacotes necessários, repositório e instalando via repositório:

```bash
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```
Adicionamos o repositório oficial:

```bash
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

E instalamos o docker:

```bash
yum update -y && yum install -y \
  containerd.io-1.2.10 \
  docker-ce-19.03.4 \
  docker-ce-cli-19.03.4
```

Criar /etc/docker directory.

```bash
mkdir /etc/docker
```

Configurando daemon:

```bash
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF
```

```bash
mkdir -p /etc/systemd/system/docker.service.d
```

Restart Docker

```bash
systemctl daemon-reload
systemctl restart docker
```

* Método 2: 

Utilizando o script criado por eles:

```bash
curl https://get.docker.com/ -o script_docker.sh
```

Permissão de execução no script:

```bash
chmod +x script_docker.sh
```

E instalar usando o script:

```bash
./script_docker.sh
```

Por fim iniciamos o serviço e ativamos ele no inicio do SO:

```bash
systemctl start docker
systemctl enable docker
```

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
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
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

## References

1. https://kubernetes.io/docs/tasks/tools/install-kubectl/
1. https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
1. https://docs.docker.com/install/linux/docker-ce/centos/
1. https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
