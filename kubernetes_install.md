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

## References

1. https://kubernetes.io/docs/tasks/tools/install-kubectl/
1. https://docs.docker.com/install/linux/docker-ce/centos/
1. https://www.server-world.info/en/note?os=CentOS_7&p=kubernetes&f=1