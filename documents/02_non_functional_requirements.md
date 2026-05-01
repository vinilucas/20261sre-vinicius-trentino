# Requisitos Não Funcionais - ETL Olist

Este documento detalha os requisitos não funcionais (RNFs) para o pipeline de ETL, estruturados conforme a norma ISO 25010 e priorizados pela metodologia MoSCoW.

### 📌 Análise de Atributos ISO 25010

#### 1. Adequação Funcional (Functional Suitability)
- **RNF-01**: **Integridade de Dados (Reconciliação)**. O sistema deve garantir que 100% das linhas do CSV de origem sejam contabilizadas (como processadas ou movidas para DLQ).
- **Prioridade**: Must-have
- **Rationale**: Mitiga o risco de "Perda silenciosa de linhas" reportado pelos stakeholders.

#### 2. Eficiência de Desempenho (Performance Efficiency)
- **RNF-02**: **Tempo de Ingestão**. O processamento de 100.000 linhas deve ser concluído em menos de 60 minutos em uma instância t3.medium.
- **Prioridade**: Must-have
- **Rationale**: Garante o cumprimento do SLA das 08:00 AM, mesmo que o arquivo chegue próximo ao horário limite.

#### 3. Confiabilidade (Reliability)
- **RNF-03**: **Idempotência de Reprocessamento**. Repetições de carga para o mesmo arquivo não devem resultar em registros duplicados (Erro de duplicidade = 0).
- **Prioridade**: Must-have
- **Rationale**: Evita a falha crítica de "Reprocesso duplicando linhas".

- **RNF-04**: **Recuperação de Falha (MTTR)**. Em caso de queda da instância EC2, o tempo de reinicialização e retomada do job não deve exceder 15 minutos.
- **Prioridade**: Should-have
- **Rationale**: Minimiza o impacto de interrupções abruptas.

#### 4. Segurança (Security)
- **RNF-05**: **Gestão de Segredos**. Nenhuma credencial de banco de dados deve estar em texto claro no código ou logs; 100% dos segredos devem vir do AWS Secrets Manager.
- **Prioridade**: Must-have
- **Rationale**: Proteção de dados sensíveis e conformidade com boas práticas de SRE.

#### 5. Manutenibilidade (Maintainability)
- **RNF-06**: **Rastreabilidade de Erros**. 100% das falhas de parse ou conexão devem gerar logs estruturados com o identificador da linha/arquivo afetado.
- **Prioridade**: Must-have
- **Rationale**: Facilita a manutenção pelo Time de Dados e evita a "Dívida técnica".

#### 6. Usabilidade (Usability)
- **RNF-07**: **Atualização de Dashboards**. Os dados processados devem estar refletidos no Grafana em no máximo 5 minutos após o término do job.
- **Prioridade**: Should-have
- **Rationale**: Garante que o dashboard não mostre dados "stale" para a Operação Olist.

#### 7. Compatibilidade (Compatibility)
- **RNF-08**: **Interoperabilidade de Banco**. O sistema deve ser compatível com as versões 13 a 16 do PostgreSQL.
- **Prioridade**: Could-have
- **Rationale**: Garante longevidade tecnológica e flexibilidade para atualizações de infraestrutura.

#### 8. Portabilidade (Portability)
- **RNF-09**: **Infraestrutura como Código (IaC)**. O ambiente (EC2/Postgres) deve ser reprodutível via Terraform/CloudFormation em menos de 30 minutos.
- **Prioridade**: Should-have
- **Rationale**: Facilita a recuperação de desastres e paridade entre ambientes.

---

### 📊 Matriz de Mensurabilidade

| ID | Atributo | SLI (Indicador) | SLO (Objetivo) | Fonte de Medição | Prioridade |
|:---|:---|:---|:---|:---|:---|
| **RNF-01** | Adequação | Taxa de reconciliação de linhas | 100% | Log de Auditoria / SQL | Must |
| **RNF-02** | Desempenho | Duração do processamento (100k) | < 60 minutos | CloudWatch Logs | Must |
| **RNF-03** | Confiabilidade | Contagem de linhas duplicadas | 0 duplicatas | SQL Audit Query | Must |
| **RNF-04** | Confiabilidade | Tempo de recuperação (MTTR) | < 15 minutos | CloudWatch Metrics | Should |
| **RNF-05** | Segurança | Segredos expostos em logs/código | 0 ocorrências | GitGuardian / Scan | Must |
| **RNF-06** | Manutenc. | Cobertura de logs de erro | 100% das exceções | Sentry / CloudWatch | Must |
| **RNF-07** | Usabilidade | Latência de atualização do Grafana | < 5 minutos | Grafana Trace | Should |
| **RNF-08** | Compatibil. | Sucesso em testes multi-versão | 100% pass rate | CI/CD Pipeline | Could |
| **RNF-09** | Portabilid. | Tempo de deploy IaC | < 30 minutos | Terraform Output | Should |

---

### 🛠️ Premissas & Pré-requisitos
- A instância EC2 possui conectividade estável com o AWS Secrets Manager.
- O volume de dados mantém a média de 100 mil linhas (picos acima de 500k podem exigir revisão do SLO de tempo).
- O banco de dados PostgreSQL possui índices adequados nas chaves de idempotência.
