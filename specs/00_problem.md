# Problema: ETL Confiável para Marketplace Olist

## 1. Stakeholders
- **Operação Olist (negócio):** Depende dos dados para decisões comerciais.
- **Time de dados:** Responsável pela manutenção dos pipelines e modelos analíticos.
- **Clientes internos do dashboard:** Consumidores finais das métricas de vendas.
- **Plataforma / SRE:** Garante a infraestrutura, disponibilidade e observabilidade do sistema.

## 2. Fluxos Críticos
- **Ingestão diária CSV → Postgres:** Processamento de ~100 mil pedidos diários do Marketplace.
- **Consulta de dashboards Grafana:** Visualização em tempo real do estado da operação.
- **Observação de SLA:** Monitoramento da janela de entrega do dado (ex: dados disponíveis até as 08:00 AM).

## 3. Modos de Falha
- **Arquivo corrompido ou parcial:** Dados malformados ou incompletos na origem.
- **Reprocesso duplicando linhas:** Falta de idempotência ao reexecutar jobs falhos.
- **Queda de EC2 durante o run:** Interrupção abrupta do processamento.
- **Banco indisponível:** Falha de conexão ou timeout com o Postgres analítico.

## 4. Riscos Sistêmicos
- **Perda silenciosa de linhas:** Erros de parse que descartam dados sem alertar o time.
- **Dashboard mostrando dado stale:** Usuários tomando decisões baseadas em dados de ontem sem saber.
- **Custo AWS explodindo:** Loop de retentativas infinitas ou instâncias superdimensionadas.
- **Dívida técnica de IA mal revisada:** Código gerado automaticamente sem validação de casos de borda.
