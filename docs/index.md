# Lakehouse com Databricks — Locadora de Veículos

Bem-vindo à documentação do **Trabalho 3 de Engenharia de Dados**.

Este projeto implementa uma pipeline completa de dados utilizando a **Arquitetura Medalhão** no Databricks, com dados de uma locadora de veículos.

---

## O que foi construído?

Uma pipeline que transforma dados brutos de um banco de dados PostgreSQL em um modelo dimensional analítico, passando por 4 camadas:

| Camada | O que contém |
|---|---|
| **Landing** | Arquivos CSV extraídos diretamente do banco de dados |
| **Bronze** | Dados em formato Delta Lake com metadados de ingestão |
| **Silver** | Dados com qualidade aplicada e colunas padronizadas |
| **Gold** | Modelo dimensional (Star Schema) pronto para análise |

---

## Tecnologias

- **Databricks Free Edition** — plataforma cloud de dados
- **Apache Spark (PySpark)** — processamento distribuído
- **Delta Lake** — formato de armazenamento avançado
- **PostgreSQL / Supabase** — banco de dados de origem
- **Jobs & Pipelines** — orquestração da pipeline

---

## Navegação

Use o menu acima para explorar cada parte do projeto:

- **Contexto do Projeto** — domínio de negócio e fonte de dados
- **Arquitetura Medalhão** — explicação das camadas Landing, Bronze, Silver e Gold
- **Spark e Delta Lake** — conceitos técnicos fundamentais
- **Databricks** — plataforma utilizada e recursos explorados
- **Modelagem Dimensional** — Star Schema e Ralph Kimball
