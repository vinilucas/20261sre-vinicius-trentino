# Requisitos Funcionais - ETL Olist

Este documento detalha os requisitos funcionais para o pipeline de ETL do Marketplace Olist, conforme elicitação realizada com base no problema de negócio.

### 📌 Resumo do Problema
O sistema consiste em um pipeline de dados (ETL) responsável por ingerir diariamente cerca de 100.000 pedidos do Marketplace Olist a partir de arquivos CSV para um banco de dados PostgreSQL. O foco principal é a confiabilidade, garantindo a idempotência, a integridade dos dados e a visibilidade do estado do processamento para os stakeholders através de dashboards e alertas de SLA.

### 👥 Atores Identificados
*   **Time de Dados (Executor):** Responsável por operar e monitorar o pipeline.
*   **Sistema de Origem (Externo):** Provedor dos arquivos CSV.
*   **Banco de Dados Analítico (Externo):** Destino dos dados processados (PostgreSQL).
*   **Grafana (Consumidor):** Sistema de visualização que consome os dados e métricas de saúde.
*   **Plataforma / SRE (Monitor):** Garante a infraestrutura e observa o cumprimento dos SLAs.

---

### 📋 Requisitos Funcionais

**Categoria: Ingestão de Dados**

- **RF01: Ingestão de Dados CSV**
    - **Descrição:** O sistema deve ser capaz de ler arquivos CSV contendo os pedidos do marketplace e carregar os dados no banco PostgreSQL.
    - **Atores:** Sistema de Origem, Time de Dados.
    - **Entrada:** Arquivos CSV (~100 mil linhas/dia).
    - **Saída:** Dados inseridos nas tabelas correspondentes no PostgreSQL.
    - **Regras de Negócio:** O processo deve ocorrer diariamente para garantir que os dados estejam disponíveis até as 08:00 AM.

- **RF02: Garantia de Idempotência**
    - **Descrição:** O sistema deve ser capaz de reprocessar arquivos sem gerar duplicidade de registros no banco de dados.
    - **Atores:** Time de Dados.
    - **Entrada:** Identificador do lote ou arquivo para reprocessamento.
    - **Saída:** Banco de dados atualizado sem linhas duplicadas.
    - **Regras de Negócio:** Em caso de falha e reinicialização, o sistema deve sobrescrever ou ignorar registros já existentes baseando-se em chaves únicas.

- **RF03: Validação de Integridade na Origem**
    - **Descrição:** O sistema deve validar a estrutura e o conteúdo dos arquivos CSV antes de iniciar a carga (check de arquivo corrompido ou parcial).
    - **Atores:** Time de Dados.
    - **Entrada:** Arquivo CSV original.
    - **Saída:** Log de erro ou sinalização de "Pronto para Ingestão".
    - **Regras de Negócio:** Se o arquivo estiver malformado ou incompleto, a ingestão deve ser interrompida e o erro notificado.

**Categoria: Observabilidade e Alertas**

- **RF04: Monitoramento de Status do Job**
    - **Descrição:** O sistema deve reportar o estado atual do processamento (Iniciado, Em Andamento, Sucesso, Falha) para consumo pelo Grafana.
    - **Atores:** Time de Dados, Grafana.
    - **Entrada:** Eventos internos do pipeline.
    - **Saída:** Métricas/Logs de status.
    - **Regras de Negócio:** Toda execução deve gerar um rastro que permita saber se o dado é "fresco" ou "stale".

- **RF05: Alerta de Violação de SLA**
    - **Descrição:** O sistema deve ser capaz de notificar o time de SRE/Dados caso o processamento não seja concluído até o horário limite estabelecido.
    - **Atores:** Plataforma / SRE, Time de Dados.
    - **Entrada:** Horário atual vs. Status da Ingestão.
    - **Saída:** Notificação/Alerta.
    - **Regras de Negócio:** O SLA crítico é 08:00 AM.

- **RF06: Notificação de Perda de Linhas (Erro de Parse)**
    - **Descrição:** O sistema deve contabilizar e alertar caso linhas sejam descartadas durante o processo de transformação/parse por erros de tipo ou dados inválidos.
    - **Atores:** Time de Dados.
    - **Entrada:** Linhas lidas vs. Linhas inseridas.
    - **Saída:** Alerta de "Silent Loss" detectada.
    - **Regras de Negócio:** Evitar a perda silenciosa; qualquer descarte acima de um limiar (threshold) deve gerar intervenção manual.

- **RF10: Reconciliação Fim-a-Fim (Row Count)**
    - **Descrição:** O sistema deve realizar uma contagem total de linhas no arquivo de origem e compará-la com o total de linhas inseridas + linhas descartadas no destino.
    - **Atores:** Time de Dados, Clientes Internos.
    - **Entrada:** CSV de origem e Banco de Dados.
    - **Saída:** Relatório de reconciliação.
    - **Regras de Negócio:** Mitiga o risco de perda silenciosa não detectada apenas pelo parse.

- **RF11: Persistência de Metadados de Freshness (Watermarking)**
    - **Descrição:** O sistema deve registrar a data/hora da última carga bem-sucedida para que dashboards identifiquem dados "stale".
    - **Atores:** Grafana, Time de Dados.
    - **Entrada:** Eventos de finalização de carga.
    - **Saída:** Registro de timestamp no banco.
    - **Regras de Negócio:** Permite visibilidade sobre a atualidade do dado para o negócio.

**Categoria: Resiliência e Recuperação**

- **RF07: Recuperação de Falha de Conexão**
    - **Descrição:** O sistema deve implementar tentativas de reconexão (retry) com backoff exponencial para o PostgreSQL.
    - **Atores:** Banco de Dados Analítico.
    - **Entrada:** Falha de conexão/Timeout.
    - **Saída:** Retomada da execução.
    - **Regras de Negócio:** Limitar retentativas para evitar explosão de custos (conforme RF08).

- **RF09: Controle de Concorrência (Locking)**
    - **Descrição:** O sistema deve impedir que duas instâncias do mesmo job rodem simultaneamente para o mesmo período/arquivo.
    - **Atores:** Time de Dados.
    - **Entrada:** Estado de execução/Lock system.
    - **Saída:** Bloqueio de execução duplicada.
    - **Regras de Negócio:** Garante integridade e evita contenção de recursos.

- **RF13: Handshake de Desligamento Gracioso (Graceful Shutdown)**
    - **Descrição:** O sistema deve capturar sinais de terminação da instância (SIGTERM) e encerrar conexões de forma limpa.
    - **Atores:** Plataforma / SRE.
    - **Entrada:** Sinais do Sistema Operacional.
    - **Saída:** Logs de encerramento seguro.

**Categoria: Governança e Qualidade**

- **RF08: Limite de Tempo de Execução (Timeout)**
    - **Descrição:** O sistema deve interromper automaticamente o processo caso ele ultrapasse um tempo limite pré-definido.
    - **Atores:** Time de Dados, Plataforma / SRE.
    - **Entrada:** Timer de execução.
    - **Saída:** Interrupção e alerta.
    - **Regras de Negócio:** Evita custos excessivos por jobs travados ou loops infinitos.

- **RF12: Validação de Esquema e Tipagem Estrita**
    - **Descrição:** O sistema deve validar se os tipos de dados nas colunas do CSV condizem com o contrato esperado (ex: datas, preços).
    - **Atores:** Time de Dados.
    - **Entrada:** Dados brutos do CSV.
    - **Saída:** Dados validados ou desvio para erro.
    - **Regras de Negócio:** Garante que casos de borda ou dados malformados não corrompam o banco analítico.

---

### ❓ Perguntas para Refinamento

1.  **Formato do CSV:** Os arquivos CSV possuem um esquema fixo? Existe algum dicionário de dados ou cabeçalho padrão?
2.  **Identificador Único:** Qual é a chave primária (ex: `order_id`) que deve ser usada para garantir a idempotência no PostgreSQL?
3.  **Janela de Ingestão:** Qual é o horário de início da ingestão (quando os arquivos ficam disponíveis)?
4.  **Threshold de Erro:** Qual a porcentagem aceitável de linhas descartadas antes de considerar o job como "Falha Crítica"?
5.  **Destino dos Logs:** Onde os logs detalhados de erros de parse devem ser armazenados para auditoria (S3, CloudWatch, Tabela de Erros)?
6.  **Dead Letter Queue (DLQ):** Linhas que falham na validação (RF12) devem ser salvas em local separado para análise posterior?
7.  **Notificação de Sucesso:** Os stakeholders da Operação desejam receber confirmação ativa (ex: Slack) quando os dados estiverem prontos às 08:00 AM?
