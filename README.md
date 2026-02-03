<img width="1400" height="350" alt="image" src="https://github.com/user-attachments/assets/da17a009-03f2-4605-a971-6a735277f96c" />

## Tech Stack

[![Microsoft Azure](https://img.shields.io/badge/Microsoft%20Azure-0078D4?logo=microsoftazure&logoColor=white)](https://azure.microsoft.com/)
[![Azure Databricks](https://img.shields.io/badge/Azure%20Databricks-FF3621?logo=databricks&logoColor=white)](https://learn.microsoft.com/azure/databricks/)
[![Azure Data Factory](https://img.shields.io/badge/Azure%20Data%20Factory-0078D4?logo=microsoftazure&logoColor=white)](https://learn.microsoft.com/azure/data-factory/)
[![ADLS Gen2](https://img.shields.io/badge/ADLS%20Gen2-0078D4?logo=microsoftazure&logoColor=white)](https://learn.microsoft.com/azure/storage/blobs/data-lake-storage-introduction/)
[![Apache Spark](https://img.shields.io/badge/Apache%20Spark-E25A1C?logo=apachespark&logoColor=white)](https://spark.apache.org/)
[![Delta Lake](https://img.shields.io/badge/Delta%20Lake-0A0A0A?logo=delta&logoColor=white)](https://delta.io/)
[![Unity Catalog](https://img.shields.io/badge/Unity%20Catalog-FF3621?logo=databricks&logoColor=white)](https://learn.microsoft.com/azure/databricks/data-governance/unity-catalog/)
[![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)](https://www.python.org/)


# Pipeline Imobiliário — Azure Databricks + Data Factory + Delta Lake (Bronze/Silver)

## Visão geral
Projeto de engenharia de dados que cria um Data Lake e processa dados de imóveis a partir da camada **Inbound**,
transformando-os nas camadas **Bronze** e **Silver** usando **Delta Lake** no Azure Databricks.
A orquestração é feita com Azure Data Factory (execução horária).

<img width="1277" height="171" alt="image" src="https://github.com/user-attachments/assets/720ac4d8-4343-444f-ae0c-b113deeda24d" />

## Arquitetura (Medallion)
Inbound (JSON) → Bronze (Delta raw) → Silver (Delta estruturado)

- **Inbound**: arquivo bruto (JSON) depositado no Data Lake
- **Bronze**: dados persistidos em Delta com mínima transformação
- **Silver**: dados “achatados” e prontos para análise (ex.: endereço aberto em colunas, remoção de campos)

## Tecnologias
- Azure Databricks (Spark + Delta)
- Azure Data Lake Storage Gen2
- Unity Catalog (governança de acesso)
- Azure Data Factory (orquestração)

## Estrutura do repositório
> Pastas presentes no repositório:
- `notebooks/` — notebooks do Databricks com as transformações (Inbound → Bronze, Bronze → Silver)
- `pipeline/` — definition(s) de pipeline do Data Factory
- `trigger/` — trigger(s) do Data Factory (agendamento)
- `linkedService/` — linked services (conexões) do Data Factory
- `factory/` — recursos do Data Factory (artefatos exportados)
- `data/` — dados de exemplo e/ou artefatos auxiliares


#  Quickstart (rodar do zero)

## Pré-requisitos
1. Workspace do Azure Databricks com Unity Catalog habilitado
2. Storage Account (ADLS Gen2) + container utilizado no projeto
3. Access Connector / Managed Identity com permissão no container:
   - role sugerida: **Storage Blob Data Contributor**
4. Storage Credential + External Location (Unity Catalog) apontando para o container do Data Lake

## Paths (exemplo)
> Ajuste para o seu storage/container.
- Inbound: `abfss://<container>@<storage>.dfs.core.windows.net/inbound/`
- Bronze (Delta): `abfss://<container>@<storage>.dfs.core.windows.net/bronze/dataset_imoveis`
- Silver (Delta): `abfss://<container>@<storage>.dfs.core.windows.net/silver/dataset_imoveis`

## Como executar (manual)
### 1) Subir arquivo no Inbound
Enviar o JSON bruto para a pasta `inbound/`.

### 2) Rodar notebook Inbound → Bronze
Executar o notebook correspondente em `notebooks/`.

Validações:
- `dbutils.fs.ls("<path bronze>")`
- existência de `<path bronze>/_delta_log`

### 3) Rodar notebook Bronze → Silver
Executar o notebook correspondente em `notebooks/`.

Validações:
- `dbutils.fs.ls("<path silver>/_delta_log")`
- `spark.read.format("delta").load("<path silver>").count()`

## Como executar (orquestrado pelo Data Factory)
1. Importar/publicar os artefatos do ADF contidos nas pastas:
   - `linkedService/`, `pipeline/`, `trigger/` (e `factory/` quando aplicável)
2. Publicar no Data Factory
3. Ativar trigger (agendamento)
4. Monitorar execuções no painel de Monitor do ADF

## Regras principais aplicadas nas transformações
### Bronze
- Persistência em Delta
- Ajustes mínimos e padronização de colunas

### Silver
- Expansão de struct: `anuncio.*` e `anuncio.endereco.*`
- Remoção de campos: `caracteristicas` e `endereco`
- Tratamento de arrays (quando necessário) para facilitar consumo analítico

## Troubleshooting
- **Scala not supported by Serverless compute**
  - Use notebooks em Python/SQL no Serverless ou mude para cluster clássico para Scala.
- **DELTA_DUPLICATE_COLUMNS_FOUND (id)**
  - Evite selecionar `id` duas vezes ao expandir `anuncio.*`.
- **Permissões/credencial default (UnauthorizedAccessException)**
  - Garanta Storage Credential + External Location corretas no Unity Catalog e permissões no ADLS.

## Boas práticas de custo (Azure credits)
- Use cluster pequeno + **Auto-termination (10–15 min)**
- Evite deixar cluster “idle”
- Serverless é ótimo para sessões curtas (principalmente Python/SQL)

