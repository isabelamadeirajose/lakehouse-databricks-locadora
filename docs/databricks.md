# Databricks

## O que é Databricks?

**Databricks** é uma plataforma em nuvem para Engenharia de Dados e Machine Learning, construída sobre o Apache Spark. Ela oferece um ambiente gerenciado onde é possível criar clusters, executar notebooks, orquestrar pipelines e gerenciar dados — tudo sem precisar instalar nada no computador.

Databricks foi criado pelos próprios criadores do Apache Spark.

---

## Por que Databricks?

| Recurso | Benefício |
|---|---|
| **Notebooks interativos** | Escrever e executar código PySpark/SQL em células |
| **Cluster gerenciado** | Sem necessidade de configurar servidores |
| **Delta Lake nativo** | Suporte completo ao formato Delta |
| **Unity Catalog** | Gerenciamento centralizado de dados (schemas, tabelas, volumes) |
| **Jobs & Pipelines** | Orquestração automática de pipelines |
| **Free Edition** | Versão gratuita para estudos e projetos |

---

## Unity Catalog

O **Unity Catalog** é o sistema de governança de dados do Databricks. Ele organiza os dados em uma hierarquia de 3 níveis:

```
Catalog (workspace)
    └── Schema (banco de dados)
            └── Table / Volume
```

Neste projeto:
- **Catalog:** `workspace` (padrão do Free Edition)
- **Schemas:** `landing`, `bronze`, `silver`, `gold`
- **Volume:** `workspace.landing.dados` (para os arquivos CSV)
- **Tabelas:** Delta Lake em cada schema

### Schemas criados

```sql
CREATE SCHEMA IF NOT EXISTS workspace.landing;
CREATE SCHEMA IF NOT EXISTS workspace.bronze;
CREATE SCHEMA IF NOT EXISTS workspace.silver;
CREATE SCHEMA IF NOT EXISTS workspace.gold;
```

### Volume

Um **Volume** é um espaço de armazenamento de arquivos dentro do Unity Catalog. Diferente de tabelas (que armazenam dados estruturados), volumes armazenam arquivos brutos (CSV, JSON, imagens, etc.).

```sql
CREATE VOLUME IF NOT EXISTS workspace.landing.dados;
```

Os arquivos CSV extraídos ficam em: `/Volumes/workspace/landing/dados/`

---

## Notebooks

Os notebooks no Databricks são documentos interativos divididos em **células**, onde cada célula pode conter:
- Código Python (`%python` ou padrão)
- SQL (`%sql`)
- Markdown (`%md`)

### Exemplo de célula Python
```python
df = spark.read.format("delta").table("bronze.reserva")
display(df)
```

### Exemplo de célula SQL
```sql
%sql
SHOW TABLES IN bronze
```

---

## Jobs & Pipelines (Workflows)

O **Databricks Jobs** é o sistema de orquestração da plataforma. Permite agendar e encadear notebooks em sequência, formando uma pipeline automatizada.

### Job criado: `lakehouse-locadora`

```
Task 1: 001-preparando-ambiente  ──►
Task 2: 002-extracao             ──►
Task 3: 003-bronze               ──►
Task 4: 004-silver               ──►
Task 5: 005-gold
```

Cada task só inicia após a anterior ter concluído com sucesso. Se uma task falhar, o job para e registra o erro.

**Tempo total de execução:** ~3 minutos e 8 segundos.

---

## dbutils

`dbutils` é um conjunto de utilitários do Databricks para operações no sistema de arquivos:

```python
# Listar arquivos em um volume
dbutils.fs.ls('/Volumes/workspace/landing/dados/')

# Remover arquivos
dbutils.fs.rm('/Volumes/workspace/landing/dados/reserva', recurse=True)
```

É como o `ls` e `rm` do Linux, mas para o sistema de arquivos do Databricks.
