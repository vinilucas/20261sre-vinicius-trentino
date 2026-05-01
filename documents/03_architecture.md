# Arquitetura do Sistema - ETL Olist (RM-ODP)

Este documento descreve a arquitetura do sistema de ETL para o Marketplace Olist, seguindo o framework RM-ODP e respeitando as restrições do AWS Academy Learner Lab.

## 1. Enterprise Viewpoint
O objetivo do sistema é garantir que os dados de vendas do Marketplace Olist sejam processados e disponibilizados para análise de forma confiável e pontual.
- **Atores de Negócio:** Operação Olist, Time de Dados, SRE.
- **Processo Principal:** Ingestão diária de 100k pedidos com SLA de entrega até as 08:00 AM.
- **Conformidade:** Garantia de integridade e visibilidade (observabilidade).

## 2. Information Viewpoint
Define a estrutura dos dados e as regras de transformação.
- **Modelagem:**
    - **Source:** Arquivos CSV (Raw Data).
    - **Target:** Esquema relacional no PostgreSQL (Analytical Data).
    - **Metadata:** Tabela de `watermarking` para controle de carga e `audit_log` para reconciliação.
- **Fluxo de Informação:** CSV -> Validação -> Transformação -> Carga (Upsert).

## 3. Computational Viewpoint
Descreve os componentes funcionais e suas interfaces.
- **ETL Script (Python):** Componente central responsável por RF01, RF02, RF03, RF06, RF07, RF08, RF10, RF12.
- **Scheduler (Cron/Systemd):** Orquestra a execução diária (RF01, RF05).
- **Database Engine (Postgres):** Armazena dados e metadados (RF02, RF11).
- **Monitoring Agent:** Coleta métricas para o Grafana (RF04, RF05).

## 4. Engineering Viewpoint
Descreve a infraestrutura e os mecanismos de distribuição.
- **EC2 Instance (Python Runner):** Executa o script ETL. Atende RNF-02 (Desempenho) e RNF-04 (MTTR).
- **RDS PostgreSQL:** Instância gerenciada para o banco analítico. Atende RNF-03 (Idempotência via Constraints) e RNF-08 (Versões).
- **S3 Bucket:** Armazenamento dos arquivos CSV de origem (Staging Area).
- **CloudWatch Logs/Metrics:** Centraliza logs e métricas. Atende RNF-06 (Rastreabilidade).
- **Secrets Manager:** Armazena credenciais do banco. Atende RNF-05 (Segurança).

## 5. Technology Viewpoint
Tecnologias específicas utilizadas no AWS Academy.
- **Linguagem:** Python 3.11 (Pandas/SQLAlchemy).
- **Banco de Dados:** AWS RDS PostgreSQL 15.
- **Infraestrutura:** AWS EC2 (t3.medium - Linux).
- **Orquestração:** Systemd Timers (nativo Linux).
- **IaC:** Terraform 1.5+.

---

## Architecture Decision Records (ADR)

### ADR 01: Uso de Python em EC2 para o Processo ETL
- **Contexto:** Necessidade de processar 100k linhas diariamente no AWS Academy, sem acesso ao AWS Glue.
- **Decisão:** Utilizar um script Python executado em uma instância EC2 t3.medium.
- **Consequência:** Maior controle sobre o código e custos (respeitando limites do Learner Lab), mas exige gestão manual do servidor e do scheduler.

### ADR 02: Implementação de Idempotência via `ON CONFLICT` (Upsert)
- **Contexto:** RF02 exige que reprocessamentos não dupliquem dados.
- **Decisão:** Utilizar a cláusula `ON CONFLICT` do PostgreSQL no momento da carga.
- **Consequência:** Garante integridade atômica no nível do banco, simplificando a lógica do script ETL e reduzindo o risco de duplicidade silenciosa.

### ADR 03: Uso de Systemd Timers para Agendamento
- **Contexto:** Necessidade de disparar o job diariamente e garantir o SLA das 08:00 AM (RF01, RF05).
- **Decisão:** Utilizar Systemd Timers em vez de Cron convencional ou Airflow (não disponível/complexo para o escopo).
- **Consequência:** Melhor integração com o logging do sistema (journald) e maior confiabilidade na reinicialização pós-falha (RNF-04).

---

## Mapeamento de Requisitos

| Componente | RFs Atendidos | RNFs Atendidos |
|:---|:---|:---|
| **Python ETL Script** | RF01, RF02, RF03, RF06, RF07, RF08, RF10, RF12 | RNF-01, RNF-02, RNF-06 |
| **RDS PostgreSQL** | RF01, RF02, RF11 | RNF-03, RNF-08 |
| **CloudWatch / Grafana** | RF04, RF05 | RNF-07 |
| **Secrets Manager** | - | RNF-05 |
| **Terraform** | - | RNF-09 |
