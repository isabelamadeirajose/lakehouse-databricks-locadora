# 🏗️ Lakehouse com Databricks — Locadora de Veículos

> **Trabalho 3 — Engenharia de Dados**  
> Implementação de uma pipeline completa com Arquitetura Medalhão (Landing → Bronze → Silver → Gold) utilizando Databricks, Apache Spark, Delta Lake e modelagem dimensional de Ralph Kimball.

---

## 📋 Sobre o Projeto

Este projeto implementa um **Lakehouse** para uma locadora de veículos, onde os dados são extraídos de um banco de dados PostgreSQL (Supabase), processados em camadas progressivas de qualidade e entregues em um modelo dimensional para análise.

### Fonte de Dados
- **Banco de dados:** PostgreSQL hospedado no [Supabase](https://supabase.com)
- **Domínio:** Sistema de locação de veículos
- **Tabelas:** `categoria`, `marca`, `modelo`, `estado`, `cidade`, `agencia`, `cliente`, `carro`, `reserva`, `pagamento`

---

## 🏛️ Arquitetura Medalhão

```
PostgreSQL (Supabase)
        │
        ▼
┌─────────────┐
│   LANDING   │  Arquivos CSV brutos extraídos via JDBC
│  (Volume)   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   BRONZE    │  Delta Lake com metadados (data_hora, nome_arquivo)
│  (Delta)    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   SILVER    │  Data Quality: colunas padronizadas (uppercase, abreviações expandidas)
│  (Delta)    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│    GOLD     │  Modelagem Dimensional (Ralph Kimball) — Star Schema
│  (Delta)    │  dim_cliente, dim_carro, dim_localidade, dim_tempo, fato_reserva
└─────────────┘
```

---

## 📓 Notebooks

| Notebook | Descrição |
|---|---|
| `001-preparando-ambiente` | Cria schemas (landing, bronze, silver, gold) e volume de armazenamento |
| `002-extracao` | Extrai 10 tabelas do PostgreSQL via JDBC e salva como CSV no Landing |
| `003-bronze` | Lê CSVs do Landing, adiciona metadados e salva como Delta Lake |
| `004-silver` | Aplica Data Quality (padronização de colunas) e salva na camada Silver |
| `005-gold` | Cria modelo dimensional com fatos e dimensões (SCD Tipo 1 via MERGE INTO) |
| `006-destruindo-ambiente` | Utilitário para limpar todos os schemas e dados (não incluído no Job) |

---

## ⚙️ Tecnologias Utilizadas

| Tecnologia | Função |
|---|---|
| **Databricks (Free Edition)** | Plataforma de execução dos notebooks e orquestração via Jobs |
| **Apache Spark (PySpark)** | Engine de processamento distribuído |
| **Delta Lake** | Formato de armazenamento com suporte a ACID, histórico e MERGE |
| **PostgreSQL / Supabase** | Banco de dados relacional de origem |
| **JDBC** | Protocolo de conexão Databricks → PostgreSQL |
| **Unity Catalog** | Gerenciamento de schemas e volumes no Databricks |

---

## 🚀 Como Reproduzir

### Pré-requisitos
- Conta no [Databricks Free Edition](https://community.cloud.databricks.com)
- Conta no [Supabase](https://supabase.com) com banco PostgreSQL configurado
- Tabelas da locadora criadas no Supabase

### Passo a Passo

**1. Configure o banco de dados no Supabase**  
Crie as tabelas da locadora de veículos conforme o schema do projeto.

**2. Faça upload dos notebooks no Databricks**  
- Acesse seu workspace no Databricks
- Crie uma pasta chamada `lakehouse-locadora`
- Importe os notebooks da pasta `notebooks/` deste repositório

**3. Configure a conexão JDBC**  
No notebook `002-extracao`, substitua `[YOUR-PASSWORD]` pela senha do seu banco Supabase:

```python
jdbc_url = "jdbc:postgresql://<seu-host>/postgres?user=<seu-user>&password=[YOUR-PASSWORD]"
```

> ⚠️ **Nunca suba sua senha real para o GitHub!**

**4. Crie e execute o Job no Databricks**  
- Vá em **Workflows > Jobs > Create Job**
- Adicione as tasks na ordem: `001 → 002 → 003 → 004 → 005`
- Execute o Job e aguarde a conclusão

---

## 📁 Estrutura do Repositório

```
lakehouse-databricks-locadora/
├── notebooks/
│   ├── 001-preparando-ambiente.ipynb
│   ├── 002-extracao.ipynb
│   ├── 003-bronze.ipynb
│   ├── 004-silver.ipynb
│   ├── 005-gold.ipynb
│   └── 006-destruindo-ambiente.ipynb
├── docs/
│   ├── index.md
│   ├── contexto.md
│   ├── arquitetura-medalhao.md
│   ├── spark-e-delta-lake.md
│   ├── databricks.md
│   └── modelagem-dimensional.md
├── .gitignore
├── mkdocs.yml
└── README.md
```

---

## 📚 Documentação

A documentação completa do projeto está disponível em: *(link do MkDocs será adicionado)*

---

## 👩‍💻 Autora

**Isabela Madeira Jose**  
Curso de Engenharia de Dados
