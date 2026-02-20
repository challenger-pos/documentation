# RFC 0001 – Decisões Técnicas de Arquitetura

## Status
**Aprovado / Atualizado (Tech Challenge)**

## Autor
Equipe de Arquitetura

## Data
2026

---

## 1. Contexto

A aplicação original foi concebida com uma arquitetura monolítica para facilitar a validação inicial do modelo de negócio (Sistema de Oficina Mecânica). [cite_start]Com o crescimento da complexidade, a necessidade de integração com provedores externos de pagamento e a exigência de maior escalabilidade, o projeto precisou evoluir.

Este documento registra as decisões tomadas durante o processo de estrangulamento (*Strangler Fig Pattern*) do monólito, resultando na atual arquitetura orientada a microsserviços.

## 2. Decisões Arquiteturais

### 2.1. Decomposição em Microsserviços (Domain-Driven Design)
[cite_start]Foi decidido dividir a aplicação em três microsserviços distintos, baseados nos subdomínios do negócio:
* [cite_start]**Work Order Service (Core):** Responsável por gerenciar Ordens de Serviço, Clientes, Veículos e Mecânicos. [cite_start]Centraliza o fluxo principal do negócio e evita que seja sobrecarregado com lógicas de terceiros.
* **Stock Service:** Responsável por gerenciar o catálogo de peças e o inventário. [cite_start]O controle de concorrência ganha escalabilidade independente, blindando o serviço core contra problemas de *locks* no banco de dados.
* **Payment Service:** Atua como uma Camada Anticorrupção (*Anti-Corruption Layer*) processando pagamentos e integrando com o provedor externo (Mercado Pago). [cite_start]Isola o sistema central de possíveis instabilidades e mudanças de contrato da API terceira.

### 2.2. API Gateway
[cite_start]Adotamos o uso de um API Gateway como único ponto de entrada unificado para clientes e front-ends externos.
* [cite_start]**Justificativa:** Facilita o roteamento das requisições síncronas (HTTP/REST) para os respectivos microsserviços  e abstrai a complexidade da rede interna do cluster Kubernetes.

### 2.3. Padrão Saga (Coreografia) com AWS SQS
[cite_start]Como o banco de dados foi segregado, o ciclo de vida de uma Ordem de Serviço passou a exigir uma **transação distribuída**  (criar a OS, reservar as peças e realizar a cobrança). [cite_start]Optamos pela abordagem de **Saga Coreografada**.
* [cite_start]**Justificativa:** Diferente da orquestração, a coreografia elimina o risco e gargalo de um ponto central único de falha. Os serviços assinam e publicam eventos de forma totalmente isolada.
* [cite_start]**Tecnologia:** Adotamos o **AWS SQS** como Message Broker para a comunicação assíncrona orientada a eventos, garantindo a tolerância a falhas (via Dead Letter Queues) para retentativas de processos como estorno de peças ou falhas de pagamento.

### 2.4. Segregação de Dados (*Database per Service*)
[cite_start]Cada microsserviço agora possui seu próprio banco de dados isolado.
* **PostgreSQL:** Escolhido para o `Work Order Service` e o `Stock Service`. [cite_start]A garantia de integridade ACID continua sendo essencial para o cadastro cruzado de clientes/veículos e para a reserva precisa de peças no inventário. [cite_start]O versionamento de migrações é gerenciado via **Liquibase**.
* **Amazon DynamoDB (NoSQL):** Escolhido para o `Payment Service`.
  * [cite_start]**Justificativa:** Necessidade de alta performance de leitura/escrita e adaptação nativa à natureza estruturada em documentos/JSON das transações financeiras e *payloads* flexíveis de logs do webhook do Mercado Pago.

### 2.5. Infraestrutura e Observabilidade
[cite_start]Para viabilizar a operação e a manutenibilidade da nova arquitetura distribuída, definimos a seguinte base:
* [cite_start]**Linguagem Principal:** Java com Spring Boot.
* **Containerização e Orquestração:** Docker e **Kubernetes (K8s)**. [cite_start]Faremos uso de *Deployments*, *Services* e HPA (*Horizontal Pod Autoscaler*) para garantir a alta disponibilidade.
* [cite_start]**Infrastructure as Code (IaC):** Uso de **Terraform** para o provisionamento dos ambientes e garantia de recursos reprodutíveis na nuvem AWS.
* [cite_start]**Monitoramento/Observabilidade:** Uso do **Datadog**, permitindo o rastreamento das requisições (*distributed tracing*) e o monitoramento da saúde dos três microsserviços no ecossistema.

## 3. Consequências

* **Positivas:**
  * Altíssimo isolamento de falhas: indisponibilidade no Mercado Pago ou queda do banco de estoque não impede que usuários continuem acessando ou consultando OSs finalizadas.
  * Escalabilidade granular: em momentos de pico, podemos escalar apenas o serviço necessário (ex: `Stock Service` durante entrada de inventário) sem gastar recursos com a aplicação inteira.
* **Negativas / Desafios:**
  * Complexidade operacional aumentada. A entrega agora depende de *pipelines* de CI/CD orquestradas para múltiplos repositórios.
  * O sistema agora lida com "consistência eventual". Operações não ocorrem mais em uma transação síncrona única, o que exige das interfaces client-side um tratamento mais inteligente (ex: status pendente aguardando a fila processar).