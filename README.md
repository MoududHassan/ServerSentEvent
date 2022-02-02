Quarkus RabbitMQ Server side event R&D
============================

## Start the application in dev mode

In a first terminal, run:

```bash
> mvn -f rabbitmq-quickstart-producer quarkus:dev
```

In a second terminal, run:

```bash
> mvn -f rabbitmq-quickstart-processor quarkus:dev
```  

Then, open your browser to `http://localhost:8080/quotes.html`, and click on the "Request Quote" button.

## Build the application in JVM mode

To build the applications, run:

```bash
> mvn -f rabbitmq-quickstart-producer package
> mvn -f rabbitmq-quickstart-processor package
```

Because we are running in _prod_ mode, we need to provide an RabbitMQ broker.
The [docker-compose.yml](docker-compose.yml) file starts the broker and your application.

Start the broker and the applications using:

```bash
> docker compose up --build
```

Then, open your browser to `http://localhost:8080/quotes.html`, and click on the "Request Quote" button.
 