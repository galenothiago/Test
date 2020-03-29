# How to use Nginx with NFS server on Kubernetes

## Instalando o NFS Server para uso pelo kubernetes

[Instalação NFS Server](https://github.com/galenothiago/tutoriais/blob/master/nfs-server.md)

Após a instalação do server, em outra maquina, ou em uma das maquinas do cluster (não recomendado)
é bom testar se o manager consegue pingar o NFS Server e acessar sua pasta compartilhada.

```bash
ping <IP_NFS_SERVER>
```

Com esse comando conseguimos saber a partição compartilhada

```bash
showmount -e <IP_NFS_SERVER>
```

Saida será algo parecido com:

```bash
Export list for <IP_NFS_SERVER>:
/pasta/compartilhada *
```

## Entendimento Teórico

Para que a solução funcione precisamos usar os conceitos de Volume, Volume persistente e Volume claim

### Volume

```bash
Volume Kubernetes, tem uma vida útil explícita - o mesmo que o Pod que o inclui. Conseqüentemente,
um volume sobrevive a todos os contêineres executados no Pod e os dados são preservados nas reinicializações do contêiner.
Obviamente, quando um Pod deixar de existir, o volume deixará de existir também. Talvez mais importante que isso,
o Kubernetes suporta muitos tipos de volumes, e um Pod pode usar qualquer número deles simultaneamente
Na sua essência, um volume é apenas um diretório, possivelmente com alguns dados, acessíveis aos Containers em um Pod.
Como esse diretório é criado, a mídia que o suporta e o conteúdo é determinado pelo tipo de volume específico usado.
```

### Volume Persistente

```bash
Um PersistentVolume (PV) é uma parte de armazenamento no cluster que foi provisionada por um administrador
ou provisionada dinamicamente usando StorageClass. É um recurso no cluster,
assim como um nó é um recurso de cluster. PVs são plug-ins de volume como Volumes,
mas têm um ciclo de vida independente de qualquer Pod individual que usa o PV.
Esse objeto da API captura os detalhes da implementação do armazenamento,
seja NFS, iSCSI ou um sistema de armazenamento específico do provedor de nuvem.
```

### Volume Claim

```bash
Um persistentVolumeClaim volume é usado para montar um PersistentVolume em um Pod.
Os PersistentVolumes são uma maneira dos usuários "reivindicarem" o armazenamento durável
(como um GCE PersistentDisk ou um volume iSCSI) sem conhecer os detalhes do ambiente em nuvem específico.

Um PersistentVolumeClaim (PVC) é uma solicitação de armazenamento por um usuário.
É semelhante a um Pod. Os pods consomem recursos de nó e os PVCs consomem recursos de PV.
Os pods podem solicitar níveis específicos de recursos (CPU e memória).
As reivindicações podem solicitar tamanhos específicos e modos de acesso
(por exemplo, eles podem ser montados uma vez que sejam de leitura / gravação ou muitas vezes somente leitura).
```

 df -kh



## References

1. <https://docs.docker.com/ee/ucp/kubernetes/storage/use-nfs-volumes/>
1. <https://www.thegeekdiary.com/endpoint-is-not-created-for-service-in-kubernetes/>
1. <https://medium.com/@myte/kubernetes-nfs-and-dynamic-nfs-provisioning-97e2afb8b4a9>
1. <https://kubernetes.io/docs/concepts/storage/volumes/>
1. <https://kubernetes.io/docs/concepts/storage/persistent-volumes/>
1. <https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/>
1. <https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/>
