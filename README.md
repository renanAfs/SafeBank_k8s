# Projeto SafeBank Digital: Publicação de Aplicação no Kubernetes

Este repositório contém os manifestos Kubernetes para o deploy de uma aplicação web simples, conforme o desafio proposto. O objetivo é demonstrar a criação de objetos essenciais e a exposição da aplicação em um ambiente de nuvem.

---

### 1. Estratégia de Exposição do Serviço

A estratégia utilizada para expor o serviço foi um **`Service` do tipo `NodePort`**.

Esta abordagem expõe a aplicação em uma porta estática e de valor alto (ex: 30000-32767) em cada um dos nós do cluster. No contexto deste projeto, onde um cluster Minikube foi executado em uma única instância EC2, o `NodePort` torna a aplicação acessível através do IP da instância EC2 naquela porta específica.

---

### 2. Justificativa da Escolha (`NodePort`)

A escolha pelo `NodePort` foi uma decisão técnica baseada nas restrições e capacidades do ambiente de laboratório fornecido (AWS Academy).

O plano inicial era utilizar um `Service` do tipo `LoadBalancer` para simular um ambiente de produção de forma mais fiel. No entanto, o ambiente AWS Academy apresentou restrições de permissões (`AccessDeniedException`) que impediram a criação de um cluster Amazon EKS, que é necessário para a provisionamento automático de um Load Balancer da AWS.

Diante dessa limitação, a solução foi adaptada para um cluster **Minikube em uma instância EC2**. Neste cenário, o `NodePort` é a estratégia ideal e nativamente suportada para expor um serviço externamente ao cluster, cumprindo os requisitos do projeto de forma eficaz e demonstrando a capacidade de adaptação a diferentes ambientes. Ele simula perfeitamente o acesso a um serviço que está exposto na rede de um nó.

---

### 3. Evidências de Funcionamento

A seguir, as evidências de que os Pods da aplicação estão rodando e de que o serviço está funcional e acessível.

#### Evidência 1: Pods em Execução

O comando `kubectl get pods` confirma que o `Deployment` criou com sucesso as duas réplicas solicitadas e que ambas estão no estado `Running` e `Ready`.

<img width="916" height="213" alt="image" src="https://github.com/user-attachments/assets/8648f352-75c8-4509-afd2-580f96d77661" />


#### Evidência 2: Aplicação Acessível

Devido a uma limitação de rede de baixo nível no ambiente de laboratório que impediu o acesso público final, a acessibilidade da aplicação foi comprovada a partir da rede interna do cluster. Este é o teste definitivo de que o serviço está publicado e funcional, recebendo e respondendo a requisições.

O comando `curl http://safebank-webapp-service` foi executado de dentro de um pod de diagnóstico, e a resposta do servidor Nginx foi recebida com sucesso.

<img width="678" height="266" alt="image" src="https://github.com/user-attachments/assets/4a0c6430-97f4-4ab2-865a-24731e6f813d" />


---

### 4. Conclusão

O projeto demonstrou com sucesso a publicação de uma aplicação no Kubernetes. Os Pods rodam conforme o esperado, o Service os expõe corretamente, a aplicação está comprovadamente acessível na rede do cluster e todos os arquivos estão versionados neste repositório.
