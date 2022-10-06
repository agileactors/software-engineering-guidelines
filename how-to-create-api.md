# How To Create a new API service 

Creating an API is not always an easy task. You have to take into account many different parameters and each API is 
different than the others.

If this is your first time, creating an API service then you should work in small tasks:

### Understand the requirements of your task at hand and design the API

What exactly does our API have to do? 
What code statuses is going to return under which circumstances? 
If you've been a given a task with requirements then everything is easier.Use [OpenApi](https://swagger.io/specification) 
to design the API upfront since this will be the specification to communicate with whoever wants to integrate with (the 
contract).

HTTP Statuses Explained: https://httpstatuses.org/

### Create the skeleton of your project

Most of the times you don't have to re-invent the wheel. Check if the organization you work at, has a template service 
to fork. After you know what you want to implement and decide on the tools, it is time to create your project. Start 
with the very basics; for example, if you're working with Maven - select the archetype you want to use. Then fill in 
the pom.xml with the frameworks, libraries and tools you decided upon on task 2. After that you're ready for the 
implementation! It is good to use the same libraries and frameworks the other APIs have in your company. If you
don't have anywhere to see for guidance, then search for the best tools for your project. Have in mind though,
that your list with the frameworks and libraries will grow also during the implementation of your service.

In case you're using Spring Boot find [here](https://github.com/agileactors/spring-boot-template-service) a sample 
template service. 

Code linting is very important so go ahead and use [checkstyle](https://checkstyle.sourceforge.io/) before writing any 
piece of code! Using Google's standard is ok if your organization doesn't have one.

### Design the database tables

Design and review the database tables! You will base your class domain on this. Use a tool like 
[dbdiagram](https://dbdiagram.io/) to achieve this. 

**IMPORTANT: Prefer using UUID as the primary key of your entity, since UUIDs can be generated from the app and your app
must be designed to scale, therefore using sequence, triggers etc to generate it will finally make the database your 
bottleneck. Additionally some databases don't have a native way to generate unique ids! 
[Read more](https://dzone.com/articles/uuid-as-primary-keys-how-to-do-it-right).**

### Design the domain classes

These classes are the keepers of the business/domain language. A colleague should look at them and understand 
immediately what these models are about! Every entity should have its model.

Use tools like [flyway](https://flywaydb.org/) or [liquibase](https://www.liquibase.org/) to ensure database versioning. 
Your database version must follow your code's version and the opposite. **DO NOT USE SCRIPTS LIVING OUTSIDE THE CODE**.

**IMPORTANT: All domain classes must implement hashCode and equals methods. 
[Read why](https://www.baeldung.com/jpa-entity-equality)**.

## Work on the repositories

From where you will get the data that you will populate your models with? Here you should connect with the DBs, 
other APIs, queues etc. You get the data and then adapt their results to DTOs (Data Transfer Objects), which you will 
adapt on your model classes later on.

### Write unit and integration tests for the repositories

Before continuing with writing the service level classes. You should write the tests for the repositories; unit and 
integration ones. You should be sure that your code you've written on step 5 is working as expected. If you don't check 
that, then afterwards you'll have to face it! And believe me if you have to debug and search all of your service to end 
to the repositories then this is time-consuming! Writing these tests will take you more time in the first tries, but 
eventually it will take less and less. And also you will have less time for debugging!

### Work on the services

The services are actually the heart of your business. They do all the hard work since they have the logic of your service. 
They get inputs from the controllers, communicate with repositories DAOs or other services, combine the data and 
implement the business requirement. In most of the cases the service layer methods must return the model

In case you integrate with a third party service to perform a write operation follow the following
pattern:
1. Pre flight transaction: Prepare your request and persist it, otherwise you will never know it happened in case 
something goes wrong during the integration (e.g. timeout, the service stopped).
2. In-flight: Perform the integration operation - typically this is over http
3. Post flight transaction: Capture the response and continue the operation actions.
Following the pattern above, you will be able to recover from any possible error and self-heal your service in certain
cases.

Design your business implementation taking care of [idempotence](https://en.wikipedia.org/wiki/Idempotence). There will 
be cases were your API will be invoked using exactly the same data because of an issue, or a timeout or... a human error.

**IMPORTANT: All transactional operations are taking place on this layer. It is important to use one single transaction 
when using ACID compliant databases, otherwise you will end up with performance issues. In case of non ACID compliant 
databases it is important to remember that each upsert operation is atomic therefore you have to take care of the 
rollback scenarios.**

### Write unit tests for the services

Test a lot and early in order to avoid many bugs in the end, which are also time-consuming.

### Work on the controllers / APIs

The controllers are the contracts of your service. Once created it will be difficult to change since anyone using these 
must adapt to the changes. Do not return the domain class even if it seems the obvious! Create DTOs for each case e.g. 
GetUserResponseDto, CreateUserRequestDto. DTOs are part of your API so semantically they should be different even if 
they seem to be similar.

### Write unit tests about the controllers
 
Tests again and again! For the same reason! 

### Write component tests
 
Test your API e2e. Send a request to your api and receive the response. Test if everything 
works as expected. Ideally, you should work with a different DB. The best way is to use docker containers here - have 
all the repos up in containers, populate them with the correct data and do your testing!

### Readiness, Liveness

Unless offered out-of-the-box (e.g. 
[Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.monitoring))
create a dummy API that reveals that your servie is up and running. Is is mandatory for platforms like kubernetes since
it tells to the orchestrator that the service us up and running, otherwise... boom it will kill it with fire.

### Supportability

Now that you finished, are you able to support your service when it goes to a production system? Take care of the 
following:
1. Distributed tracing. Imagine a platform running 50 services 3 instances each (150 services) and a request
touching 5 of them... Which was it? [Read here](https://microservices.io/patterns/observability/distributed-tracing.html)
2. Logging. Use debug logging to log input and output of all methods. Use info to deliver meaningful logging since this 
is what you will look at in case of an issue (debug won't be open at this point :D). Messages like "No user found" are 
not very helpful. Use data to enrich the message following the same pattern everywhere (imagine trying to use cat grep 
to spot a message). A message like "User [id=1234] not found" might be very helpful but check with security first.