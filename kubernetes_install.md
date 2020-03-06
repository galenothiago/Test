# How to install Kubernetes on Centos7 with Kubeadm:

## Primeiro passo é desabilitar o firewall e colocar o selinux em modo permissive:

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

## Instalando o Kubectl e Minikube:

### Kubectl

Adicionamos o repositorio oficial do kubernetes e depois instalamos o kubectl:

```bash
cat <<'EOF' > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

```bash
yum -y install kubectl
```

### Minikube

Baixamos do storage do google damos permissão de execução:

```bash
wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 -O minikube
```

Colocamos nos binarios da maquina para ser reconhecido como um comando:

```bash
chmod 755 minikube
mv minikube /usr/bin/
```


## Instalando o docker ce:

Para que o minikube funcione corretamente precisamos instalar um virtualizador (kvm, virtualbox) ou o docker (que é o que usaremos):

Pacotes necessários:

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
E por fim instalamos e iniciamos o serviço:

```bash
yum install docker-ce docker-ce-cli containerd.io
systemctl start docker
systemctl enable docker
```
## Após todas essas configurações efetuadas startamos o minikube:

```bash
minikube start --vm-driver none
``` 

## References

1. https://kubernetes.io/docs/tasks/tools/install-kubectl/
1. https://docs.docker.com/install/linux/docker-ce/centos/
1. https://www.server-world.info/en/note?os=CentOS_7&p=kubernetes&f=1