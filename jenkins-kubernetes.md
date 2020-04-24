# How to use Jenkins com Lets Encrypt on Kubernetes

## Instalando o NFS Server para uso pelo kubernetes

> [Configurando o let's encrypt no kubernetes](https://github.com/galenothiago/tutoriais/blob/master/nfs-server.md)

## Entendimento Teórico

## O que é Jenkins

É um servidor de Integração Contínua open-source feito em Java, criado por Kohsuke Kawaguchi.
Ele pode ser rodado de forma standalone ou como uma web aplicação dentro de um servidor web.
Neste sentido, ele consiste em um código de automação aberto, cujo a inscrição estava em Java,
o que auxilia na automatização da parte não humana.
Principalmente no que diz respeito aos aspectos de desenvolvimento de software,
facilitando os elementos técnicos de entrega contínua.

O mesmo destaca-se que este sistema é baseado em um servidor que tem sua execução em contêineres de servlet.
Assim suportando ferramentas de controle de versão,
assim como permite a possbilidade execução de projetos que são pautados em Apache Ant, maven e sbt.

### Podemos destacar algumas vantagens no uso do Jenkins:

* Builds periódicos;
* Mailler
* Credencias
* Agente SSH
* Testes automatizados;
* Possibilita analise de código;
* Identificar erros mais cedo;
* Fácil de operar e configurar;
* Builds em diversos ambientes;

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
  namespace: default
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
      containers:
        - name: jenkins
          image: jenkins/jenkins:lts
          env:
            - name: JAVA_OPTS
              value: -Djenkins.install.runSetupWizard=false
          ports:
            - name: http-port
              containerPort: 8080
          volumeMounts:
            - name: jenkins-home
              mountPath: /var/jenkins_home
      volumes:
        - name: jenkins-home
          emptyDir: {}

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
    - jenkins.35.239.225.194.nip.io
    secretName: jenkins-tls
  rules:
  - host: jenkins.35.239.225.194.nip.io
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

1. <https://medium.com/cwi-software/testes-automatizados-no-jenkins-recursos-plugins-e-dicas-para-aumentar-a-produtividade-1685ffa1e9db>
2. <https://phoenixnap.com/kb/how-to-install-jenkins-kubernetes>
3. <https://www.blazemeter.com/blog/how-to-setup-scalable-jenkins-on-top-of-a-kubernetes-cluster>
