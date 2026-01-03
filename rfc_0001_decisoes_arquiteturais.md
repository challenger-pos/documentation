# RFC 0001 – Decisões Técnicas de Arquitetura

## Status
**Aprovado**

## Autor
Equipe de Arquitetura

## Data
2026-01-02

---

## 1. Contexto

Este documento tem como objetivo registrar e justificar decisões técnicas relevantes adotadas no projeto, seguindo o modelo de **RFC (Request for Comments)**. O uso de RFCs permite documentar escolhas arquiteturais de forma clara, rastreável e alinhada às boas práticas de engenharia de software.

As decisões aqui descritas impactam diretamente aspectos de **segurança, escalabilidade, confiabilidade e manutenção** da solução.

---

## 2. Decisão 1 – Escolha da Nuvem: AWS (Amazon Web Services)

### 2.1 Decisão

Utilizar a **AWS (Amazon Web Services)** como provedora de infraestrutura em nuvem do projeto.

### 2.2 Justificativa

A AWS foi escolhida por ser uma das plataformas de nuvem mais consolidadas e amplamente utilizadas no mundo, adotada por empresas de diversos portes e segmentos. Os principais fatores que motivaram essa decisão são:

- **Alta confiabilidade e disponibilidade**, com infraestrutura distribuída globalmente (Regions e Availability Zones).
- **Modelo de segurança robusto**, baseado em IAM, VPC, criptografia em repouso e em trânsito, além de conformidade com padrões internacionais (ISO, SOC, PCI-DSS, entre outros).
- **Ecossistema maduro de serviços gerenciados**, reduzindo a sobrecarga operacional.
- **Escalabilidade sob demanda**, permitindo crescimento do sistema sem necessidade de reestruturações complexas.

### 2.3 Consequências

- Dependência do ecossistema AWS (lock-in aceitável para o contexto do projeto).
- Redução de esforço operacional e maior foco no desenvolvimento da aplicação.

---

## 3. Decisão 2 – Banco de Dados: PostgreSQL

### 3.1 Decisão

Adotar o **PostgreSQL** como banco de dados relacional principal do projeto.

### 3.2 Justificativa

O PostgreSQL foi escolhido por ser um banco de dados **open source, maduro e amplamente utilizado em ambientes corporativos**, oferecendo:

- **Alta robustez e estabilidade**, comprovadas por anos de uso em sistemas críticos.
- **Recursos avançados de segurança**, como controle granular de permissões, autenticação forte e suporte a criptografia.
- **Excelente suporte a integridade de dados**, transações ACID e consistência.
- **Compatibilidade com serviços gerenciados da AWS**, como o Amazon RDS, facilitando backup, monitoramento e escalabilidade.

### 3.3 Consequências

- Curva de aprendizado moderada para equipes sem experiência prévia.
- Forte confiabilidade dos dados e facilidade de manutenção a longo prazo.

---

## 4. Decisão 3 – Estratégia de Autenticação e Autorização

### 4.1 Decisão

Implementar uma estratégia de autenticação baseada em:

- **AWS API Gateway** como ponto de entrada das requisições
- **AWS Lambda** responsável pela geração e validação de tokens
- **JWT (JSON Web Token)** como mecanismo de autenticação

### 4.2 Arquitetura Proposta

1. O cliente realiza uma requisição de autenticação via **API Gateway**.
2. O API Gateway encaminha a requisição para uma **Lambda Function**.
3. A Lambda valida as credenciais do usuário.
4. Em caso de sucesso, é gerado um **token JWT** assinado.
5. O token é retornado ao cliente e utilizado nas requisições subsequentes.

### 4.3 Justificativa

Essa abordagem foi escolhida pelos seguintes motivos:

- **Desacoplamento da lógica de autenticação** da aplicação principal.
- **Escalabilidade automática**, inerente ao modelo serverless das Lambdas.
- **Redução de custos**, pois o modelo é baseado em uso sob demanda.
- **JWT é stateless**, eliminando a necessidade de armazenamento de sessão no servidor.
- **Integração nativa com serviços AWS**, incluindo authorizers no API Gateway.

### 4.4 Consequências

- Necessidade de cuidado com a gestão de chaves e tempo de expiração do token.
- Maior segurança e padronização do fluxo de autenticação.

---

## 5. Considerações Finais

As decisões documentadas neste RFC priorizam **segurança, confiabilidade, escalabilidade e manutenibilidade**, alinhando o projeto a práticas modernas de arquitetura em nuvem e desenvolvimento de software.

Este documento deve ser atualizado sempre que decisões arquiteturais relevantes forem revisadas ou alteradas.

---

## 6. Referências

- https://aws.amazon.com
- https://www.postgresql.org
- https://jwt.io

