# Arquitetura Medalhão

A **Arquitetura Medalhão** (Medallion Architecture) é um padrão de organização de dados em camadas, onde os dados ficam progressivamente mais limpos, tratados e prontos para análise conforme avançam pelas camadas.

O nome vem das medalhas olímpicas: **Bronze → Silver → Gold**.

---

## Visão Geral das Camadas

```
PostgreSQL (Supabase)
        │
        ▼
┌─────────────────────────────────────────────┐
│                  LANDING                    │
│   Arquivos CSV brutos, sem transformação    │
│   Armazenados em Volume (não são tabelas)   │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│                  BRONZE                     │
│   Delta Lake — dados brutos + metadados     │
│   data_hora_bronze, nome_arquivo            │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│                  SILVER                     │
│   Delta Lake — Data Quality aplicada        │
│   Colunas padronizadas (uppercase,          │
│   abreviações expandidas)                   │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│                   GOLD                      │
│   Modelagem Dimensional — Star Schema       │
│   dim_cliente, dim_carro, dim_localidade,   │
│   dim_tempo, fato_reserva                   │
└─────────────────────────────────────────────┘
```

---

## Camada Landing

**O que é:** Área de pouso dos dados brutos. Os arquivos chegam exatamente como estão na fonte — sem nenhuma transformação.

**Formato:** CSV (arquivos de texto)

**Onde fica:** `Volumes/workspace/landing/dados/` — um Volume do Unity Catalog, não uma tabela Delta.

**Notebook responsável:** `002-extracao`

!!! note "Por que CSV?"
    O Landing serve como "cópia fiel" da fonte. Guardar em CSV garante que temos os dados originais preservados, independente de qualquer processamento posterior.

---

## Camada Bronze

**O que é:** Primeira camada Delta Lake. Os dados do Landing são lidos e salvos como tabelas Delta, com a adição de **metadados de rastreabilidade**.

**Formato:** Delta Lake (tabelas gerenciadas)

**Metadados adicionados:**
- `data_hora_bronze` — quando o dado foi ingerido no Bronze
- `nome_arquivo` — de qual arquivo CSV veio

**Notebook responsável:** `003-bronze`

```python
df = df.withColumn("data_hora_bronze", current_timestamp())
       .withColumn("nome_arquivo", lit("reserva.csv"))

df.write.format("delta").mode("overwrite").saveAsTable("bronze.reserva")
```

!!! info "Por que Delta Lake?"
    Delta Lake permite controle de versão, rollback, e suporta operações ACID (Atomicidade, Consistência, Isolamento, Durabilidade) — garantias que arquivos CSV comuns não têm.

---

## Camada Silver

**O que é:** Camada de **qualidade de dados**. Os dados do Bronze são padronizados e limpos.

**Transformações aplicadas:**
1. Colunas renomeadas para **UPPERCASE**
2. Abreviações expandidas: `CD_` → `CODIGO_`, `VL_` → `VALOR_`, `DT_` → `DATA_`, `NM_` → `NOME_`, `DS_` → `DESCRICAO_`, `NR_` → `NUMERO_`
3. Metadados do Bronze removidos (`data_hora_bronze`, `nome_arquivo`)
4. Novos metadados adicionados: `NOME_TABELA_BRONZE`, `DATA_HORA_SILVER`

**Notebook responsável:** `004-silver`

```python
def renomear(col):
    n = col.upper()
    n = n.replace("CD_", "CODIGO_")
    n = n.replace("VL_", "VALOR_")
    n = n.replace("DT_", "DATA_")
    return n
```

---

## Camada Gold

**O que é:** Camada analítica final. Os dados do Silver são reorganizados em um **modelo dimensional** seguindo a metodologia de Ralph Kimball.

**Tabelas criadas:**
- `dim_cliente` — quem alugou
- `dim_carro` — qual veículo foi alugado
- `dim_localidade` — onde foi alugado (agência, cidade, estado)
- `dim_tempo` — quando (calendário de 2023 a 2026)
- `fato_reserva` — o evento central (cada reserva com suas medidas)

**Notebook responsável:** `005-gold`

**Técnica usada:** SCD Tipo 1 via `MERGE INTO` — quando um registro muda, o valor é sobrescrito (sem histórico).

---

## Job de Orquestração

Os notebooks são executados automaticamente em sequência por um **Job no Databricks**:

```
001-preparando-ambiente
        ↓
002-extracao
        ↓
003-bronze
        ↓
004-silver
        ↓
005-gold
```

Tempo total de execução: ~3 minutos.
