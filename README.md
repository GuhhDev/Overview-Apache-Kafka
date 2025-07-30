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

---

## 🐳 Subindo o Apache Kafka com Docker Compose

Agora que entendemos os principais conceitos do Kafka, podemos subir um ambiente de testes com `docker-compose`.

### ✅ Pré-requisitos

- Docker instalado ([instalar](https://docs.docker.com/get-docker/))
- Docker Compose instalado (caso necessário, use `docker compose` com Docker Desktop)

### 📁 Estrutura do Projeto

Crie um arquivo chamado `docker-compose.yml` com o seguinte conteúdo:

```yaml
version: "3"

services:  

  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
    extra_hosts:
      - "host.docker.internal:172.17.0.1"

  kafka:
    image: confluentinc/cp-kafka:6.0.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
      - "9094:9094"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
      KAFKA_LISTENERS: INTERNAL://:9092,OUTSIDE://:9094
      KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka:9092,OUTSIDE://host.docker.internal:9094
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,OUTSIDE:PLAINTEXT
    extra_hosts:
      - "host.docker.internal:172.17.0.1"

  control-center:
    image: confluentinc/cp-enterprise-control-center:6.0.0
    hostname: control-center
    depends_on:
      - kafka
    ports:
      - "9021:9021"
    environment:
      CONTROL_CENTER_BOOTSTRAP_SERVERS: 'kafka:9092'
      CONTROL_CENTER_REPLICATION_FACTOR: 1
      CONTROL_CENTER_CONNECT_CLUSTER: http://kafka-connect:8083
      PORT: 9021
    extra_hosts:
      - "host.docker.internal:172.17.0.1"
```

## 🛠️ Acessando e Utilizando o Kafka no Docker

Com nosso ambiente Docker instanciado, podemos acessar o terminal do container Kafka para realizar testes e comandos administrativos.

### ▶️ Acessar o container

```bash
docker exec -it kafka bash
```
## 📄Comandos com kafka-topics
- Utilizado para criar, listar e descrever tópicos no Kafka.

###🔧 Criar tópico
` kafka-topics --create --topic teste --bootstrap-server localhost:9092 --partitions 3`

###📜 Listar tópicos
`kafka-topics --list --bootstrap-server=localhost:9092`

🔍 Descrever tópico
`kafka-topics --describe --topic=teste --bootstrap-server=localhost:9092`

Exemplo de saída:
```
Topic: teste  PartitionCount: 3  ReplicationFactor: 1  Configs:
  Topic: teste  Partition: 0  Leader: 1  Replicas: 1  Isr: 1
  Topic: teste  Partition: 1  Leader: 1  Replicas: 1  Isr: 1
  Topic: teste  Partition: 2  Leader: 1  Replicas: 1  Isr: 1
```

✉️ Produzindo e Consumindo Mensagens
📤 Enviar mensagens (Producer)
`kafka-console-producer --topic=teste --bootstrap-server=localhost:9092`

📥 Consumir mensagens (Consumer)
`kafka-console-consumer --topic=teste --bootstrap-server=localhost:9092`

🕓 Ler mensagens desde o início
`kafka-console-consumer --topic=teste --from-beginning --bootstrap-server=localhost:9092`

🔸 Com --from-beginning, o consumidor lerá todas as mensagens do tópico, inclusive as anteriores à conexão.
> ⚠️ As mensagens podem vir fora de ordem se não houver key.

👥 Usando --group com Consumers
Para consumir de forma organizada e paralela em múltiplas instâncias, usamos consumer groups:

`kafka-console-consumer --topic=teste --bootstrap-server=localhost:9092 --group=x`

📊 Monitorar grupos de consumidores

`kafka-consumer-groups --bootstrap-server=localhost:9092 --group=x --describe`

Exemplo de saída:

```
GROUP  TOPIC  PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG  CONSUMER-ID      HOST          CLIENT-ID
x      teste  0          9               9               0    consumer-x-1...  192.168...    consumer-x-1
```

🌐 Acesso Visual com Confluent Control Center
Se estiver utilizando a imagem da Confluent, acesse via navegador:

`http://localhost:9021`

Painel visual com overview do cluster
Tópicos, consumidores, produção, métricas, etc.
