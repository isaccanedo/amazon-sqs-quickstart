# Demonstração do Quarkus: Cliente SQS

Este exemplo mostra como usar o cliente AWS SQS com o Quarkus. Como pré-requisito, instale a [interface de linha de comando da AWS](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html).

# Instância local do AWS SQS

Basta executá-lo da seguinte maneira para iniciar o SQS localmente:
`docker run --rm --name local-sqs -p 8010:4566 -e SERVICES=sqs -e START_WEB=0 -d localstack/localstack:1.4.0`
O SQS escuta em `localhost:8010` para endpoints REST.

Create an AWS profile for your local instance using AWS CLI:

```
$ aws configure --profile localstack
AWS Access Key ID [None]: test-key
AWS Secret Access Key [None]: test-secret
Default region name [None]: us-east-1
Default output format [None]:
```

## Create SQS queue

Crie uma fila SQS e armazene o URL da fila na variável de ambiente, pois precisaremos fornecê-la ao nosso aplicativo
```
$> QUEUE_URL=`aws sqs create-queue --queue-name=ColliderQueue --profile localstack --endpoint-url=http://localhost:8010`
```

# Run the demo on dev mode

- Run `./mvnw clean package` and then `java -Dqueue.url=$QUEUE_URL -jar ./target/quarkus-app/quarkus-run.jar`
- In dev mode `./mvnw clean quarkus:dev -Dqueue.url=$QUEUE_URL`

## Send messages to the queue
Shoot with a couple of quarks
```
curl -XPOST -H"Content-type: application/json" http://localhost:8080/sync/cannon/shoot -d'{"flavor": "Charm", "spin": "1/2"}'
curl -XPOST -H"Content-type: application/json" http://localhost:8080/sync/cannon/shoot -d'{"flavor": "Strange", "spin": "1/2"}'
```
And receive it from the queue
```
curl http://localhost:8080/sync/cannon/shoot
```

Repeat the same using async endpoints
```
curl -XPOST -H"Content-type: application/json" http://localhost:8080/async/cannon/shoot -d'{"flavor": "Charm", "spin": "1/2"}'
curl -XPOST -H"Content-type: application/json" http://localhost:8080/async/cannon/shoot -d'{"flavor": "Strange", "spin": "1/2"}'
```
And receive it from the queue
```
curl http://localhost:8080/async/cannon/shoot
```

# Running in native

You can compile the application into a native binary using:

`./mvnw clean install -Pnative`

and run with:

`./target/amazon-sqs-quickstart-1.0.0-SNAPSHOT-runner` 


# Running native in container

Build a native image in container by running:
`./mvnw package -Pnative -Dnative-image.docker-build=true`

Build a docker image:
`docker build -f src/main/docker/Dockerfile.native -t quarkus/amazon-sqs-quickstart .`

Create a network that connect your container with localstack
`docker network create localstack`

Stop your localstack container you started at the beginning
`docker stop local-sqs`

Start localstack and connect to the network
`docker run --rm --network=localstack --name localstack -p 8010:4566 -e SERVICES=sqs -e START_WEB=0 -d localstack/localstack:1.4.0`

Create queue
```
$> QUEUE_URL=`aws sqs create-queue --queue-name=ColliderQueue --profile localstack --endpoint-url=http://localhost:8010`
```
Run quickstart container connected to that network (note that we're using internal port of the localstack)
`docker run -i --rm --network=localstack -p 8080:8080 quarkus/amazon-sqs-quickstart -Dquarkus.sqs.endpoint-override=http://localstack:4566`

Send messsage
```
curl -XPOST -H"Content-type: application/json" http://localhost:8080/sync/cannon/shoot -d'{"flavor": "Charm", "spin": "1/2"}'
```

Receive message
```
curl http://localhost:8080/sync/cannon/shoot
```
