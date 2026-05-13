# Apache Spark e Delta Lake

## Apache Spark

### O que é?

**Apache Spark** é um **engine de processamento de dados distribuído**. Ele divide o trabalho entre vários computadores (ou núcleos) ao mesmo tempo, o que permite processar grandes volumes de dados muito mais rápido do que um único computador conseguiria.

### Por que usar Spark?

| Característica | Detalhe |
|---|---|
| **Distribuído** | Processa dados em paralelo em múltiplas máquinas |
| **Em memória** | Guarda dados na RAM durante o processamento (muito mais rápido que disco) |
| **Multi-linguagem** | Suporta Python (PySpark), SQL, Scala, Java, R |
| **Open Source** | Mantido pela Apache Software Foundation |

### PySpark

**PySpark** é a API do Spark para Python. Permite escrever código Python que é executado de forma distribuída pelo Spark.

Neste projeto, o PySpark é usado para:
- Ler CSVs do Landing Volume
- Transformar DataFrames (adicionar colunas, renomear, filtrar)
- Escrever tabelas Delta Lake

### DataFrame

Um **DataFrame** é a principal estrutura de dados do Spark. É como uma tabela em memória — tem linhas e colunas — mas pode estar distribuída em vários computadores.

```python
# Criando um DataFrame lendo um CSV
df = spark.read.option("header", "true").csv("/Volumes/workspace/landing/dados/reserva")

# Adicionando uma coluna
df = df.withColumn("data_hora_bronze", current_timestamp())

# Salvando como tabela Delta
df.write.format("delta").saveAsTable("bronze.reserva")
```

!!! warning "DataFrame não persiste sozinho"
    Um DataFrame existe apenas na memória durante a execução do notebook. Para persistir, é necessário salvá-lo como tabela (`saveAsTable`) ou arquivo.

---

## Delta Lake

### O que é?

**Delta Lake** é um formato de armazenamento de dados que adiciona funcionalidades avançadas em cima de arquivos Parquet. É o formato padrão do Databricks e o coração de um Lakehouse.

### Por que não usar CSV ou Parquet simples?

| Recurso | CSV | Parquet | Delta Lake |
|---|---|---|---|
| Leitura rápida | ❌ | ✅ | ✅ |
| Compressão | ❌ | ✅ | ✅ |
| Transações ACID | ❌ | ❌ | ✅ |
| INSERT/UPDATE/DELETE | ❌ | ❌ | ✅ |
| Histórico de versões | ❌ | ❌ | ✅ |
| Schema enforcement | ❌ | Parcial | ✅ |
| Time Travel | ❌ | ❌ | ✅ |

### Transações ACID

ACID garante que operações no banco sejam seguras:

- **A**tomicidade — ou tudo acontece, ou nada (sem dados pela metade)
- **C**onsistência — os dados sempre ficam em estado válido
- **I**solamento — operações paralelas não se interferem
- **D**urabilidade — dados salvos ficam salvos mesmo com falhas

### Delta Log

O Delta Lake mantém um arquivo `_delta_log/` com todo o histórico de operações. Isso permite:

```sql
-- Ver o histórico de uma tabela
DESCRIBE HISTORY bronze.reserva

-- Ler versão anterior (Time Travel)
SELECT * FROM bronze.reserva VERSION AS OF 1
```

### MERGE INTO (SCD Tipo 1)

O `MERGE INTO` é usado na camada Gold para atualizar registros existentes e inserir novos — sem duplicar dados:

```sql
MERGE INTO gold.dim_cliente AS destino
USING novos_clientes AS origem
ON destino.id_cliente = origem.id_cliente
WHEN MATCHED THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *
```

### Tabelas Managed vs External

| Tipo | Quem controla os dados | Onde ficam |
|---|---|---|
| **Managed** (Gerenciada) | Databricks | Armazenamento interno do Databricks |
| **External** (Externa) | Você | Caminho que você define |

Neste projeto, todas as tabelas são **Managed** — o Databricks gerencia onde os dados ficam armazenados.
