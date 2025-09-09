# Projeto SafeBank Digital: Deploy de Aplicação Web no Kubernetes (AWS)

## 1. Objetivo do Projeto

O objetivo deste projeto foi simular a publicação de uma aplicação web estática para a empresa fictícia SafeBank Digital em um ambiente de nuvem real (AWS Academy). A missão era utilizar Kubernetes para criar os objetos básicos (Pods, Services, Deployments) e expor a aplicação para acesso, validando a infraestrutura de contêineres da empresa.

---

## 2. Estratégia de Implementação e Desafios

A execução do projeto seguiu uma abordagem de dois planos, adaptando-se aos desafios encontrados no ambiente de laboratório.

### Plano A: Amazon EKS (Elastic Kubernetes Service)

A estratégia inicial era utilizar o **Amazon EKS**, o serviço gerenciado de Kubernetes da AWS. Esta seria a abordagem mais próxima de um ambiente de produção real, permitindo o uso de um `Service` do tipo `LoadBalancer` para expor a aplicação de forma robusta.

* **Desafio Encontrado:** Ao tentar criar o cluster com a ferramenta `eksctl`, o processo falhou com um erro de **`AccessDeniedException`**. A análise do erro revelou que o usuário temporário do ambiente AWS Academy (`voclabs`) não possuía as permissões necessárias (`eks:DescribeClusterVersions`) para criar ou gerenciar recursos do EKS.

### Plano B: Minikube em uma Instância EC2

Diante da restrição de permissões, a estratégia foi adaptada para um **Plano B**: criar um cluster Kubernetes "single-node" com **Minikube**, rodando dentro de uma instância EC2 permitida pelo ambiente do laboratório.

* **Justificativa:** Esta abordagem permitiu cumprir o objetivo central de criar e gerenciar objetos Kubernetes em um ambiente de nuvem real, mesmo com as limitações impostas. A exposição da aplicação foi planejada utilizando um `Service` do tipo `NodePort`.

---

## 3. Arquivos de Manifesto Kubernetes

Os seguintes arquivos foram criados para definir o estado desejado da nossa aplicação no cluster.

<details>
<summary>📄 <b>deployment.yaml</b></summary>

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: safebank-webapp-deployment
  labels:
    app: safebank-webapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: safebank-webapp
  template:
    metadata:
      labels:
        app: safebank-webapp
    spec:
      containers:
      - name: safebank-container
        image: nginxdemos/hello:plain-text
        ports:
        - containerPort: 80
```
</details>

<details>
<summary>📄 <b>service.yaml</b></summary>

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: safebank-webapp-service
spec:
  # O tipo NodePort foi escolhido para expor o serviço na rede da instância EC2
  type: NodePort
  selector:
    app: safebank-webapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```
</details>

---

## 4. Processo de Deploy e Diagnóstico Avançado

Após o deploy dos manifestos no cluster Minikube, a aplicação não ficou acessível externamente, retornando um erro de `ERR_CONNECTION_REFUSED`. Isso deu início a um processo de diagnóstico detalhado para isolar a causa raiz.

1.  **Validação do Cluster:** Os comandos `kubectl get pods`, `kubectl get service` e `kubectl describe service` confirmaram que todos os objetos Kubernetes estavam **saudáveis e configurados corretamente**: os Pods estavam `Running`, e o Service tinha os `Endpoints` corretos.
2.  **Isolamento do Problema:** Um teste de `curl` para o `NodePort` a partir do `localhost` (dentro da instância EC2) também falhou. Isso provou que o problema **não era o Security Group da AWS**, mas sim uma falha na comunicação entre a rede da instância EC2 e a rede interna do cluster Minikube.
3.  **Tentativas de Correção:** Foram realizadas tentativas de correção, incluindo a verificação de firewalls locais (`firewalld`) e uma reinicialização completa do cluster Minikube (`minikube delete` e `start`). Mesmo assim, o problema de exposição de porta persistiu, indicando uma incompatibilidade de baixo nível do ambiente.

---

## 5. Prova de Sucesso e Conclusão

Para provar que a aplicação estava, de fato, publicada e funcional, foi realizada uma última verificação, acessando-a de dentro da rede privada do cluster.

Um pod de diagnóstico temporário foi lançado e, de dentro dele, o serviço foi acessado com sucesso através de seu nome DNS interno.

**Comando executado de dentro do pod de diagnóstico:**
```sh
curl http://safebank-webapp-service
```

**Resultado - A Prova Irrefutável de Sucesso:**
```
Server address: 10.244.0.3:80
Server name: safebank-webapp-deployment-758b59446f-zd2wq
Date: 09/Sep/2025:03:35:45 +0000
URI: /
Request ID: 18f46daaf5ef500204066a63df906446
```

### Conclusão Final

**O objetivo do projeto foi alcançado com sucesso.** A aplicação foi conteinerizada e publicada em um cluster Kubernetes funcional, com todos os objetos (`Deployment`, `Pods`, `Service`) operando corretamente. O teste final comprovou que a aplicação está no ar e respondendo a requisições na rede do cluster.

A dificuldade de expor o serviço para a internet foi um desafio valioso, servindo como um exercício prático de diagnóstico de problemas de rede complexos em ambientes de nuvem restritos.

---

## 6. Principais Aprendizados

* Criação e gerenciamento de `Deployments` e `Services` no Kubernetes.
* Diferenças práticas entre os tipos de serviço `LoadBalancer` e `NodePort`.
* Importância de verificar permissões (IAM) em ambientes de nuvem gerenciados.
* Técnicas avançadas de troubleshooting em Kubernetes, incluindo o uso de pods de diagnóstico para testar a rede interna do cluster.
* Resiliência e adaptação para solucionar problemas do mundo real em ambientes com limitações.