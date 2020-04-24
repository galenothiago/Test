# How to use Jenkins com Lets Encrypt on Kubernetes

## Introdução

A Let’s Encrypt é uma autoridade certificadora (AC) gratuita, automatizada e aberta que opera em prol do benefício público. É um serviço provido pela Internet Security Research Group (ISRG).

Os princípios chave da Let’s Encrypt são:

* Gratuita
* Automatizada
* Segura
* Transparente:
* Aberta
* Cooperativa

### Escrevendo os YAML

#### Primeiro criamos a entidade certificado

Criamos um namespace para a mesma para isolar o recurso:

```bash
kubectl create namespace cert-manager
```

Istalamos o cert-manager é um serviço do Kubernetes que fornece certificados TLS do Let’s Encrypt

```bash
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.14.2/cert-manager.yaml
```

O emissor de produção

```bash
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
  namespace: cert-manager
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: your_email_address_here
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod
    # Enable the HTTP-01 challenge provider
    solvers:
    - http01:
        ingress:
          class: haproxy
```

E quando vamos utilizar o mesmo adicionamos os campos "annotations" e "spec: tls" ao ingress usado:
Exemplo do ingress de um jenkins

```bas
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

## Aplicação em uso

### Jenkins subiu com certificado

![Jenkinns UP](https://raw.githubusercontent.com/galenothiago/tutoriais/master/images/jenkins.jpeg)

## References

1. <https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nginx-ingress-with-cert-manager-on-digitalocean-kubernetes-pt>
2. <https://cert-manager.io/docs/configuration/acme/>
