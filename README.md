Quarkus RabbitMQ Server side event R&D
============================

##Architecture

[https://i.ibb.co/stnQXBP/amqp-qs-architecture.png](https://i.ibb.co/stnQXBP/amqp-qs-architecture.png)

> In this guide, we are going to develop two applications communicating with a RabbitMQ broker. The first application sends a quote request to the RabbitMQ quote requests exchange and consumes messages from the quote queue. The second application receives the quote request and sends a quote back.


> The first application, the producer, will let the user request some quotes over an HTTP endpoint. For each quote request, a random identifier is generated and returned to the user, to put the quote request on pending. At the same time the generated request id is sent to the quote-requests exchange.

>The second application, the processor, in turn, will read from the quote-requests queue put a random price to the quote, and send it to an exchange named quotes.
Lastly, the producer will read the quotes and send them to the browser using server-sent events. The user will therefore see the quote price updated from pending to the received price in real-time.

## The Quote object
> The Quote class will be used in both producer and processor projects.

~~~
import io.quarkus.runtime.annotations.RegisterForReflection;

@RegisterForReflection
public class Quote {

    public String id;
    public int price;

    /**
    * Default constructor required for Jackson serializer
    */
    public Quote() { }

    public Quote(String id, int price) {
        this.id = id;
        this.price = price;
    }

    @Override
    public String toString() {
        return "Quote{" +
                "id='" + id + '\'' +
                ", price=" + price +
                '}';
    }
}
~~~

> JSON representation of Quote objects will be used in messages sent to the RabbitMQ queues and also in the server-sent events sent to browser clients.

## Sending quote request
~~~
import java.util.UUID;

import javax.ws.rs.GET;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

import org.acme.rabbitmq.model.Quote;
import org.eclipse.microprofile.reactive.messaging.Channel;
import org.eclipse.microprofile.reactive.messaging.Emitter;

import io.smallrye.mutiny.Multi;

@Path("/quotes")
public class QuotesResource {

    @Channel("quote-requests") Emitter<String> quoteRequestEmitter; 

    /**
     * Endpoint to generate a new quote request id and send it to "quote-requests" channel (which
     * maps to the "quote-requests" RabbitMQ exchange) using the emitter.
     */
    @POST
    @Path("/request")
    @Produces(MediaType.TEXT_PLAIN)
    public String createRequest() {
        UUID uuid = UUID.randomUUID();
        quoteRequestEmitter.send(uuid.toString()); 
        return uuid.toString();
    }
}
~~~

> Inject a Reactive Messaging Emitter to send messages to the quote-requests channel.
> On a post request, generate a random UUID and send it to the RabbitMQ queue using the emitter.

> This channel is mapped to a RabbitMQ exchange using the configuration we will add to the application.properties file. Open the src/main/resource/application.properties file

## Processing quote requests
~~~
import java.util.Random;

import javax.enterprise.context.ApplicationScoped;

import org.acme.rabbitmq.model.Quote;
import org.eclipse.microprofile.reactive.messaging.Incoming;
import org.eclipse.microprofile.reactive.messaging.Outgoing;

import io.smallrye.reactive.messaging.annotations.Blocking;

/**
 * A bean consuming data from the "quote-requests" RabbitMQ queue and giving out a random quote.
 * The result is pushed to the "quotes" RabbitMQ exchange.
 */
@ApplicationScoped
public class QuoteProcessor {

    private Random random = new Random();

    @Incoming("requests")       
    @Outgoing("quotes")         
    @Blocking                   
    public Quote process(String quoteRequest) throws InterruptedException {
        // simulate some hard-working task
        Thread.sleep(1000);
        return new Quote(quoteRequest, random.nextInt(100));
    }
}
~~~


> Indicates that the method consumes the items from the requests channel


> Indicates that the objects returned by the method are sent to the quotes channel


> Indicates that the processing is blocking and cannot be run on the caller thread.


> The process method is called for every RabbitMQ message from the quote-requests queue, and will send a Quote object to the quotes exchange.


> Note that in this case we have one incoming and one outgoing connector configuration, each one distinctly named. The configuration keys are structured as follows:
mp.messaging.[outgoing|incoming].{channel-name}.property=value
The channel-name segment must match the value set in the @Incoming and @Outgoing annotation:

> quote-requests → RabbitMQ queue from which we read the quote requests


> quotes → RabbitMQ exchange in which we write the quotes

## Receiving quotes
> Back to our producer project. Let’s modify the QuotesResource to consume quotes, bind it to an HTTP endpoint to send events to clients:
~~~
import io.smallrye.mutiny.Multi;
//...

@Channel("quotes") Multi<Quote> quotes;     

/**
 * Endpoint retrieving the "quotes" queue and sending the items to a server sent event.
 */
@GET
@Produces(MediaType.SERVER_SENT_EVENTS) 
public Multi<Quote> stream() {
    return quotes; 
}

~~~

> Injects the quotes channel using the @Channel qualifier


> Indicates that the content is sent using Server Sent Events


> Returns the stream (Reactive Stream)




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
 
