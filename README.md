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

![image](https://user-images.githubusercontent.com/27693622/231145812-5bd1f8b5-25a8-4358-8489-460a18fd3370.png)
