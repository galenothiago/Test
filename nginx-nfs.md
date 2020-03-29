# How to use Nginx with NFS server on Kubernetes

## Instalando o NFS Server para uso pelo kubernetes

> [Instalação NFS Server](https://github.com/galenothiago/tutoriais/blob/master/nfs-server.md)

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
um volume sobrevive a todos os contêineres executados no Pod e
os dados são preservados nas reinicializações do contêiner. Obviamente, quando um Pod deixar de existir,
o volume deixará de existir também. Talvez mais importante que isso,
o Kubernetes suporta muitos tipos de volumes, e um Pod pode usar qualquer número deles simultaneamente.
Na sua essência, um volume é apenas um diretório, possivelmente com alguns dados,
acessíveis aos Containers em um Pod. Como esse diretório é criado,
a mídia que o suporta e o conteúdo é determinado pelo tipo de volume específico usado.
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
(eles podem ser montados uma vez que sejam de leitura / gravação ou muitas vezes somente leitura).
```

### Storage Class

```bash
Um StorageClass fornece uma maneira para os administradores descreverem as “classes” de armazenamento.
Classes diferentes podem ser mapeadas para níveis de qualidade de serviço,
para políticas de backup ou para políticas arbitrárias determinadas pelos administradores de cluster.
O próprio Kubernetes não tem opinião sobre o que as classes representam.
Às vezes, esse conceito é chamado de "perfis" em outros sistemas de armazenamento
```

### Criando o persistence volume storage class NFS

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
  labels:
    name: mynfs # qualquer nome
spec:
  storageClassName: manual # mesmo storage class que persistence volume claim
  capacity:
    storage: 200Mi
  accessModes:
    - ReadWriteMany
  nfs:
    server: <IP_NFS_SERVER> # ip do NFS server
    path: "/pasta/compartilhada" # pasta compartilhada
```

* Fazemos o deploy do mesmo:

```bash
kubectl apply -f nfs.yaml
```

### Criando o persistence volume claim

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  storageClassName: manual # mesmo storage class que persistence volume
  accessModes:
    - ReadWriteMany #  mesmo tipo que no  PersistentVolume
  resources:
    requests:
      storage: 50Mi
```

* Fazemos o deploy do mesmo:

```bash
kubectl apply -f pvc-nfs.yaml
```

### Com o PV e PVC criados podemos fazer o deploy do nginx para usa-los

#### Neste deploy usarei o nginx como um serviço sendo exposto pelo HAProxy-Ingress e o nip.io como DNS

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nfs-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
      - name: nfs-test
        persistentVolumeClaim:
          claimName: nfs-pvc # mesmo nome do pvc que foi criado
      containers:
      - image: nginx
        resources:
          requests:
            memory: 50Mi
            cpu: 50m
          limits:
            memory: 100Mi
            cpu: 100m
        name: nginx
        volumeMounts:
        - name: nfs-test # o nome do volume deve corresponder ao volume ClaimName
          mountPath: /usr/share/nginx/html # aonde o volume será montado

---

apiVersion: v1
kind: Service
metadata:
  labels:
    run: nginx
  name: nfs-nginx
  namespace: default
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - name: http
      port: 80
      targetPort: 80

---

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nfs-nginx
spec:
  rules:
  - host: nfs-nginx.IP.nip.io
    http:
      paths:
      - path: /
        backend:
          serviceName: nfs-nginx
          servicePort: 80
```

* Fazemos o deploy do mesmo:

```bash
kubectl apply -f nginx-nfs.yaml
```

#### Aqui podemos ver que o NFS server foi montado (na mão) no manager

![Montando o NFS SErver no manager](https://github.com/galenothiago/tutoriais/blob/master/images/nfs-manager.jpeg?raw=true)

#### Aqui podemos ver que o NFS Server foi montado no worker como um volume persistente

Mesmo sem termos montado nada worker

![Volume persistente no worker](https://github.com/galenothiago/tutoriais/blob/master/images/pv-worker.jpeg?raw=true)

#### Por fim o resultado

O nignx subiu pegando o arquivo index.html criado diretamente no NFS server

![Nginx UP](https://github.com/galenothiago/tutoriais/blob/master/images/nginx.jpg?raw=true)

## References

1. <https://docs.docker.com/ee/ucp/kubernetes/storage/use-nfs-volumes/>
1. <https://www.thegeekdiary.com/endpoint-is-not-created-for-service-in-kubernetes/>
1. <https://medium.com/@myte/kubernetes-nfs-and-dynamic-nfs-provisioning-97e2afb8b4a9>
1. <https://kubernetes.io/docs/concepts/storage/volumes/>
1. <https://kubernetes.io/docs/concepts/storage/persistent-volumes/>
1. <https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/>
1. <https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/>
