# How to use Jenkins com Lets Encrypt on Kubernetes


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

## Escrevendo os YAML

### Neste deploy usarei o Jenkins como um serviço sendo exposto pelo HAProxy-Ingress e o nip.io como DNS

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  labels:
    run: jenkins
  name: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      volumes:
      - name: jenkins-persistent-volume
        persistentVolumeClaim:
          claimName: jenkins-pvc # same name of pvc that was created
      containers:
      - image: jenkins/jenkins:lts
        ports:
          - name: http-port
            containerPort: 8080
        name: jenkins
        volumeMounts:
        - name: jenkins-persistent-volume # name of volume should match claimName volume
          mountPath: /var/jenkins_home # mount inside of contianer

---

apiVersion: v1
kind: Service
metadata:
  name: jenkins
spec:
  type: ClusterIP
  selector:
    app: jenkins
  ports:
    - name: jenkins
      port: 8080
      targetPort: 8080

---

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: jenkins
  annotations:
    certmanager.k8s.io/acme-challenge-type: http01
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - jenkins.<ip_servidor>.nip.io
    secretName: jenkins-tls
  rules:
  - host: jenkins.<ip_servidor>.nip.io
    http:
      paths:
      - path: /
        backend:
          serviceName: jenkins
          servicePort: 8080
```

* Fazemos o deploy do mesmo:

```bash
kubectl apply -f dsi_jenkins.yaml
```

## Aplicação em uso

### Jenkins subiu com certificado

![Jenkinns UP](https://github.com/galenothiago/tutoriais/blob/master/images/nfs-manager.jpeg?raw=true)

## References

1. <https://docs.docker.com/ee/ucp/kubernetes/storage/use-nfs-volumes/>
