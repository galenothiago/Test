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
mkdir /var/nfsshare
chown nfsnobody:nfsnobody /var/nfsshare
chmod 755 /var/nfsshare
```

Declaramos a pasta que será exposta:

```bash
vim /etc/exports
```

Dentro do /etc/exports declaramos:

```bash
/var/nfs        *(rw,sync,no_subtree_check)
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

```bash
mount -t nfs <IP_NFS_SERVER>:/var/nfsshare /mnt/
```

### Observações

* O "*"  faz com que aceite como cliente qualquer IP

## References

1. <https://www.howtoforge.com/tutorial/setting-up-an-nfs-server-and-client-on-centos-7/>
