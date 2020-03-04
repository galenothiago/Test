# Concepts of Kubernetes:

## O que é?:

```bash
O Kubernetes (K8s) é um sistema de código aberto para automatizar a implatação, 
dimensionamento e gerenciamento de aplicativos em contêiner.
```
## diagrama de um cluster Kubernetes com todos os componentes ligados:

![](https://d33wubrfki0l68.cloudfront.net/7016517375d10c702489167e704dcb99e570df85/7bb53/images/docs/components-of-kubernetes.png)


#### Possibilidades que o kubernetes traz:

* Orquestrar containers em vários hosts.
* Aproveitar melhor o hardware para maximizar os recursos necessários na execução das aplicações corporativas.
* Controlar e automatizar as implantações e atualizações de aplicações.
* Montar e adicionar armazenamento para executar aplicações com monitoração de estado.
* Escalar rapidamente as aplicações em containers e recursos relacionados.
* Gerenciar serviços de forma declarativa, garantindo que as aplicações sejam executadas sempre da mesma maneira como foram implantadas.
* Verificar a integridade e autorrecuperação das aplicações com posicionamento, reinício, replicação e escalonamento automáticos.

## Partes integrantes: 

![](https://www.redhat.com/cms/managed-files/kubernetes-diagram-2-824x437.png)


#### Master:

```bash
a máquina que controla os nós do Kubernetes. É nela que todas as atribuições de tarefas se originam.
```

#### Node:

```bash
são máquinas que realizam as tarefas solicitadas e atribuídas. A máquina mestre do Kubernetes controla os nós.
```

#### Pod: 

```bash
um grupo de um ou mais containers implantados em um único nó. 
Todos os containers em um pod compartilham o mesmo endereço IP, IPC, nome do host e outros recursos.
Os pods separam a rede e o armazenamento do container subjacente. 
Isso facilita a movimentação dos containers pelo cluster.
```

#### ReplicaSets: 

```bash
controla quantas cópias idênticas de um pod devem ser executadas em um determinado local do cluster.
```

#### Service:

```bash
desacopla as definições de trabalho dos pods. 
Os proxies de serviço do Kubernetes automaticamente levam as solicitações de serviço para o pod correto, 
independentemente do local do pod no cluster ou se foi substituído.
```

#### Kubelet: 

```bash
um serviço executado nos nós que lê os manifestos do container e garante que os containers
definidos foram iniciados e estão em execução.
```

#### kubectl: 

```bash
a ferramenta de configuração da linha de comando do Kubernetes.
```

#### Deployments:

```bash
Um Deployment fornece atualizações declarativas para Pods e ReplicaSets.
Você descreve um estado desejado em umDeployment e o Deployment Controller altera o estado real para o estado
desejado a uma taxa controlada. Você pode definir implantações para criar novos ReplicaSets
ou remover implantações existentes e adotar todos os seus recursos com novas implantações.
```
#### Namespaces:

```bash
O Kubernetes suporta vários clusters virtuais suportados pelo mesmo cluster físico. 
Esses clusters virtuais são chamados de namespaces.
```

#### DaemonSet:

```
Um DaemonSet garante que todos (ou alguns) nós executem uma cópia de um Pod. 
À medida que os nós são adicionados ao cluster, os Pods são adicionados a eles. 
À medida que os nós são removidos do cluster, esses Pods são coletados como lixo. 
A exclusão de um DaemonSet limpará os Pods que ele criou.
```
#### StatefulSets:

```
StatefulSet é o objeto da API de carga de trabalho usado para gerenciar aplicativos com estado.
Gerencia a Deployment e o dimensionamento de um conjunto de Pods, 
e fornece garantias sobre o pedido e a exclusividade desses Pods. Como uma Deployment, 
um StatefulSet gerencia os Pods baseados em uma especificação de contêiner idêntica. 
Ao contrário de uma Deployment, um StatefulSet mantém uma identidade persistente para cada um de seus Pods. 
Esses pods são criados com a mesma especificação, 
mas não são intercambiáveis: cada um possui um identificador persistente que mantém em qualquer reagendamento.
```

#### Jobs - Run to Completion:

```bash
Um job cria um ou mais Pods e garante que um número especificado deles seja encerrado com êxito.
Conforme os pods são concluídos com êxito, o Job rastreia as conclusões bem-sucedidas.
Quando um número especificado de conclusões bem-sucedidas é atingido, a tarefa (ou seja, Trabalho) é concluída.
A exclusão de um trabalho limpará os Pods criados.
Um caso simples é criar um objeto de trabalho para executar um pod de maneira confiável até a conclusão.
O objeto Job iniciará um novo Pod se o primeiro Pod falhar ou for excluído
(por exemplo, devido a uma falha de hardware do nó ou uma reinicialização do nó).

```

#### kube-proxy:

```bash
Para gerenciar sub-redes de hosts individuais e tornar os serviços disponíveis para outros componentes, 
um pequeno serviço de proxy chamado kube-proxy é executado em cada servidor de node. 
Este processo encaminha requisições aos containers corretos, 
e é geralmente responsável por certificar-se de que o ambiente de rede é previsível e acessível, 
mas isolado quando apropriado.
```

#### Volumes:

```bash
Os arquivos em disco em um Contêiner são efêmeros, 
o que apresenta alguns problemas para aplicativos não triviais ao executar em Contêineres. Primeiro, quando um Container falha, 
o kubelet o reinicia, mas os arquivos são perdidos - o Container começa com um estado limpo. 
Segundo, ao executar contêineres juntos Pod, geralmente é necessário compartilhar arquivos entre esses contêineres. 
A Volumeabstração do Kubernetes resolve esses dois problemas.
```

#### etcd:

```bash
Armazenamento de valores-chave consistente e de alta disponibilidade usado como armazenamento de apoio
do Kubernetes para todos os dados do cluster. Se o cluster do Kubernetes usa o 
etcd como repositório de backup, verifique se você tem um plano de backup para esses dados.
Você pode encontrar informações detalhadas sobre o etcd na documentação oficial .
```

## References

1. https://kubernetes.io/docs/tasks/tools/install-kubectl/
1. https://www.redhat.com/pt-br/topics/containers/what-is-kubernetes
1. https://www.server-world.info/en/note?os=CentOS_7&p=kubernetes&f=1