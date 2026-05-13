# Modelagem Dimensional

## Ralph Kimball e o Star Schema

**Ralph Kimball** é o criador da metodologia de **Modelagem Dimensional**, amplamente usada em Data Warehouses e camadas analíticas (Gold).

A ideia central é organizar os dados em dois tipos de tabelas:

| Tipo | Nome | O que contém |
|---|---|---|
| **Fato** | `fato_*` | Os eventos do negócio (o que aconteceu), com medidas numéricas |
| **Dimensão** | `dim_*` | O contexto dos eventos (quem, onde, quando, o quê) |

O resultado é um **Star Schema** (Esquema Estrela) — a tabela fato no centro, rodeada pelas dimensões:

```
        dim_tempo
            │
dim_localidade ──── fato_reserva ──── dim_cliente
            │
        dim_carro
```

---

## Tabelas Gold do Projeto

### fato_reserva

Tabela central. Registra cada **reserva** realizada, com suas medidas e chaves para as dimensões.

| Coluna | Tipo | Descrição |
|---|---|---|
| `id_reserva` | INT | Chave primária |
| `id_cliente` | INT | FK → dim_cliente |
| `id_carro` | INT | FK → dim_carro |
| `id_localidade` | INT | FK → dim_localidade |
| `id_tempo_retirada` | INT | FK → dim_tempo (data retirada) |
| `id_tempo_devolucao` | INT | FK → dim_tempo (data devolução) |
| `valor_total` | DECIMAL | Valor total da reserva |
| `dias_reserva` | INT | Quantidade de dias |
| `valor_diaria` | DECIMAL | Valor por dia |
| `forma_pagamento` | STRING | Como foi pago |
| `status_reserva` | STRING | Status (confirmada, cancelada...) |

---

### dim_cliente

Quem realizou a reserva.

| Coluna | Tipo | Descrição |
|---|---|---|
| `id_cliente` | INT | Chave primária |
| `nome_cliente` | STRING | Nome completo |
| `cpf` | STRING | CPF do cliente |
| `cidade_cliente` | STRING | Cidade onde mora |
| `estado_cliente` | STRING | Estado onde mora |

---

### dim_carro

Qual veículo foi reservado.

| Coluna | Tipo | Descrição |
|---|---|---|
| `id_carro` | INT | Chave primária |
| `placa` | STRING | Placa do veículo |
| `marca` | STRING | Marca (Toyota, Honda...) |
| `modelo` | STRING | Modelo (Corolla, Civic...) |
| `categoria` | STRING | Categoria (Econômico, SUV...) |
| `ano` | INT | Ano de fabricação |
| `cor` | STRING | Cor do veículo |

---

### dim_localidade

Onde a reserva foi feita (agência).

| Coluna | Tipo | Descrição |
|---|---|---|
| `id_localidade` | INT | Chave primária |
| `nome_agencia` | STRING | Nome da agência |
| `cidade_agencia` | STRING | Cidade da agência |
| `estado_agencia` | STRING | Estado da agência |

---

### dim_tempo

Calendário para análise temporal (2023–2026).

| Coluna | Tipo | Descrição |
|---|---|---|
| `id_tempo` | INT | Chave primária (formato YYYYMMDD) |
| `data_completa` | DATE | Data completa |
| `dia` | INT | Dia do mês |
| `mes` | INT | Mês |
| `nome_mes` | STRING | Nome do mês (Janeiro, Fevereiro...) |
| `trimestre` | INT | Trimestre (1, 2, 3, 4) |
| `ano` | INT | Ano |
| `dia_semana` | STRING | Nome do dia (Segunda, Terça...) |
| `fim_de_semana` | BOOLEAN | True se sábado ou domingo |

---

## SCD — Slowly Changing Dimensions

**SCD** (Dimensões de Mudança Lenta) define como tratar registros de dimensões quando seus dados mudam.

### Tipo 1 — Sobrescrever (usado neste projeto)

Quando um registro muda, o valor antigo é **substituído** pelo novo. Não há histórico.

**Exemplo:** Cliente muda de cidade → a coluna `cidade_cliente` é atualizada.

Implementado via `MERGE INTO`:

```sql
MERGE INTO gold.dim_cliente AS destino
USING (SELECT ...) AS origem
ON destino.id_cliente = origem.id_cliente
WHEN MATCHED THEN
  UPDATE SET
    destino.nome_cliente = origem.nome_cliente,
    destino.cidade_cliente = origem.cidade_cliente
WHEN NOT MATCHED THEN
  INSERT (id_cliente, nome_cliente, cidade_cliente, ...)
  VALUES (origem.id_cliente, origem.nome_cliente, origem.cidade_cliente, ...)
```

### Comparativo de Tipos

| Tipo | Estratégia | Mantém histórico? |
|---|---|---|
| **SCD 1** | Sobrescreve | ❌ Não |
| **SCD 2** | Adiciona nova linha com flag de ativo/inativo | ✅ Sim |
| **SCD 3** | Adiciona coluna "valor anterior" | Parcial |

---

## Por que Star Schema?

- **Consultas simples:** Uma tabela fato + JOINs com dimensões
- **Desempenho:** Menos JOINs que um modelo normalizado
- **Legibilidade:** Qualquer analista entende a estrutura
- **Ferramenta padrão:** Suportado por Power BI, Tableau, Looker e outros
