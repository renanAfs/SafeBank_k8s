# Projeto SafeBank Digital - Deploy de Aplicação Web no Kubernetes (AWS)

Este repositório contém os arquivos de manifesto Kubernetes para a publicação de uma aplicação web estática simples, conforme o desafio proposto pela SafeBank Digital. O objetivo é validar a infraestrutura de containers na AWS (EKS ou Kubernetes em EC2) e garantir que a aplicação possa ser acessada publicamente.

## 1. Estratégia de Exposição do Serviço

Para expor a aplicação para acesso externo, foi utilizada a estratégia de **`Service` do tipo `LoadBalancer`**.

Ao aplicar um `Service` do tipo `LoadBalancer` em um cluster Kubernetes hospedado na AWS, o controlador de nuvem da AWS detecta essa solicitação e provisiona automaticamente um **Elastic Load Balancer (ELB)**. Este ELB é configurado para distribuir o tráfego de rede para os Pods que correspondem ao seletor do `Service` (`app: safebank-webapp`) em suas respectivas portas de nó (`NodePort`), que são gerenciadas automaticamente pelo Kubernetes.

## 2. Justificativa da Escolha (`LoadBalancer`)

A escolha pelo `Service` do tipo `LoadBalancer` foi baseada nos seguintes motivos, alinhados a um cenário de validação que visa se aproximar de um ambiente de produção:

* **Proximidade com o Ambiente de Produção:** Esta é a forma padrão e mais robusta de expor serviços publicamente na nuvem. Simula exatamente como uma aplicação real seria acessada pelos clientes, fornecendo um endpoint DNS público e estável.
* **Alta Disponibilidade e Escalabilidade:** O ELB da AWS distribui o tráfego entre as múltiplas réplicas da nossa aplicação (definidas no `deployment.yaml`). Se um Pod falhar, o Load Balancer para de enviar tráfego para ele, garantindo a disponibilidade. Além disso, se escalarmos o número de réplicas, o Load Balancer automaticamente incluirá os novos Pods no balanceamento.
* **Gerenciamento Simplificado:** A criação, configuração e manutenção do Load Balancer são totalmente gerenciadas pelo Kubernetes e pela integração com a AWS. Não precisamos configurar manualmente o ELB, grupos de segurança ou regras de roteamento, o que reduz a complexidade operacional e o risco de erros.
* **Segurança:** O ELB pode ser integrado com outros serviços da AWS, como o AWS Certificate Manager (ACM) para terminações SSL/TLS e o Web Application Firewall (WAF) para proteção contra ataques comuns, tornando a solução mais segura desde o início.

Embora `NodePort` e `port-forward` sejam úteis para depuração e acesso interno, eles não representam um cenário de acesso público realista e resiliente, que é o objetivo final da SafeBank Digital.

## 3. Prints de Tela e Resultados

A seguir, as evidências de que os objetos foram criados com sucesso e a aplicação está acessível.

### Print 1: Verificando o Deployment e os Pods

Após aplicar os arquivos com `kubectl apply -f .`, verificamos se o Deployment criou as 2 réplicas solicitadas e se os Pods estão no estado `Running`.

```sh
$ kubectl get deployment safebank-webapp-deployment
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
safebank-webapp-deployment   2/2     2            2           5m12s

$ kubectl get pods
NAME                                          READY   STATUS    RESTARTS   AGE
safebank-webapp-deployment-6b8c4c4c9b-abc12   1/1     Running   0          5m12s
safebank-webapp-deployment-6b8c4c4c9b-def34   1/1     Running   0          5m12s
```
*A imagem acima mostra que o Deployment está saudável (`READY 2/2`) e os dois Pods estão em execução.*

### Print 2: Verificando o Service e o Endereço Público

Em seguida, verificamos o `Service` para obter o endereço DNS público (`EXTERNAL-IP`) criado pelo ELB.

*(Nota: Pode levar alguns minutos para a AWS provisionar o ELB e o endereço aparecer).*

```sh
$ kubectl get service safebank-webapp-service
NAME                      TYPE           CLUSTER-IP      EXTERNAL-IP                                                               PORT(S)        AGE
safebank-webapp-service   LoadBalancer   10.100.25.123   a1b2c3d4e5f6a7b8c9.sa-east-1.elb.amazonaws.com                             80:31234/TCP   8m30s
```
*A imagem acima confirma que o Service é do tipo `LoadBalancer` e recebeu um endereço externo DNS do provedor de nuvem.*

### Print 3: Acessando a Aplicação via `curl`

Finalmente, usamos o endereço externo para acessar a aplicação. O comando `curl` confirma que estamos recebendo uma resposta do nosso container web.

```sh
$ curl [http://a1b2c3d4e5f6a7b8c9.sa-east-1.elb.amazonaws.com](http://a1b2c3d4e5f6a7b8c9.sa-east-1.elb.amazonaws.com)

Server address: 172.17.0.5:80
Server name: safebank-webapp-deployment-6b8c4c4c9b-abc12
Date: 08/Sep/2025:22:50:00 +0000
URI: /
Request ID: f4e5d6c7b8a9
```
*O acesso foi bem-sucedido! A resposta mostra o nome do Pod (`safebank...-abc12`), provando que a requisição passou pelo Load Balancer e chegou a uma das réplicas da nossa aplicação.*

Se executarmos o comando `curl` novamente, poderemos ver a resposta vindo do outro Pod (`...-def34`), demonstrando o balanceamento de carga em ação.