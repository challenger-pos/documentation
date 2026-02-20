## Diagrama de Componentes

O diagrama abaixo apresenta a visão atualizada de componentes da aplicação distribuída, refletindo a transição para microsserviços. O ecossistema inclui AWS API Gateway, Cluster Kubernetes (EKS), mensageria via AWS SQS para orquestração da Saga, segregação de bancos de dados (RDS PostgreSQL e DynamoDB), e monitoramento integrado via Datadog.

![Diagrama de Componentes](images/Diagrama%20de%20componente.png)