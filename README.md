Apache Foundation -> Apache Kafka (open source) 
* https://kafka.apache.org (core)
* confluent: flavor com dashboard, etc.. https://github.com/confluentinc/cp-all-in-one   / cp-all-in-one/cp-all-in-one/docker-compose.yml

* comunicacao assincrona = resiliencia e tolerancia a falhas
* concorrentes: RabbtQ, NATS, BigQ - Apache Pulsar
* modoinstalado localmente ou  gerenciado (provedor de cloud services faz gestao)

*Cheat sheet: https://medium.com/@TimvanBaarsen/apache-kafka-cli-commands-cheat-sheet-a6f06eac01b#8198

HTTP Syncrono
cliente -REQ-> servidor

Apache Kafka (asyncrono)
Front -> backend (registro do pedido) -> Kafka -> Paypal -> Kafka -> backend (confirma pagto)
*processo streaming de dados (eventos)


Conceito de PRODUCER x CONSUMER
Publish / Subscribe (pub/Sub)

PRODUCER -> evento -> fila FIFO do Kafka

ZOOKEPER: permite a comunicação com os brokers

REPLICA: backup em tempo real quando ativa a replica: é guardado em outro broker (n brokers = REPLICATION FACTOR)

*** Executando comandos no bash do container criado pelo docker-compose
docker-compose up
http://localhost:9021/  //interface do control center
docker-compose exec kafka bash
kafka- **TAB para ver os comandos

***Exemplo de topico sem particao (todos consumidores consomem as mesmas msgs)
kafka-topics --create --topic topico-exemplo --bootstrap-server localhost:9092   //criando topico o servidor local
kafka-topics --describe --topic topico-exemplo --bootstrap-server localhost:9092  //listando configs
        * Topic: topico-exemplo   TopicId: GY4IW4U8SvartKoNV6OlXA PartitionCount: 1       ReplicationFactor: 1    Configs:
        * Topic: topico-exemplo   Partition: 0    Leader: 1       Replicas: 1     Isr: 1
kafka-console-consumer --topic topico-exemplo --bootstrap-server localhost:9092  //consumindo o topico
kafka-console-producer --topic topico-exemplo --bootstrap-server localhost:9092 //publicando no topico
***Exemplo de topico sem particao (todos consumidores veem as mesmas msgs)
kafka-topics --delete --topic topico-exemplo --bootstrap-server localhost:9092   //deletando topico - nao pode ter consumers rodando


***Exemplo de topico com particao e grupo (os consumidores consomem FIFO, ou seja, quem pegar primeiro consome a msg e os demais nao recebem) - isso é interessante para ter serviços replicados consumindo uma mesma fila de topicos = - importante: nao adianta ter mais consumidores que particao, pois os execdentes nao receberao msgs - para dividir as msgs, tem que estar no group do topico, caso contraios ele recebera uma copia de todas msgs
kafka-topics --create --topic topico-exemplo --partitions 2 --bootstrap-server localhost:9092   //criando topico com 2 particoes 
        Topic: topico-exemplo   TopicId: o_aajmn-QcSR19x7K6S_CA PartitionCount: 2       ReplicationFactor: 1    Configs:
        Topic: topico-exemplo   Partition: 0    Leader: 1       Replicas: 1     Isr: 1
        Topic: topico-exemplo   Partition: 1    Leader: 1       Replicas: 1     Isr: 1
kafka-console-consumer --topic topico-exemplo --group group1 --bootstrap-server localhost:9092  //consumindo dentro do mesmo grupo 
kafka-console-producer --topic topico-exemplo --bootstrap-server localhost:9092
kafka-consumer-groups --describe --group group1 --bootstrap-server localhost:9092 
GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG CONSUMER-ID                                            HOST            CLIENT-ID
group1          topico-exemplo  1          6               6               0   consumer-group1-1-dcb87816-e057-4140-a69d-bc97175bb7a1 /172.19.0.3     consumer-group1-1
group1          topico-exemplo  0          4               4               0   consumer-group1-1-7184752f-1825-44cc-93dc-9fd040346073 /172.19.0.3     consumer-group1-1


Exemplo de multiplos consmidores e grupos
kafka-topics --create --topic topico-exemplo --partitions 2 --bootstrap-server localhost:9092
kafka-console-producer --topic topico-exemplo --bootstrap-server localhost:9092
kafka-console-consumer --topic topico-exemplo --group group1 --bootstrap-server localhost:9092  //consumindo dentro do grupo 1
kafka-console-consumer --topic topico-exemplo --group group1 --bootstrap-server localhost:9092  //consumindo dentro do grupo 1 
kafka-console-consumer --topic topico-exemplo --group group2 --bootstrap-server localhost:9092  //consumindo dentro do grupo 2 
kafka-console-consumer --topic topico-exemplo --group group2 --bootstrap-server localhost:9092  //consumindo dentro do grupo 2
kafka-console-consumer --topic topico-exemplo --bootstrap-server localhost:9092  //consumindo todas as msg do producer (ex. logger)


--from-beginning: le todos as msg desde o comeco //nao funciona para o grupo
--replication-factor X - cria as replicas das particoes nos brokers (obs: precisa ter o mesmo numero de brokers ativos)

kafka-topics --bootstrap-server localhost:9092 --create --topic topico-exemplo --partitions 2 --replication-factor 3


EXEMPLO de Kafka com a aplicacao nest-auth de autorizacao com JWT

cd ~/projects/live-nest-auth
docker-compose up
docker-compose exec app bash
** para usar o kafka, instalar a biblioteca 
npm install @nestjs/microservices kafkajs --save