# 📘 Overview-Apache-Kafka

## 🧭 Conceitos Fundamentais

### 🟢 Produtor e Consumidor
- Kafka atua como **intermediário** entre **Producer** (produtor) e **Consumer** (consumidor).

` producer --> Kafka <-- consumer `

- Kafka **não envia** nem **distribui** mensagens. Ele apenas **armazena**.
- O **Consumer** é quem vai até o Kafka e **lê** a mensagem.
- Kafka funciona em um **cluster** (conjunto de máquinas), formado por **nós** chamados de **brokers**.

> 🔸 Recomendação mínima: **3 brokers**

## 🛠️ Zookeeper
- Necessário para o **gerenciamento e monitoramento** dos nós do Kafka.
- Garante a **orquestração e liderança** dos brokers.

## 🧩 Tópicos
- Canal de comunicação usado para **receber e disponibilizar dados**.
- Funciona como um **log particionado e persistente**.

`producer --(tópico)--> Kafka <--(tópico)-- consumer`

### Estrutura do Registro (Mensagem)
- Cada mensagem tem:
  - **Offset** (ID incremental por partição)
  - **Headers**
  - **Key**
  - **Value**
  - **Timestamp**

## 🧱 Partições
- Cada **tópico** pode ter uma ou mais **partições**, garantindo:
  - **Distribuição de carga**
  - **Resiliência**

```
Tópico
├── Partição 1 --> mensagem A (offset 0)
├── Partição 2 --> mensagem B (offset 0)
└── Partição 3 --> mensagem C (offset 0)
```
## 🗝️ Uso de Keys
- A **Key** define **em qual partição** a mensagem será enviada.
- Útil para garantir **ordem**:
  - Mensagens com a mesma Key vão sempre para a **mesma partição**.

### Exemplo prático:
- 💳 Transferência bancária (offset 0 da partição 1)
- 🧾 Estorno (offset 0 da partição 2)

> Problema: estorno pode ser processado **antes** da transferência.

🔑 **Solução**: Usar a **mesma Key** para garantir a **ordem** entre transferência e estorno.

## 🛡️ Replicação e Resiliência
- **Replication Factor** define o número de **cópias** de cada partição.
  - Exemplo: fator 2 ⇒ cada partição terá **2 réplicas**.
- Garante alta **disponibilidade** em caso de falha de um broker.

### 🏷️ Liderança de Partição
- Cada partição tem um **líder** (em um broker), responsável por leitura e escrita.
- Se o líder falhar, o Kafka elege um **novo líder** automaticamente.

## 📦 Garantia de Entrega (Producer)

### Tipos de `acks`:

| Tipo de ack      | Comportamento             | Garantia        | Desempenho |
|------------------|---------------------------|------------------|-------------|
| `acks=0`         | Não aguarda resposta      | Nenhuma         | 🚀 Alta     |
| `acks=1`         | Aguarda só do líder       | Parcial         | ⚡ Média    |
| `acks=all / -1`  | Aguarda líder e réplicas  | Total           | 🐢 Baixa    |

## 📊 Modelos de Entrega

| Modelo             | Característica               | Risco                    |
|--------------------|------------------------------|--------------------------|
| **At most once**   | Melhor performance           | Pode perder mensagens    |
| **At least once**  | Entrega garantida            | Pode duplicar mensagens  |
| **Exactly once**   | Entrega única garantida      | Performance mais lenta   |

## 🌀 Idempotência
- Garante que **mensagens duplicadas** não sejam processadas mais de uma vez.
- Kafka detecta e **descarta duplicatas**.
- Útil em falhas de conexão e reenvios automáticos pelo producer.


## 👥 Consumidores e Partições

- Podemos ter cenários como:
`1 Consumer --> 3 Partições`


- Também é possível usar **Consumer Groups** (grupos de consumidores):
  - Cada **grupo** pode ser, por exemplo, uma **instância do mesmo sistema** rodando em diferentes máquinas.
  - Exemplo:
```
Grupo X:
├── Consumer A → lê Partições 0 e 1
└── Consumer B → lê Partição 2
```


> ⚠️ **Regra importante**: Dois **consumidores do mesmo grupo** **não podem** ler a **mesma partição** ao mesmo tempo.

- Um **Consumer fora de grupo** (isolado) irá **ler todas as partições**, independentemente de divisão de carga.
