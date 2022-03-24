# Snippets

## MongoDB

```yaml
version: '2'

services:
  mongo:
    image: mongo:latest
    hostname: mongo
    container_name: mongo
    command: --auth
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: "${MONGO_USER}"
      MONGO_INITDB_ROOT_PASSWORD: "${MONGO_PASS}"
    ports:
      - 27017:27017
```

## MongoDB ReplicaSet

```yaml
# username for DB is 'root'
version: '2'

services:
  mongodb-primary:
    image: 'bitnami/mongodb:latest'
    environment:
      - MONGODB_ADVERTISED_HOSTNAME=mongodb-primary
      - MONGODB_REPLICA_SET_MODE=primary
      - MONGODB_ROOT_PASSWORD=toor
      - MONGODB_REPLICA_SET_KEY=replicasetkey
    volumes:
      - mongodb_master_data:/bitnami
    ports:
      - 27017:27017

  mongodb-secondary:
    image: 'bitnami/mongodb:latest'
    depends_on:
      - mongodb-primary
    environment:
      - MONGODB_ADVERTISED_HOSTNAME=mongodb-secondary
      - MONGODB_REPLICA_SET_MODE=secondary
      - MONGODB_INITIAL_PRIMARY_HOST=mongodb-primary
      - MONGODB_INITIAL_PRIMARY_PORT_NUMBER=27017
      - MONGODB_INITIAL_PRIMARY_ROOT_PASSWORD=toor
      - MONGODB_REPLICA_SET_KEY=replicasetkey

  mongodb-arbiter:
    image: 'bitnami/mongodb:latest'
    depends_on:
      - mongodb-primary
    environment:
      - MONGODB_ADVERTISED_HOSTNAME=mongodb-arbiter
      - MONGODB_REPLICA_SET_MODE=arbiter
      - MONGODB_INITIAL_PRIMARY_HOST=mongodb-primary
      - MONGODB_INITIAL_PRIMARY_PORT_NUMBER=27017
      - MONGODB_INITIAL_PRIMARY_ROOT_PASSWORD=toor
      - MONGODB_REPLICA_SET_KEY=replicasetkey

volumes:
  mongodb_master_data:
    driver: local
```

## MySQL

```yaml
# username for DB is 'root'
version: '2'

services:
  mysql:
    image: mysql:latest
    restart: on-failure
    environment:
      MYSQL_DATABASE: "db"
      MYSQL_ROOT_PASSWORD: "toor"
    ports:
      - "3306:3306"
    volumes:
      - ./db:/var/lib/mysql
```

## Kafka & Zookeeper

```yaml
version: '2.1'

services:
  zoo:
    image: confluentinc/cp-zookeeper:latest
    hostname: zoo
    container_name: zoo
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_SERVER_ID: 1
      ZOOKEEPER_SERVERS: zoo:2888:3888

  kafka:
    image: confluentinc/cp-kafka:latest
    hostname: kafka
    container_name: kafka
    ports:
      - "9092:9092"
      - "9999:9999"
    environment:
      KAFKA_ADVERTISED_LISTENERS: LISTENER_DOCKER_INTERNAL://kafka:19092,LISTENER_DOCKER_EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: LISTENER_DOCKER_INTERNAL:PLAINTEXT,LISTENER_DOCKER_EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: LISTENER_DOCKER_INTERNAL
      KAFKA_ZOOKEEPER_CONNECT: "zoo:2181"
      KAFKA_BROKER_ID: 1
      KAFKA_LOG4J_LOGGERS: "kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO"
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_JMX_PORT: 9999
      KAFKA_JMX_HOSTNAME: ${DOCKER_HOST_IP:-127.0.0.1}
      KAFKA_AUTHORIZER_CLASS_NAME: kafka.security.authorizer.AclAuthorizer
      KAFKA_ALLOW_EVERYONE_IF_NO_ACL_FOUND: "true"
    depends_on:
      - zoo
```

## PostgreSQL

```yaml
version: '3'

services:
  postgres:
    container_name: postgres
    image: postgres:9.3.10
    environment:
      POSTGRES_DB: ${PG_DB_NAME}
      POSTGRES_USER: ${PG_DB_USER}
      POSTGRES_PASSWORD: ${PG_DB_PASSWORD}
    ports:
      - 5432:5432
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
    driver: local
```