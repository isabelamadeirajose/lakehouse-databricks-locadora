# Contexto do Projeto

## O Negócio: Locadora de Veículos

O projeto utiliza dados de uma **locadora de veículos** — empresa que aluga carros para clientes por períodos determinados, cobrando por diárias ou km rodado.

Este domínio foi escolhido por ser familiar e possuir relacionamentos naturais entre entidades, facilitando a construção de uma modelagem dimensional rica.

---

## Fonte de Dados

Os dados estão armazenados em um banco **PostgreSQL** hospedado no [Supabase](https://supabase.com), uma plataforma de banco de dados em nuvem.

### Por que Supabase?

- Gratuito e acessível via internet (sem necessidade de containers locais)
- Compatível com conexão JDBC a partir do Databricks
- Suporta IPv4 via Session Pooler (necessário para a Free Edition do Databricks)

---

## Tabelas do Banco de Dados

O banco possui **10 tabelas** organizadas em entidades de negócio:

### Entidades de Produto/Catálogo
| Tabela | Descrição |
|---|---|
| `categoria` | Categorias de veículos (Econômico, SUV, Luxo...) |
| `marca` | Marcas dos veículos (Toyota, Honda, BMW...) |
| `modelo` | Modelos de veículos vinculados a marca e categoria |

### Entidades Geográficas
| Tabela | Descrição |
|---|---|
| `estado` | Estados brasileiros |
| `cidade` | Cidades vinculadas aos estados |
| `agencia` | Agências da locadora em diferentes cidades |

### Entidades de Cliente e Veículo
| Tabela | Descrição |
|---|---|
| `cliente` | Dados dos clientes (nome, CPF, cidade) |
| `carro` | Frota de veículos disponíveis para aluguel |

### Entidades Transacionais
| Tabela | Descrição |
|---|---|
| `reserva` | Registro de cada locação realizada |
| `pagamento` | Formas e valores de pagamento por reserva |

---

## Conexão com o Databricks

A extração dos dados é feita via **JDBC** (Java Database Connectivity), um protocolo padrão para conexão entre sistemas Java (como o Spark) e bancos de dados relacionais.

```python
jdbc_url = "jdbc:postgresql://<host>/postgres?user=<usuario>&password=<senha>"

df = spark.read \
    .format("jdbc") \
    .option("url", jdbc_url) \
    .option("dbtable", "reserva") \
    .option("driver", "org.postgresql.Driver") \
    .load()
```

O Spark lê a tabela inteira do PostgreSQL e a transforma em um **DataFrame** — estrutura de dados distribuída que pode ser processada em memória.
