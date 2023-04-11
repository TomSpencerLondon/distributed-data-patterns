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

