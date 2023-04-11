# Microservices application: Customers, accounts and money transfers

This is the source code for a simple banking application.
The application architecture is based on [microservices](http://microservices.io/patterns/microservices.html), [Event Sourcing](http://microservices.io/patterns/data/event-sourcing.html) and [CQRS](http://microservices.io/patterns/data/cqrs.html).
It is built using Spring Cloud, Spring Boot and the [Eventuate platform](http://eventuate.io/).
The application is used in hands-on labs that are part of a [microservices services class](http://www.chrisrichardson.net/training.html) that is taught by [Chris Richardson](http://www.chrisrichardson.net/about.html).


### Required Architectural quality attributes
- Devops = testability, deployability, maintainability
- Autonomous teams = modularity
- long-lived = modularity, evolvability

Standard architecture is monolithic architecture:
![image](https://user-images.githubusercontent.com/27693622/231139288-68a26bc6-fe05-4ca4-ba8e-67af8518549f.png)

The monolithic pattern is not necessarily an antipattern it can fulfil:
- testability
- deployability
- maintainability
- modularity
- evolvability

The problem is that successful applications can grow. The application can grow and the rate of change increases.
All teams would contribute to the large code base. The application no longer fits in developers heads but can
become a big ball of mud. The technology stack can become obsolete but rewrites are not feasible.

![image](https://user-images.githubusercontent.com/27693622/231146431-05bb818b-b1bb-4a83-a702-e82621a22d14.png)

Chris Richardson mentions [The Art of Scalability, Martin Abbot and Michael Fisher](https://www.amazon.co.uk/Art-Scalability-Architecture-Organizations-Enterprise-ebook/dp/B00YF0OSHC/ref=sr_1_1?crid=3AQNJRQUPUNQU&keywords=art+of+scalability&qid=1681212190&sprefix=art+of+scalability%2Caps%2C94&sr=8-1)

![image](https://user-images.githubusercontent.com/27693622/231148864-01b85caf-6bdf-4b11-861e-beb5ed6521be.png)

Here we have three dimensions for the scale cube. X axis = multiple copies on load balancer. Z axis = split by categories on requests.
Y axis = functional decomposition, breaking by function.

The microservice architecture is an architectural style that structures an application as a set of services. Each microservice is:
- highly maintainable and testable
- loosely coupled
- independently deployable
- organized around business capabilities
- owned by a small team

Top tip: Start with one service per team and only split service if it solves a problem. Monzo, for instance, has ten services per developer
which is excessive.

![image](https://user-images.githubusercontent.com/27693622/231155137-52de4117-5e7d-4e7a-8077-6edab00e0c16.png)

The above diagram shows individual services and functional decomposition on the front end to query each service. The complexity can be a downside
but there are some benefits:
- maintainability: small service => easier to understand and change
- modularity: service API is impermeable => enforces modularity
- evolvability: evolve each service's technology stack independently
- testability: small service => easier and faster to test
- deployability: each service is independently deployable

This can increase scalability and fault tolerance. Interprocess communication and partial failure and distributed data can increase complexity.
There are challenges for integration testing. Each service is its own application and this can increase deployment challenges.
Identifying service boundaries is challenging - getting it wrong can lead to a distributed monolith anti-pattern. Refactoring to microservices
can take a long time. We are now going to talk about the individual services. Traditional 3 tiered architecture doesn't reflect reality:
Presentation, Business Logic and Persistence are not only single. We can use hexagonal architecture to describe the individual microservice
architectures. 

![image](https://user-images.githubusercontent.com/27693622/231159986-c74647e8-ea1c-4abd-a756-585040524490.png)

The reason for a service and team is to expose an API including: Commands that change data, Queries that extract data without modifying it. Often it is
better to use Asynchronous messaging to reduce coupling between components. It is common for services to emit events to signal that the state of the business
object has changed. The service may invoke other services and subscribe to events.

### Types of coupling
Coupling is inevitable but must be loose. There are two types of coupling:
- runtime coupling - order service cannot respond to a synchronous request until another service responds
- Design time coupling - changes to a service can involve changing the client

Design time coupling can be difficult because it forces teams to coordinate work. Cross team collaboration slows down development. Decision making within the
team is ten times faster than cross team collaboration. The service should be able to change without changing the API. We should avoid sharing database tables in
microservices.

Run time coupling can occur when two services are using the same database blocking the other service from doing its work. Ideally we should use separate databases for
each service.

### Runtime coupling
The trouble with synchronous interprocess communications is that they can lead to reduced availability. Synchronous coupling can make the services less available.
Method calls are fast but services syncrhonously calling each other can inhibit response times. 
Regarding asynchronout messaging Chris Richardson mentions [Enterprise Integration Patterns, Gregor Hohpe](https://www.amazon.co.uk/Enterprise-Integration-Patterns-Designing-Addison-Wesley-ebook/dp/B007MQLL4E/ref=tmm_kin_swatch_0?_encoding=UTF8&qid=1681216342&sr=8-1):

![image](https://user-images.githubusercontent.com/27693622/231164747-7f1e3e81-362b-432a-93bc-c41d5726a724.png)

The sender sends a message over a channel to a recipient.
- Abstraction of message broker capabilities:
  - apache kafka topics
  - JMS queues and topics
- channel types:
  - point-to-point - deliver to one recipient
  - publish-subscribe - deliver to all recipients

### Sagas
https://microservices.io/patterns/data/saga.html

![image](https://user-images.githubusercontent.com/27693622/231169706-6658a4d2-004a-4ad2-9428-23627ca54811.png)

The Order Service returns immediately and the Customer Service confirms whether if the order is possible.
There is more availability but this is an eventually consistent design. The response doesn't validate the outcome immediately.

Another option to reduce is CQRS:
https://microservices.io/patterns/data/cqrs.html

![image](https://user-images.githubusercontent.com/27693622/231172329-ed4753dd-8652-4351-8d7d-bd4335532ac5.png)

The responsibility of knowing the credit is owned by the Order Service. The response can then tell immediately whether
the request is accepted.

### Testing
The goal of Microservices is to enable Devops which requires automated testing. The complexity of the architecture
requires good automated testing.

![image](https://user-images.githubusercontent.com/27693622/231181790-7facd428-5b0d-485d-b4b4-15e6429f52f0.png)

### Deployment pipeline - laptop to production
![image](https://user-images.githubusercontent.com/27693622/231222597-ec607498-5d35-4ed4-b3d6-4ba7931187eb.png)

### Deployment pipeline per service

![image](https://user-images.githubusercontent.com/27693622/231224768-b9e027c5-dded-4cce-857a-7809cf037201.png)

Testing should be made at the service level.

### Consumer-driven contract testing
Verify that a service (a.k.a provider) and its clients (a.k.a consumers) can communicate while testing them in isolation.

### Contract testing example
![image](https://user-images.githubusercontent.com/27693622/231228289-5776856d-3534-43b0-b298-b5a1e0bb5b0a.png)

This is an example of a contract test:
```groovy
org.springframework.cloud.contract.spec.Contract.make {
    request {
      method 'GET'
      url '/orders/99'
    }
    response {
      status 200
      headers {
        header('Content-Type': 'application/json;charset=UTF-8')
      }
      body('''{"orderId": "99", "state": "APPROVAL_PENDING"}''')
    }
}
```

The above is API definition by example. Wiremock simulates the Order Service and simulates the API.
We use the contracts to test the controller. With the same contract testing document we ensure that the
two components can communicate effectively. This can help minimize end-to-end tests.


### Testing in production
- challenge
  - end-to-end testing is brittle, slow and costly
  - end-to-end tests are a simulation of production
  - no matter how much we test issues will appear in production
- Therefore:
  - separate deployment (running in production) from release (available to users)
  - test deployed code before releasing
  - automate for fast deployment, rollback and roll forward

We can use Canary deployment and route test traffic through the new version of services to ensure that the service works
effectively.

### Transactions and Queries in Microservice Architecture
Distributed data patterns are important for transtactions and queries with Microservices.
We use the Saga pattern to enable transactions across services.

https://microservices.io/patterns/data/saga.html

![image](https://user-images.githubusercontent.com/27693622/231235236-e8815745-5892-4076-862b-b8cca95d3ccf.png)

It is not correct to do joins across services:
```sql
SELECT DISTINCT c.*
FROM CUSTOMER c, ORDER o
WHERE
    c.id = o.ID
        AND c.id = ?
        AND o.PAID_DATE >= ?
```
Joins within services are fine but across services would lead to design-time coupling.
There are two patterns for queries across services:
- API composition
- CQRS

CQRS is more powerful.

https://microservices.io/patterns/index.html#data-management

This is API composition:
![image](https://user-images.githubusercontent.com/27693622/231240010-21fba972-3daf-467d-9829-4bfc94ea12c3.png)

It is simple but can be inefficient involving too many round trips on the network.

This is CQRS:
![image](https://user-images.githubusercontent.com/27693622/231243611-7cc5b72b-b129-480d-993b-e80994d8b8cd.png)

It is more flexible but more complex and eventually consistent.
This is a helpful overview of the different patterns:
"A query that spans services cannot simply join across service databases since that's design-time coupling. ==> use the API Composition or CQRS patterns.

A command that spans services cannot use traditional distributed transactions since that's tight runtime coupling. ==> use the Saga pattern."

### Saga coordination
#### Choreography Based Sagas
- coordination logic = code that publishes events and event handlers
- when a saga participant participant updates a business object, it publishes domain events announcing what it has done
- Saga participants have event handlers that update business objects

![image](https://user-images.githubusercontent.com/27693622/231249333-f90c8688-bf4c-4a17-ad66-197cc498627f.png)

### Benefits and drawbacks of choreography
#### Benefits
- often simple especially when using event sourcing
- loose runtime coupling

#### Drawbacks
- Decentralized implementation - potentially difficult to understand
- Risk of excessive design time coupling - e.g. Customer Service must know about Order events that affect credit
- Cyclic dependencies - services listen to each other's events

https://github.com/eventuate-examples/eventuate-examples-java-customers-and-orders

This is an example of the Order Service publishing domain events:
https://github.com/eventuate-tram/eventuate-tram-examples-customers-and-orders
```java
public class OrderService {
    
    @Autowired
    private DomainEventPublisher domainEventPublisher;
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Transactional
    public Order createOrder(OrderDetails orderDetails) {
        Order order = Order.createOrder(orderDetails);
        orderRepository.save(order);
        domainEventPublisher.publish(Order.class,
                order.getId(),
                singletonList(new OrderCreatedEvent(order.getId(), orderDetails)));
        return order;
    }
}
```
Here we publish the event of creating an Order. There is then a consumer for the createOrderEvent:

```java
import java.util.Collections;

public class OrderEventConsumer {
  @Autowired
  private CustomerRepository customerRepository;

  @Autowired
  private DomainEventPublisher domainEventPublisher;

  public DomainEventHandlers domainEventHandlers() {
    return DomainEventHandlersBuilder
            .forAggregateType("io.eventuate.examples.tram.ordersandcustomers.orders.domain.Order")
            .onEvent(OrderCreatedEvent.class, this::orderCreatedEventHandler)
            .build();
  }

  private void orderCreatedEventHandler(DomainEventEnvelope<OrderCreatedEvent> domainEventEnvelope) {
    OrderCreatedEvent orderCreatedEvent = domainEventEnvelope.getEvent();
    Customer customer = customerRepository
            .findOne(orderCreatedEvent.getOrderDetails().getCustomerId());

    try {
      customer.reserveCredit(orderCreatedEvent.getOrderId(),
              orderCreatedEvent.getOrderDetails().getOrderTotal());

      CustomerCreditReservedEvent customerCreditReservedEvent =
              new CustomerCreditReservedEvent(orderCreatedEvent.getOrderId());
      domainEventPublisher.publish(Customer.class,
              customer.getId(),
              Collections.singletonList(customerCreditReservedEvent));
    } catch (CustomerCreditLimitExceededException e) {
        CustomerCreditReservationFailedEvent customerCreditReservationFailedEvent =
                new CustomerCreditReservationFailedEvent(orderCreatedEvent.getOrderId());
        
        domainEventPublisher.publish(Customer.class,
                customer.getId(),
                Collections.singletonList(customerCreditReservationFailedEvent));
    }
  }
}
```

The orderCreatedEventHandler attempts to reserve credit and if successful it publishes the CustomerCreditReservedEvent otherwise
it publishes a CustomerCreditReservationFailedEvent.



