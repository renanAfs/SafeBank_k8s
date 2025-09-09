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
        app: safeb