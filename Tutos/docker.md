# Instalando o docker ce

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
  containerd.io \
  docker-ce \
  docker-ce-cli
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
