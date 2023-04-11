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

