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


### Using Choreography based sagas

#### Implement choreography-based saga in Account Service
Here I describe how I implemented the Account Service's choreography-based saga in order to validate the newly
created account's customerId.

#### Overview of the saga

```mermaid
---
title: Saga Choreography
---

sequenceDiagram
    autonumber
    participant Account Service
    participant Customer Service
    
    Account Service->> Account Service: createAccount() PENDING_CUSTOMER_VALIDATION
    Account Service -->> Customer Service: AccountOpenedEvent
    Customer Service->> Customer Service: validateCustomerId(AccountOpenedEvent)
    
    alt invalid input
        Customer Service-->>Account Service: CustomerValidationFailedEvent
        Account Service->> Account Service: CUSTOMER_VALIDATION_FAILED
    else valid input
        Customer Service-->>Account Service: CustomerValidatedEvent
        Account Service->> Account Service: Account OPEN
        Note left of Customer Service: Depends on valid customerId
    end
```

The above sequence diagram describes the saga:
1. Account Service creates an Account in the PENDING_CUSTOMER_VALIDATION state
2. Account Service publishes an AccountOpenedEvent
3. Customer Service consumes the AccountOpenedEvent and validates the customerId
4. Invalid: Customer Service publishes CustomerValidationFailedEvent
5. Invalid: Account Service changes state of Account to CUSTOMER_VALIDATION_FAILED
6. Valid: Customer Service publishes CustomerValidatedEvent
7. Valid: Account Service changes state of Account to OPEN

This is the code for the AccountService:
```java
@Service
@Transactional
public class AccountService {
    private Logger logger = LoggerFactory.getLogger(getClass());

    private AccountRepository accountRepository;
    private DomainEventPublisher domainEventPublisher;

    public AccountService(AccountRepository accountRepository, DomainEventPublisher domainEventPublisher) {
        this.accountRepository = accountRepository;
        this.domainEventPublisher = domainEventPublisher;
    }


    public Account openAccount(AccountInfo accountInfo) {
        logger.info("AccountService is opening account = {}", accountInfo);
        Account account = new Account(accountInfo);
        accountRepository.save(account);
        logger.info("AccountService created and saved account = {}", account);
        publishAccountOpenedEvent(account);
        logger.info("AccountService published AccountOpenedEvent for account = {}", account);
        return account;
    }

    private void publishAccountOpenedEvent(Account account) {
        domainEventPublisher.publish(Account.class,
                account.getId(),
                Collections.singletonList(new AccountOpenedEvent(account.getAccountInfo())));
    }

}
```
I added the code within publishAccountOpenedEvent(Account account). This publishes the AccountOpenedEvent.

The test for the AccountService includes a mock for the accountRepository and verifies that the domainEventPublisher was
called:
```java

public class AccountServiceTest {


  private AccountService accountService;
  private AccountRepository accountRepository;
  private DomainEventPublisher domainEventPublisher;

  @Before
  public void setUp() {
    accountRepository = mock(AccountRepository.class);
    domainEventPublisher = mock(DomainEventPublisher.class);

    accountService = new AccountService(accountRepository, domainEventPublisher);
  }

  @Test
  public void shouldOpenAccount() {

    when(accountRepository.save(any(Account.class))).then(invocation -> {
      ((Account) invocation.getArguments()[0]).setId(accountId);
      return null;
    });

    AccountInfo accountInfo = new AccountInfo(customerId, "Checking", initialBalance);
    Account account = accountService.openAccount(accountInfo);

    verify(accountRepository).save(account);

    verify(domainEventPublisher).publish(Account.class, accountId, Collections.singletonList(new AccountOpenedEvent(accountInfo)));
  }
}
```

There is also an AccountServiceIntegrationtest to verify that the service can connect to the database and message broker:
```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = AccountServiceIntegrationTest.Config.class, webEnvironment = SpringBootTest.WebEnvironment.NONE)
public class AccountServiceIntegrationTest {


  @Autowired
  private AccountService accountService;

  @Test
  public void shouldSaveAndLoadAccount() {

    AccountInfo accountInfo = AccountMother.makeAccount();

    // Account savedAccount = accountService.openAccount(accountInfo);
    Account savedAccount = accountService.openAccount(accountInfo);

    //    Account loadedAccount = accountService.findAccount(savedAccount.getId());
    Optional<Account> loadedAccount = accountService.findAccount(savedAccount.getId());

    assertTrue(loadedAccount.isPresent());

    assertEquals(accountInfo, loadedAccount.get().getAccountInfo());
  }

  @Configuration
  @Import({AccountBackendConfiguration.class,
          TramInMemoryConfiguration.class, EventuateTransactionTemplateConfiguration.class
  })
  @EnableAutoConfiguration
  public static class Config {

  }
}
```

Next I added an event handler for the CustomerValidatedEvent. Specifically, I implemented the handleCustomerValidatedEvent()
method in the AccountEventHandlers class and made it call accountService.noteCustomerValidated():

```java

public class AccountEventHandlers {
  private Logger logger = LoggerFactory.getLogger(getClass());

  private AccountService accountService;

  public AccountEventHandlers(AccountService accountService) {
    this.accountService = accountService;
  }

  public DomainEventHandlers domainEventHandlers() {
    return DomainEventHandlersBuilder
            .forAggregateType("net.chrisrichardson.bankingexample.customerservice.backend.Customer")
            .onEvent(CustomerValidatedEvent.class, this::handleCustomerValidatedEvent)
            .onEvent(CustomerValidationFailedEvent.class, this::handleCustomerValidationFailedEvent)
            .build();
  }

  private void handleCustomerValidatedEvent(DomainEventEnvelope<CustomerValidatedEvent> dee) {
    accountService.noteCustomerValidated(dee.getEvent().getAccountId());
  }
  private void handleCustomerValidationFailedEvent(DomainEventEnvelope<CustomerValidationFailedEvent> dee) {
    accountService.noteCustomerValidationFailed(dee.getEvent().getAccountId());
  }
}

```

We confirm that the class is working by running the AccountEventHandlersTest:
```java
public class AccountEventHandlersTest {

  private AccountEventHandlers accountEventHandlers;
  private AccountService accountService;

  @Before
  public void setUp() {
    accountService = mock(AccountService.class);
    accountEventHandlers = new AccountEventHandlers(accountService);
  }

  @Test
  public void shouldHandleCustomerValidatedEvent() {
    given().
      eventHandlers(accountEventHandlers.domainEventHandlers()).
    when().
      aggregate("net.chrisrichardson.bankingexample.customerservice.backend.Customer", customerId).
      publishes(new CustomerValidatedEvent(accountId)).
    then().
      verify(() -> {
        verify(accountService).noteCustomerValidated(accountId);
      })
    ;

  }
  @Test
  public void shouldHandleCustomerValidationFailedEvent() {
    given().
      eventHandlers(accountEventHandlers.domainEventHandlers()).
    when().
      aggregate("net.chrisrichardson.bankingexample.customerservice.backend.Customer", customerId).
      publishes(new CustomerValidationFailedEvent(accountId)).
    then().
      verify(() -> {
        verify(accountService).noteCustomerValidationFailed(accountId);
      })
    ;

  }

}
```
Here, we noteCustomerValidated() or noteCustomerValidationFAiled() depending on the event received.

We then run the end-to-end tests with:
```bash
./gradlew endToEndTests
```

This shows that the shouldCreateAccounts test and the shouldCreateCustomer tests pass:
![image](https://user-images.githubusercontent.com/27693622/231401395-9c01ac77-84ba-4987-bef6-276d3c384ff7.png)

Other failing tests relate to returnAccountWithCustomer, shouldTransferMoney and shouldUpdateCustomerView.
These are unimplemented.

#### Manual Testing
We then use Swagger UI to create customers and accounts:
```bash
./gradlew buildAndStartServicesMySql
```

![image](https://user-images.githubusercontent.com/27693622/231408552-441d7514-d533-4f11-8aec-79728b7e6369.png)

It returns a 200 response:
![image](https://user-images.githubusercontent.com/27693622/231408751-d19cb380-b3a9-429d-b9e1-d47d0dee3ef9.png)

I can also find the customer:
![image](https://user-images.githubusercontent.com/27693622/231409160-bcf69e6a-aebe-4460-bcb6-d9b67b5c9d52.png)

I then connect to the database and can see the updated tables and the events:

```mysql
mysql> show tables;
+----------------------------+
| Tables_in_eventuate        |
+----------------------------+
| accounts                   |
| cdc_monitoring             |
| customers                  |
| entities                   |
| events                     |
| hibernate_sequence         |
| message                    |
| offset_store               |
| received_messages          |
| saga_instance              |
| saga_instance_participants |
| saga_lock_table            |
| saga_stash_table           |
| snapshots                  |
| transfers                  |
+----------------------------+
15 rows in set (0.00 sec)

mysql> select * from customers;
+----+---------+----------------+---------------+----------+----------+------------+-----------+--------------+------------+
| id | city    | state          | street1       | street2  | zip_code | first_name | last_name | phone_number | ssn        |
+----+---------+----------------+---------------+----------+----------+------------+-----------+--------------+------------+
|  1 | Oakland | CA             | 1 high street | NULL     | 94719    | John       | Doe       | 510-555-1212 | xxx-yy-zzz |
|  4 | Oakland | CA             | 1 high street | NULL     | 94719    | John       | Doe       | 510-555-1212 | xxx-yy-zzz |
|  7 | Oakland | CA             | 1 high street | NULL     | 94719    | John       | Doe       | 510-555-1212 | xxx-yy-zzz |
|  8 | Oakland | CA             | 1 high street | NULL     | 94719    | John       | Doe       | 510-555-1212 | xxx-yy-zzz |
| 11 | Oakland | CA             | 1 high street | NULL     | 94719    | John       | Doe       | 510-555-1212 | xxx-yy-zzz |
| 14 | London  | United Kingdom | Street 1      | Street 2 | SW24 3DQ | Tom        | Spencer   | 07954343212  | 4321123    |
+----+---------+----------------+---------------+----------+----------+------------+-----------+--------------+------------+
6 rows in set (0.00 sec)

mysql> select * from accounts;
+----+--------+-------------+----------+-------+
| id | amount | customer_id | name     | state |
+----+--------+-------------+----------+-------+
|  2 |   1234 |           1 | checking | OPEN  |
|  3 |  10086 |           1 | saving   | OPEN  |
|  5 |   1234 |           4 | checking | OPEN  |
|  6 |  10086 |           4 | saving   | OPEN  |
|  9 |   1234 |           8 | checking | OPEN  |
| 10 |  10086 |           8 | saving   | OPEN  |
| 12 |   1234 |          11 | checking | OPEN  |
| 13 |  10086 |          11 | saving   | OPEN  |
+----+--------+-------------+----------+-------+
8 rows in set (0.00 sec)

mysql> select * from message\G
*************************** 1. row ***************************
           id: 000001877498d085-0242ac12000b0000
  destination: net.chrisrichardson.bankingexample.customerservice.backend.Customer
      headers: {"PARTITION_ID":"1","event-aggregate-type":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","DATE":"Wed, 12 Apr 2023 08:33:04 GMT","b3":"94c5879a28d2da37-144eb4ff36828a1e-1","event-aggregate-id":"1","event-type":"net.chrisrichardson.bankingexample.customerservice.common.CustomerCreatedEvent","DESTINATION":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","ID":"000001877498d085-0242ac12000b0000"}
      payload: {"customerInfo":{"name":{"firstName":"John","lastName":"Doe"},"phoneNumber":"510-555-1212","address":{"street1":"1 high street","city":"Oakland","state":"CA","zipCode":"94719"},"ssn":"xxx-yy-zzz"}}
    published: 0
creation_time: 1681288384660
*************************** 2. row ***************************
           id: 000001877498d36e-0242ac12000c0000
  destination: net.chrisrichardson.bankingexample.accountservice.backend.Account
      headers: {"PARTITION_ID":"2","event-aggregate-type":"net.chrisrichardson.bankingexample.accountservice.backend.Account","DATE":"Wed, 12 Apr 2023 08:33:05 GMT","b3":"ccdb007493e7d9a1-4ab6ce1a5db2c6b5-1","event-aggregate-id":"2","event-type":"net.chrisrichardson.bankingexample.accountservice.common.events.AccountOpenedEvent","DESTINATION":"net.chrisrichardson.bankingexample.accountservice.backend.Account","ID":"000001877498d36e-0242ac12000c0000"}
      payload: {"accountInfo":{"customerId":1,"name":"checking","balance":{"amount":1234}}}
    published: 0
creation_time: 1681288385429
*************************** 3. row ***************************
           id: 000001877498d554-0242ac12000b0000
  destination: net.chrisrichardson.bankingexample.customerservice.backend.Customer
      headers: {"PARTITION_ID":"1","event-aggregate-type":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","DATE":"Wed, 12 Apr 2023 08:33:05 GMT","b3":"ccdb007493e7d9a1-1f3b6f8068dc9bb8-1","event-aggregate-id":"1","event-type":"net.chrisrichardson.bankingexample.customerservice.common.CustomerValidatedEvent","DESTINATION":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","ID":"000001877498d554-0242ac12000b0000"}
      payload: {"accountId":2}
    published: 0
creation_time: 1681288385878
*************************** 4. row ***************************
           id: 000001877498d78f-0242ac12000c0000
  destination: net.chrisrichardson.bankingexample.accountservice.backend.Account
      headers: {"PARTITION_ID":"3","event-aggregate-type":"net.chrisrichardson.bankingexample.accountservice.backend.Account","DATE":"Wed, 12 Apr 2023 08:33:06 GMT","b3":"1164f685d183ff64-ebfce8f00d1a8385-1","event-aggregate-id":"3","event-type":"net.chrisrichardson.bankingexample.accountservice.common.events.AccountOpenedEvent","DESTINATION":"net.chrisrichardson.bankingexample.accountservice.backend.Account","ID":"000001877498d78f-0242ac12000c0000"}
      payload: {"accountInfo":{"customerId":1,"name":"saving","balance":{"amount":10086}}}
    published: 0
creation_time: 1681288386448
*************************** 5. row ***************************
           id: 000001877498d7c8-0242ac12000b0000
  destination: net.chrisrichardson.bankingexample.customerservice.backend.Customer
      headers: {"PARTITION_ID":"1","event-aggregate-type":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","DATE":"Wed, 12 Apr 2023 08:33:06 GMT","b3":"1164f685d183ff64-6c96ba9960c1c7f1-1","event-aggregate-id":"1","event-type":"net.chrisrichardson.bankingexample.customerservice.common.CustomerValidatedEvent","DESTINATION":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","ID":"000001877498d7c8-0242ac12000b0000"}
      payload: {"accountId":3}
    published: 0
creation_time: 1681288386505
*************************** 6. row ***************************
           id: 000001877498da5e-0242ac12000b0000
  destination: net.chrisrichardson.bankingexample.customerservice.backend.Customer
      headers: {"PARTITION_ID":"4","event-aggregate-type":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","DATE":"Wed, 12 Apr 2023 08:33:07 GMT","b3":"4e454678789486e7-2910836d46d9578d-1","event-aggregate-id":"4","event-type":"net.chrisrichardson.bankingexample.customerservice.common.CustomerCreatedEvent","DESTINATION":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","ID":"000001877498da5e-0242ac12000b0000"}
      payload: {"customerInfo":{"name":{"firstName":"John","lastName":"Doe"},"phoneNumber":"510-555-1212","address":{"street1":"1 high street","city":"Oakland","state":"CA","zipCode":"94719"},"ssn":"xxx-yy-zzz"}}
    published: 0
creation_time: 1681288387168
*************************** 7. row ***************************
           id: 000001877498db04-0242ac12000c0000
  destination: net.chrisrichardson.bankingexample.accountservice.backend.Account
      headers: {"PARTITION_ID":"5","event-aggregate-type":"net.chrisrichardson.bankingexample.accountservice.backend.Account","DATE":"Wed, 12 Apr 2023 08:33:07 GMT","b3":"7df136e991e46e4d-843fe138fc3e2583-1","event-aggregate-id":"5","event-type":"net.chrisrichardson.bankingexample.accountservice.common.events.AccountOpenedEvent","DESTINATION":"net.chrisrichardson.bankingexample.accountservice.backend.Account","ID":"000001877498db04-0242ac12000c0000"}
      payload: {"accountInfo":{"customerId":4,"name":"checking","balance":{"amount":1234}}}
    published: 0
creation_time: 1681288387336
*************************** 8. row ***************************
           id: 000001877498db7d-0242ac12000b0000
  destination: net.chrisrichardson.bankingexample.customerservice.backend.Customer
      headers: {"PARTITION_ID":"4","event-aggregate-type":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","DATE":"Wed, 12 Apr 2023 08:33:07 GMT","b3":"7df136e991e46e4d-ccfbca817234b3c7-1","event-aggregate-id":"4","event-type":"net.chrisrichardson.bankingexample.customerservice.common.CustomerValidatedEvent","DESTINATION":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","ID":"000001877498db7d-0242ac12000b0000"}
      payload: {"accountId":5}
    published: 0
creation_time: 1681288387456
*************************** 9. row ***************************
           id: 000001877498de0f-0242ac12000c0000
  destination: net.chrisrichardson.bankingexample.accountservice.backend.Account
      headers: {"PARTITION_ID":"6","event-aggregate-type":"net.chrisrichardson.bankingexample.accountservice.backend.Account","DATE":"Wed, 12 Apr 2023 08:33:08 GMT","b3":"c01f8e7c16410194-9ba14c49096c8e33-1","event-aggregate-id":"6","event-type":"net.chrisrichardson.bankingexample.accountservice.common.events.AccountOpenedEvent","DESTINATION":"net.chrisrichardson.bankingexample.accountservice.backend.Account","ID":"000001877498de0f-0242ac12000c0000"}
      payload: {"accountInfo":{"customerId":4,"name":"saving","balance":{"amount":10086}}}
    published: 0
creation_time: 1681288388114
*************************** 10. row ***************************
           id: 000001877498de48-0242ac12000b0000
  destination: net.chrisrichardson.bankingexample.customerservice.backend.Customer
      headers: {"PARTITION_ID":"4","event-aggregate-type":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","DATE":"Wed, 12 Apr 2023 08:33:08 GMT","b3":"c01f8e7c16410194-04ad98d28fc39f25-1","event-aggregate-id":"4","event-type":"net.chrisrichardson.bankingexample.customerservice.common.CustomerValidatedEvent","DESTINATION":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","ID":"000001877498de48-0242ac12000b0000"}
      payload: {"accountId":6}
    published: 0
creation_time: 1681288388187
*************************** 11. row ***************************
           id: 000001877498e158-0242ac12000b0000
  destination: net.chrisrichardson.bankingexample.customerservice.backend.Customer
      headers: {"PARTITION_ID":"7","event-aggregate-type":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","DATE":"Wed, 12 Apr 2023 08:33:08 GMT","b3":"a729d6bd86c46e5e-540afe75ba036daf-1","event-aggregate-id":"7","event-type":"net.chrisrichardson.bankingexample.customerservice.common.CustomerCreatedEvent","DESTINATION":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","ID":"000001877498e158-0242ac12000b0000"}
      payload: {"customerInfo":{"name":{"firstName":"John","lastName":"Doe"},"phoneNumber":"510-555-1212","address":{"street1":"1 high street","city":"Oakland","state":"CA","zipCode":"94719"},"ssn":"xxx-yy-zzz"}}
    published: 0
creation_time: 1681288388954
*************************** 12. row ***************************
           id: 000001877498e1a4-0242ac12000b0000
  destination: net.chrisrichardson.bankingexample.customerservice.backend.Customer
      headers: {"PARTITION_ID":"8","event-aggregate-type":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","DATE":"Wed, 12 Apr 2023 08:33:09 GMT","b3":"1305cc7b40873c29-7f0858194ec36939-1","event-aggregate-id":"8","event-type":"net.chrisrichardson.bankingexample.customerservice.common.CustomerCreatedEvent","DESTINATION":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","ID":"000001877498e1a4-0242ac12000b0000"}
      payload: {"customerInfo":{"name":{"firstName":"John","lastName":"Doe"},"phoneNumber":"510-555-1212","address":{"street1":"1 high street","city":"Oakland","state":"CA","zipCode":"94719"},"ssn":"xxx-yy-zzz"}}
    published: 0
creation_time: 1681288389030
*************************** 13. row ***************************
           id: 000001877498e1f2-0242ac12000c0000
  destination: net.chrisrichardson.bankingexample.accountservice.backend.Account
      headers: {"PARTITION_ID":"9","event-aggregate-type":"net.chrisrichardson.bankingexample.accountservice.backend.Account","DATE":"Wed, 12 Apr 2023 08:33:09 GMT","b3":"dc1c96fac85dacea-fed5410627b154d0-1","event-aggregate-id":"9","event-type":"net.chrisrichardson.bankingexample.accountservice.common.events.AccountOpenedEvent","DESTINATION":"net.chrisrichardson.bankingexample.accountservice.backend.Account","ID":"000001877498e1f2-0242ac12000c0000"}
      payload: {"accountInfo":{"customerId":8,"name":"checking","balance":{"amount":1234}}}
    published: 0
creation_time: 1681288389107
*************************** 14. row ***************************
           id: 000001877498e224-0242ac12000b0000
  destination: net.chrisrichardson.bankingexample.customerservice.backend.Customer
      headers: {"PARTITION_ID":"8","event-aggregate-type":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","DATE":"Wed, 12 Apr 2023 08:33:09 GMT","b3":"dc1c96fac85dacea-d61acf5d269bb0d3-1","event-aggregate-id":"8","event-type":"net.chrisrichardson.bankingexample.customerservice.common.CustomerValidatedEvent","DESTINATION":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","ID":"000001877498e224-0242ac12000b0000"}
      payload: {"accountId":9}
    published: 0
creation_time: 1681288389163
*************************** 15. row ***************************
           id: 000001877498e4b6-0242ac12000c0000
  destination: net.chrisrichardson.bankingexample.accountservice.backend.Account
      headers: {"PARTITION_ID":"10","event-aggregate-type":"net.chrisrichardson.bankingexample.accountservice.backend.Account","DATE":"Wed, 12 Apr 2023 08:33:09 GMT","b3":"02d3f753d1cea9e5-b411c48afe5da324-1","event-aggregate-id":"10","event-type":"net.chrisrichardson.bankingexample.accountservice.common.events.AccountOpenedEvent","DESTINATION":"net.chrisrichardson.bankingexample.accountservice.backend.Account","ID":"000001877498e4b6-0242ac12000c0000"}
      payload: {"accountInfo":{"customerId":8,"name":"saving","balance":{"amount":10086}}}
    published: 0
creation_time: 1681288389817
*************************** 16. row ***************************
           id: 000001877498e4ff-0242ac12000b0000
  destination: net.chrisrichardson.bankingexample.customerservice.backend.Customer
      headers: {"PARTITION_ID":"8","event-aggregate-type":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","DATE":"Wed, 12 Apr 2023 08:33:09 GMT","b3":"02d3f753d1cea9e5-413c3bb5b16b0288-1","event-aggregate-id":"8","event-type":"net.chrisrichardson.bankingexample.customerservice.common.CustomerValidatedEvent","DESTINATION":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","ID":"000001877498e4ff-0242ac12000b0000"}
      payload: {"accountId":10}
    published: 0
creation_time: 1681288389888
*************************** 17. row ***************************
           id: 000001877498e7c3-0242ac12000b0000
  destination: net.chrisrichardson.bankingexample.customerservice.backend.Customer
      headers: {"PARTITION_ID":"11","event-aggregate-type":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","DATE":"Wed, 12 Apr 2023 08:33:10 GMT","b3":"ed32284c14185d90-bbc6927b9e4d576f-1","event-aggregate-id":"11","event-type":"net.chrisrichardson.bankingexample.customerservice.common.CustomerCreatedEvent","DESTINATION":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","ID":"000001877498e7c3-0242ac12000b0000"}
      payload: {"customerInfo":{"name":{"firstName":"John","lastName":"Doe"},"phoneNumber":"510-555-1212","address":{"street1":"1 high street","city":"Oakland","state":"CA","zipCode":"94719"},"ssn":"xxx-yy-zzz"}}
    published: 0
creation_time: 1681288390597
*************************** 18. row ***************************
           id: 000001877498e7fe-0242ac12000c0000
  destination: net.chrisrichardson.bankingexample.accountservice.backend.Account
      headers: {"PARTITION_ID":"12","event-aggregate-type":"net.chrisrichardson.bankingexample.accountservice.backend.Account","DATE":"Wed, 12 Apr 2023 08:33:10 GMT","b3":"5cce2eee5eec6f94-d1d7fc72fb476d49-1","event-aggregate-id":"12","event-type":"net.chrisrichardson.bankingexample.accountservice.common.events.AccountOpenedEvent","DESTINATION":"net.chrisrichardson.bankingexample.accountservice.backend.Account","ID":"000001877498e7fe-0242ac12000c0000"}
      payload: {"accountInfo":{"customerId":11,"name":"checking","balance":{"amount":1234}}}
    published: 0
creation_time: 1681288390656
*************************** 19. row ***************************
           id: 000001877498e82c-0242ac12000b0000
  destination: net.chrisrichardson.bankingexample.customerservice.backend.Customer
      headers: {"PARTITION_ID":"11","event-aggregate-type":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","DATE":"Wed, 12 Apr 2023 08:33:10 GMT","b3":"5cce2eee5eec6f94-bb98ef39c24b4443-1","event-aggregate-id":"11","event-type":"net.chrisrichardson.bankingexample.customerservice.common.CustomerValidatedEvent","DESTINATION":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","ID":"000001877498e82c-0242ac12000b0000"}
      payload: {"accountId":12}
    published: 0
creation_time: 1681288390702
*************************** 20. row ***************************
           id: 000001877498ea74-0242ac12000c0000
  destination: net.chrisrichardson.bankingexample.accountservice.backend.Account
      headers: {"PARTITION_ID":"13","event-aggregate-type":"net.chrisrichardson.bankingexample.accountservice.backend.Account","DATE":"Wed, 12 Apr 2023 08:33:11 GMT","b3":"db6bb80071ea11b1-19a35c3f8d930da5-1","event-aggregate-id":"13","event-type":"net.chrisrichardson.bankingexample.accountservice.common.events.AccountOpenedEvent","DESTINATION":"net.chrisrichardson.bankingexample.accountservice.backend.Account","ID":"000001877498ea74-0242ac12000c0000"}
      payload: {"accountInfo":{"customerId":11,"name":"saving","balance":{"amount":10086}}}
    published: 0
creation_time: 1681288391286
*************************** 21. row ***************************
           id: 000001877498eaaa-0242ac12000b0000
  destination: net.chrisrichardson.bankingexample.customerservice.backend.Customer
      headers: {"PARTITION_ID":"11","event-aggregate-type":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","DATE":"Wed, 12 Apr 2023 08:33:11 GMT","b3":"db6bb80071ea11b1-48512a7de7b66fd2-1","event-aggregate-id":"11","event-type":"net.chrisrichardson.bankingexample.customerservice.common.CustomerValidatedEvent","DESTINATION":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","ID":"000001877498eaaa-0242ac12000b0000"}
      payload: {"accountId":13}
    published: 0
creation_time: 1681288391341
*************************** 22. row ***************************
           id: 0000018774b4f259-0242ac12000b0000
  destination: net.chrisrichardson.bankingexample.customerservice.backend.Customer
      headers: {"PARTITION_ID":"14","event-aggregate-type":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","DATE":"Wed, 12 Apr 2023 09:03:48 GMT","b3":"ba5e608aa172fc64-b3dc9bfe7a4edad9-1","event-aggregate-id":"14","event-type":"net.chrisrichardson.bankingexample.customerservice.common.CustomerCreatedEvent","DESTINATION":"net.chrisrichardson.bankingexample.customerservice.backend.Customer","ID":"0000018774b4f259-0242ac12000b0000"}
      payload: {"customerInfo":{"name":{"firstName":"Tom","lastName":"Spencer"},"phoneNumber":"07954343212","address":{"street1":"Street 1","street2":"Street 2","city":"London","state":"United Kingdom","zipCode":"SW24 3DQ"},"ssn":"4321123"}}
    published: 0
creation_time: 1681290228315
22 rows in set (0.00 sec)
```

To clean up I run:
```bash
./gradlew stopServicesMySql
```

### Orchestration Based Sagas
- Introduction to orchestration-based sagas
- Code walkthrough
- Message and API design

#### Orchestration-based coordination using command Messages
![image](https://user-images.githubusercontent.com/27693622/231420883-0a9104e7-5216-4623-8bd0-e8f7393a929c.png)

Here the orchestrator tells the participants what to do. The orchestrator invokes the participants using
asynchronous request-response.

![image](https://user-images.githubusercontent.com/27693622/231423651-e9b6d9f8-d6a7-46b3-8817-8ffc2184c8fb.png)

The orchestrator sends a command and the Customer Service then replies to the Command via the point-to-point reply channel.
The saga (orchestrator) is a persistent object that implements a state machine and invokes the participants.
It sends command messages to saga participants and when a reply comes back this triggers a state transition which then
sends a message to the next particpant.

#### Saga orchestrator behaviour
- on create:
  - invokes a saga participant
  - persists state in the database
  - waits for a reply
- On reply:
  - load state from database
  - determine which saga participant to invoke next
  - invokes saga participant
  - updates its state
  - persists updated state
  - waits for a reply
  - ...

![image](https://user-images.githubusercontent.com/27693622/231429442-be75fa57-f291-479d-a300-26d0aa317c7f.png)

#### Benefits and drawbacks of orchestration
- Benefits:
  - Centralized coordination logic is easier to understand
  - saga state is queryable
  - reduced coupling, e.g. Customer Service knows less. Simply has
API for managing available credit
  - Reduces cyclic dependencies
- Drawbacks:
  - Risk of smart sagas directing dumb services
  - More complex implementation that requires a framework

We can now look at the code that implements the CreateOrderSaga:

```java
public class LocalCreateOrderSaga implements SimpleSaga<LocalCreateOrderSaga> {
    private DomainEventPublisher domainEventPublisher;
    private OrderRepository orderRepository;
    
    public LocalCreateOrderSaga(DomainEventPublisher domainEventPublisher,
                                OrderRepository orderRepository) {
        this.domainEventPublisher = domainEventPublisher;
        this.orderRepository = orderRepository;
    }
    
    private SagaDefinition<LocalCreateOrderSaga> sagaDefinition =
            step()
                    .invokeLocal(this::create)
                    .withcompensation(this::reject)
                    .step()
                    .invokeParticipant(this::reserveCredit)
                    .step()
                    .invokeLocal(this::approve)
                    .build();
}
```
Here the saga consists of three steps, each step has a transaction and a rollback on the transaction.
This is the rest of the class:
```java

public class LocalCreateOrderSaga implements SimpleSaga<LocalCreateOrderSaga> {
    private SagaDefinition<LocalCreateOrderSaga> sagaDefinition =
            step()
                    .invokeLocal(this::create)
                    .withCompensation(this::reject)
            .step()
                    .invokeParticipant(this::reserveCredit)
            .step()
                    .invokeLocal(this::approve)
            .build();
    
    private void create(LocalCreateOrderSaga data) {
        ResultWithEvents<Order> oe = Order.createOrder(data.getOrderDetails());
        Order order = oe.result();
        orderRepository.save(order);
        data.setOrderId(order.getId());
    }
    
    private CommandWithDestination reserveCredit(LocalCreateOrderSaga data) {
        long orderId = data.getOrderId();
        Long customerId = data.getOrderDetails().getCustomerId();
        Money orderTotal = data.getOrderDetails().getOrderTotal();
        return send(new ReserveCreditCommand(customerId, orderId, orderTotal))
                .to("customerService")
                .build();
    }
} 
```
The above code executes the save for the order and then builds a command to send.
This is the Command Handler in the Customer Service:

```java
public class CustomerCommandHandler {
    @Autowired
    private CustomerRepository customerRepository;
    
    public CommandHandlers commandHandlerDefinitions() {
        return SagaCommandHandlersBuilder
                .fromChannel("customerService")
                .onMessage(ReserveCreditCommand.class, this::reserveCredit)
                .build();
    }
    
    public Message reserveCredit(CommandMessage<ReserveCreditCommand> cm) {
        ReserveCreditCommand cmd = cm.getCommand();
        long customerId = cmd.getCustomerId();
        Customer customer = customerRepository.findOne(customerId);
        
        try {
            customer.reserveCredit(cmd.getOrderId(), cmd.getOrderTotal());
            return withSuccess(new CustomerCreditReserved());
        } catch (CustomerCreditLimitExceededException e) {
            return withFailure(new CustomerCreditReservationFailed());
        }
    }
}
```

The above CustomerCommandHandler in the Customer service is invoked for each message that is spread from the Customer
Service channel. The reserveCredit retrieves the customer and attempts to reserve credit.

#### API design
Here the Order Service asynchronously invokes the CustomerService and neither publishes nor subscribes to events:
![image](https://user-images.githubusercontent.com/27693622/231440520-6f8b920e-98e7-401c-ad1d-612bd3a43412.png)

The Customer Service has a simple API with createCustomer() invoked synchronously and reserveCredit() and releaseCredit()
invoked asynchronously. Like the Order Service it neither publishes nor consumes events:

![image](https://user-images.githubusercontent.com/27693622/231441592-c357b8ed-bd50-4656-983f-355c8aeada3d.png)

#### Message Design
- Command Message
  - Headers
    - Type - identifies the command to invoke
    - ReplyTo - speicifies the reply channel
    - SagaId - correlation ID that identifies saga and is returned in the reply
  - Payload - command arguments
- Reply Message
  - Headers
    - Type - a header identifies the reply type
    - SagaId - correlation ID that identifies saga
  - Payload - results, if any

#### Message channel design

![image](https://user-images.githubusercontent.com/27693622/231443608-4f92f376-fc29-4e60-aec2-3f02de935d7f.png)

The orchestrator sends the command to the Customer Service command channel which is a point-to-point channel.
The centralized orchestrator invokes participants using asynchronous request/response. The ochestrator sends
a command messsage to a participant's command channel. The participant executes the command and sends a reply message
to the orchestrator's reply channel. Implementing orchestration-based sagas typically requires a framework but can
be easier to understand.

```mermaid
---
title: "Saga Orchestration: Order Service"
---

sequenceDiagram
    autonumber
    participant OrderService
    participant CreateOrderSaga
    participant CustomerService
    
    Order Service-> CreateOrderSaga: create()
    CreateOrderSaga->> CreateOrderSaga: create() Order PENDING
    CreateOrderSaga -->> Customer Service: ReserveCreditCommand
    Customer Service->> Customer Service: reserveCredit(Order)
    
    alt invalid input
        Customer Service-->>CreateOrderSaga: CommandReplyOutcome.FAILURE
        CreateOrderSaga->> CreateOrderSaga: Order REJECTED
    else valid input
        Customer Service-->>CreateOrderSaga: CommandReplyOutcome.SUCCESS
        CreateOrderSaga->> CreateOrderSaga: Order PENDING
        Note left of Customer Service: Depends on credit available
    end
```

#### Implement orchestration-based saga in Money Transfer Service
The story for this task is:
```text
As a Customer
I want to transfer money between two accounts
So that I can manage my finances
```
There are two parts to this task:
1. Complete orchestration-based saga in Money Transfer Service in:
2. Enhance the Account Service to subscribe to commands sent by the Money Transfer Service to credit or debit the appropriate account

This is the sequence diagram for the Money Transfer Service Saga:

```mermaid
---
title: "Saga Orchestration: Money Transfer Service"
---

sequenceDiagram
    autonumber
    participant MoneyTransferService
    participant TransferMoneySaga
    participant MoneyTransferServiceCommandHandlers
    participant AccountCommandHandlers
    participant AccountService
    
    MoneyTransferService--> TransferMoneySaga: createTransferMoneySaga()
    TransferMoneySaga->> TransferMoneySaga: complete(TransferMoneySagaData)
    MoneyTransferService --> AccountCommandHandlers: credit(CommandMessage)
    AccountCommandHandlers--> AccountService: credit()
    AccountCommandHandlers --> MoneyTransferService: SUCCESS
```

![image](https://user-images.githubusercontent.com/27693622/231465051-0fc8fc72-6391-45ae-a874-733f42efde8c.png)

![image](https://user-images.githubusercontent.com/27693622/231469191-56907185-3659-4b81-aaac-b72995ada8a6.png)

![image](https://user-images.githubusercontent.com/27693622/231469684-1e639f5e-c989-4300-b92e-d5bbfc684e8f.png)


At the moment the transfers are still in progress:
```mysql
mysql> select * from transfers;
+----+--------+-----------------+---------------+-------------+
| id | amount | from_account_id | to_account_id | state       |
+----+--------+-----------------+---------------+-------------+
|  3 |    600 |               1 |             2 | IN_PROGRESS |
|  7 |    600 |               5 |             6 | IN_PROGRESS |
+----+--------+-----------------+---------------+-------------+
2 rows in set (0.00 sec)

```

We still need to save the changes in the accounts.

Useful docker command:
```text
kill all running containers with docker kill $(docker ps -q)
delete all stopped containers with docker rm $(docker ps -a -q)
```

### Saga Counter Measures
ACID = Atomicity Consistency Isolation and Durability
We want to be able to implement Isolation with our sagas.
At the moment the Create Order Saga is quite simple:

|           | Participant      | Participant     | Participant   |
|-----------|------------------|-----------------|---------------|
| 1         | Order Service    | createOrder()   | rejectOrder() |
| 2 (pivot) | Customer Service | reserveCredit() | -             |

It doesn't have a third approve order step. In real applications we need to be able to cancel orders.
This is a cancel order saga with two steps:

|           | Participant      | Participant     | Participant |
|-----------|------------------|-----------------|-------------|
| 1 (pivot) | Order Service    | cancelOrder()   | -           |
| 2         | Customer Service | reserveCredit() | -           |

The sagas are interleaved which can make the inconsistent. This means that we could possibly release credit that wasn't reserved.
Sagas are ACD:
- Atomicity
  - saga implementation ensures that all transactions are executed OR all are compensated
- Consistency
  - Referential integrity within a service handled by local databases
  - referential integrity across services handled by application
- ~~Isolation~~
  - ~~Concurrent execution of multiple transactions is equivalent to a sequential execution~~
- Durability
  - Durability handled by local databases

Sagas are Atomic, Consistent and Durable but lack Isolation:
- Isolation guarantees the outcome of executing multiple transactions concurrently is equivalent to a sequential execution
- For example: outcome of concurrent execution of Txn A and Txn B is either
  - Txn A => Txn B
  - Txn B => Txn A
- Classic implementation uses locks
- Modern databases avoid locking by using multiversion concurrency control

Lack of isolation => outcome of concurrent execution != a sequential execution = Data anomolies

### Data Anomolies
1. Lost update
2. Dirty read - one transactoin reads the updates of another
3. Fuzzy read / non repeatable read - two steps read different data which has changed in the interim

We should use counter measures to reduce the impace of anomolies.

#### Semantic Lock - countermeasure
We saw this in Order Service createOrder() PENDING which locks the order, approveOrder() APPROVED, rejectedOrder() REJECTED - release the lock.
The semantic lock could be a boolean or additional state in the state machine.

#### Commutative Update - countermeasure
Design commands to be commutative - addition and subtraction are commutative.

#### Pessimistic View - countermeasure
Only do release funds after the possible cancel order and cancel delivery steps.

#### Re-read value - countermeasure
Verify the data is unchanged. If the data is changed abort the saga.

#### By value - countermeasure
Dynamically selects countermeasures based on business risk.


### Querying in Microservice Architecture
If a query spans services we need to use the API Composition or CQRS patterns to implement the queries.
Sometimes we might have a query that spans two services:

![image](https://user-images.githubusercontent.com/27693622/231514776-2f92467a-70fd-45da-aeb6-cb7ac79d94d9.png)

Queries that retrieve data from single services are straightforward. Queries that span services in microservices are
always challenging. Data for Customer and Order are private to their respective services. This is important because
otherwise the services could suffer from design time coupling. The two patterns to help with this issue are API composition
and CQRS:
https://microservices.io/patterns/index.html#data-management

API composition is the simplest, the more powerful pattern is CQRS.

This is API composition:
![image](https://user-images.githubusercontent.com/27693622/231240010-21fba972-3daf-467d-9829-4bfc94ea12c3.png)

It is simple but can be inefficient involving too many round trips on the network.

This is CQRS:
![image](https://user-images.githubusercontent.com/27693622/231243611-7cc5b72b-b129-480d-993b-e80994d8b8cd.png)

CQRS is more flexible but more complex and eventually consistent. In CQRS we define a view database, which is a read-only replica that 
is designed to support that query. The application keeps the replica up to date by subscribing to Domain events published 
by the service that own the data.
https://microservices.io/patterns/data/cqrs.html


#### API composition Pattern: Order Service

![image](https://user-images.githubusercontent.com/27693622/231524045-498641ac-f1e0-4e0a-80e1-d3129bed5b7c.png)

The API Gateway here plays the role of Composer and the services are providers. In a variation on the pattern the API gateway might
route a request to the Order Service and the Order Service would then play the role of composer for all the provider queries
to each service. This is useful when the API gateway is unchangeable.

API composition provides read your own writes consistency but does not provide ACID consistency across transactions. The simplest
approach of API composition is to execute the queries in sequence but it would be faster to query each service asynchronously so that
only the slowest query affects the time of the request. Reactive programming can be used for transforming outcomes in a non-blocking 
asynchronous manner. For example: Java Completable Future, RxJava Observable, Spring 5 Mono and Flux...
This is the flow diagram for asynchronous reactive http requests:
![image](https://user-images.githubusercontent.com/27693622/231534930-3e9df230-3ca0-49f9-8525-5bfc6f02909a.png)

This is the code:

```java
@Service
public class OrderServiceProxy {
    
    private OrderDestinations orderDestinations;
    private WebClient client;
    
    public OrderServiceProxy(OrderDestinations orderDestinations, WebClient webClient) {
        //..
    }
    
    public Mono<OrderInfo> findOrderById(String orderId) {
        Mono<ClientResponse> response = client
                .get()
                .uri(orderDestinations.getOrderServiceUrl() + "/orders/{orderId}", orderId)
                .exchange();
        
        return response.flatMap(resp -> {
           switch (resp.statusCode()) {
             case OK:
                 return resp.bodyToMono(OrderInfo.class);
             case NOT_FOUND:
                 return Mono.error(new OrderNotFoundException());
             default:
                 return Mono.error(new RuntimeException("Unkown" + resp.statusCode()));
           }
        });
    }
}
```

In the above code we call the get method on the client and then transform the mono into either success or not found exception.
FlatMap returns a second mono which is then updated when the client responds.

![image](https://user-images.githubusercontent.com/27693622/231537204-7a0480dc-197e-4346-8e13-68f4152ac399.png)

Here is a more complex example of Mono-based code:

```java

import java.util.Optional;

public class OrderHandlers {
  public OrderHandlers(OrderServiceProxy orderServiceProxy,
                       RestaurantOrderService restaurantOrderService,
                       DeliveryService deliveryService,
                       AccountingService accountingService) {
    //...
  }


  public Mono<ServerResponse> getOrderDetails(ServerRequest serverRequest) {
    String orderId = serverRequest.pathVariable("orderId");
    Mono<OrderInfo> orderInfo = orderService.findOrderById(orderId);
    Mono<Optional<RestaurantInfo>> restaurantOrderInfo = restaurantOrderService
            .findRestaurantOrderByOrderId(orderId)
            .map(Optional::of)
            .onErrorReturn(Optional.empty());

    Mono<Optional<BillInfo>> billInfo = accountingService
            .findBillByOrderId(orderId)
            .map(Optional::of)
            .onErrorReturn(Optional.empty());
    
    Mono<Tuple4<OrderInfo, Optional<RestaurantOrderInfo>, Optional<DeliveryInfo>, Optional<BillInfo>>> combined =
            Mono.when(orderInfo, restaurantOrderInfo, deliveryInfo, billInfo);
    
    Mono<OrderDetails> orderDetails = combined.map(OrderDetails::makeOrderDetails);
    
    return orderDetails.flatMap(od -> ServerResponse.ok()
            .contentType(MediaType.APPLICATION_JSON)
            .body(fromObject(od)));
  }
}
```

In the above code we invvoke four services and then combine the four monos into a single mono and then combine the mono result
into a single response.

![image](https://user-images.githubusercontent.com/27693622/231539937-77d096c6-25f2-4c03-a5bb-667dc8276c99.png)

The API composer invokes the provider synchronously so there can be lower availability and higher response times.
The Composer should use Timeouts, Retries, Circuit Breaker and use a fallback mechanism for each optional provider:
empty, default or cached data.