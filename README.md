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

## ⚙️ Tecnologias e Versões

| Tecnologia | Versão | Função |
|---|---|---|
| **Databricks Free Edition** | Runtime 15.4 LTS | Plataforma de execução e orquestração |
| **Apache Spark (PySpark)** | 3.5.0 | Engine de processamento distribuído |
| **Delta Lake** | 3.2.0 | Formato de armazenamento ACID |
| **Python** | 3.11 | Linguagem dos notebooks |
| **PostgreSQL** | 15 | Banco de dados de origem |
| **Supabase** | - | Hospedagem cloud do PostgreSQL |
| **JDBC Driver** | postgresql-42.x | Conexão Spark → PostgreSQL |
| **MkDocs Material** | 9.x | Geração do site de documentação |

---

## 🚀 Como Reproduzir o Ambiente

### Pré-requisitos

- Conta gratuita no [Databricks Free Edition](https://community.cloud.databricks.com)
- Conta gratuita no [Supabase](https://supabase.com)
- [Git](https://git-scm.com/) instalado na máquina
- [Python 3.11+](https://www.python.org/) instalado (para o MkDocs)

---

### Passo 1 — Clonar o repositório

```bash
git clone https://github.com/isabelamadeirajose/lakehouse-databricks-locadora.git
cd lakehouse-databricks-locadora
```

---

### Passo 2 — Configurar o banco de dados no Supabase

1. Acesse [supabase.com](https://supabase.com) e crie um novo projeto
2. No **SQL Editor**, crie as tabelas da locadora de veículos:

```sql
CREATE TABLE categoria (id_categoria SERIAL PRIMARY KEY, nome_categoria VARCHAR(50));
CREATE TABLE marca (id_marca SERIAL PRIMARY KEY, nome_marca VARCHAR(50));
CREATE TABLE modelo (id_modelo SERIAL PRIMARY KEY, nome_modelo VARCHAR(100), id_marca INT, id_categoria INT);
CREATE TABLE estado (id_estado SERIAL PRIMARY KEY, nome_estado VARCHAR(50), sigla CHAR(2));
CREATE TABLE cidade (id_cidade SERIAL PRIMARY KEY, nome_cidade VARCHAR(100), id_estado INT);
CREATE TABLE agencia (id_agencia SERIAL PRIMARY KEY, nome_agencia VARCHAR(100), id_cidade INT);
CREATE TABLE cliente (id_cliente SERIAL PRIMARY KEY, nome_cliente VARCHAR(100), cpf VARCHAR(14), id_cidade INT);
CREATE TABLE carro (id_carro SERIAL PRIMARY KEY, placa VARCHAR(10), ano INT, cor VARCHAR(30), id_modelo INT, id_agencia INT);
CREATE TABLE reserva (id_reserva SERIAL PRIMARY KEY, id_cliente INT, id_carro INT, id_agencia INT, data_retirada DATE, data_devolucao DATE, valor_total DECIMAL(10,2), status_reserva VARCHAR(20));
CREATE TABLE pagamento (id_pagamento SERIAL PRIMARY KEY, id_reserva INT, forma_pagamento VARCHAR(30), valor_pago DECIMAL(10,2), data_pagamento DATE);
```

3. Em **Project Settings → Database**, copie a string de conexão via **Session Pooler** (IPv4 compatível):

```
jdbc:postgresql://<host-session-pooler>:5432/postgres?user=postgres.<project-ref>&password=[YOUR-PASSWORD]
```

> ⚠️ **Importante:** Use o **Session Pooler** (não o Direct Connection) para compatibilidade com IPv4 do Databricks.

---

### Passo 3 — Configurar o Databricks

1. Acesse [community.cloud.databricks.com](https://community.cloud.databricks.com)
2. Crie um cluster com as configurações:
   - **Runtime:** 15.4 LTS (Spark 3.5.0, Scala 2.12)
   - **Node type:** Single node
3. No **Workspace**, crie uma pasta chamada `lakehouse-locadora`
4. Importe os notebooks da pasta `notebooks/` deste repositório:
   - Clique em **Import** → selecione cada arquivo `.ipynb`

---

### Passo 4 — Configurar a conexão JDBC

No notebook `002-extracao`, localize e substitua a URL de conexão:

```python
jdbc_url = "jdbc:postgresql://<seu-host>/postgres?user=<seu-user>&password=[YOUR-PASSWORD]"
```

> ⚠️ **Nunca suba sua senha real para o GitHub!**

---

### Passo 5 — Criar e executar o Job

1. No Databricks, acesse **Workflows → Jobs → Create Job**
2. Nomeie o job como `lakehouse-locadora`
3. Adicione as tasks na seguinte ordem:

```
Task 1: 001-preparando-ambiente
        ↓
Task 2: 002-extracao
        ↓
Task 3: 003-bronze
        ↓
Task 4: 004-silver
        ↓
Task 5: 005-gold
```

4. Clique em **Run now** e aguarde a conclusão (~3 minutos)

---

### Passo 6 — Publicar a documentação (opcional)

Instale o MkDocs e publique no GitHub Pages:

```bash
pip install mkdocs-material
mkdocs gh-deploy
```

O site ficará disponível em:  
`https://<seu-usuario>.github.io/lakehouse-databricks-locadora`

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

A documentação completa do projeto está disponível em:  
🔗 **[https://isabelamadeirajose.github.io/lakehouse-databricks-locadora](https://isabelamadeirajose.github.io/lakehouse-databricks-locadora)**

---

## 👩‍💻 Autora

**Isabela Madeira Jose**  
Curso de Engenharia de Dados
