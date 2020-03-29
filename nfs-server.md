# How to install NFS Server on CentOs 7

## NFS Server

Instalamos o pacote do NFS:

```bash
yum install nfs-utils
```

Iniciamos e habilitamos o iniciio com o SO:

```bash
systemctl start nfs-server
systemctl start rpcbind
systemctl start nfs-lock
systemctl start nfs-idmap
systemctl enable nfs-server
systemctl enable rpcbind
systemctl enable nfs-lock
systemctl enable nfs-idmap
```

Criamos a pasta que será usada para o compartilhamento:

```bash
mkdir /pasta/compartilhada
chown nfsnobody:nfsnobody /pasta/compartilhada
chmod 755 /pasta/compartilhada
```

Declaramos a pasta que será exposta:

```bash
vim /etc/exports
```

Dentro do /etc/exports declaramos:

```bash
/pasta/compartilhada    *(rw,sync,no_subtree_check)
```

Após adicionar o compartilhamento no /etc/exports

```bash
exportfs -a
```

Liberamos o serviço no firewalld:

```bash
firewall-cmd --permanent --zone=public --add-service=nfs
firewall-cmd --permanent --zone=public --add-service=mountd
firewall-cmd --permanent --zone=public --add-service=rpc-bind
firewall-cmd --reload
```

## Client NFS

Instalamos o pacote do NFS:

```bash
yum install nfs-utils
```

Montamos a partição:

* Para o uso imediato (Perde a montagem após reiniciar a máquina):
  
```bash
mount -t nfs <IP_NFS_SERVER>:/pasta/compartilhada /mnt/
```

* Para iniciar junto com SO (Fica ativo mesmo reiniciando a máquina):

```bash
vim  /etc/fstab
```

```bash
<IP_NFS_SERVER>:/pasta/compartilhada  /mnt/  nfs      rw,sync,hard,intr  0     0
```

### Observações

* O "*"  faz com que aceite como cliente qualquer IP

## References

1. <https://www.howtoforge.com/tutorial/setting-up-an-nfs-server-and-client-on-centos-7/>
