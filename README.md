Quarkus RabbitMQ Server side event R&D
============================

##Architecture
In this guide, we are going to develop two applications communicating with a RabbitMQ broker. The first application sends a quote request to the RabbitMQ quote requests exchange and consumes messages from the quote queue. The second application receives the quote request and sends a quote back.

[](https://ibb.co/BwbjY1P)

The first application, the producer, will let the user request some quotes over an HTTP endpoint. For each quote request, a random identifier is generated and returned to the user, to put the quote request on pending. At the same time the generated request id is sent to the quote-requests exchange.



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
 
