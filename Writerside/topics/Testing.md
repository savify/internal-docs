# Testing

We are trying our best to cover this application with several types of tests. Besides of that, we also test our features
manually: using Postman and directly from the mobile application.

## Manual testing

For testing our API manually we use **[Postman](https://www.postman.com)** and Postman collection that is shared between
our team in **Savify Workspace**. Ask your teammates to invite you to this workspace.

![Postman](postman.png)

## Automation tests

In backend project we use the next types of automation tests.

### Unit tests

> A **unit test** is an automated piece of code that invokes the unit of work being tested, and then checks some assumptions
> about a single end result of that unit. A unit test is almost always written using a unit testing framework.
> It can be written easily and runs quickly. It’s trustworthy, readable, and maintainable. It’s consistent in
> its results as long as production code has not changed.
> 
> **[Art of Unit Testing 2nd Edition](https://www.manning.com/books/the-art-of-unit-testing-second-edition)** by Roy Osherove

Each module should have `Tests/UnitTests` project. These projects have [NUnit](https://nunit.org/) and 
[NSubstitute](https://nsubstitute.github.io/) dependencies.

Each unit tests should extend base class `App.BuildingBlocks.Tests.UnitTests.UnitTestBase`. Base test class contains
additional assertions and methods that help to test domain logic properly, such as domain events or business rule assertions.

Each test can have 2 possible outcomes:
* Action on Aggregate executed successfully and `DomainEvent` was published.
* Business rule was broken during executing an action on Aggregate.

### Integration tests

> **Integration test** can have multiple definitions as it can test different types of integrations. For this project its
> definition is next: it verifies how system works in integration with "out-of-process" dependencies (database, 
> messaging system, file system or external API) and tests a particular use case.
> 
> See **[Article about integration tests](https://martinfowler.com/bliki/IntegrationTest.html)** by Martin Fowler.

We **DON'T** mock dependencies on which we have full control (e.g. database). And we mock dependencies that we don't have
control on (such as external API or SMTP server).

To run integration tests:
1. Run migrations on test database to ensure database schemas:
    ```bash
    make test-db-update
    ```
2. Ensure that `TestSettings.runsettings` configuration file is used during running integration tests.

Each module has its own `TestBase` class that contains basic logic for preparing tests and tearing down after execution.

During one-time set up the next steps are executed:
1. Getting database connection string from environment
2. Creating custom `WebApplicationFactory` that mocks or replace some services.
3. Creating module that will be used for executing commands and queries.
4. Initiating [WireMock](https://github.com/WireMock-Net/WireMock.Net) for mocking external API requests.

Before each test the whole database is cleaned up to ensure that every test will have clear system.

### System integration tests

> **[System Integration Testing](https://en.wikipedia.org/wiki/System_integration_testing)** is performed to verify the 
> interactions between the modules of a software system. It involves the overall testing of a complete system of many 
> subsystem components or elements.

When we are trying to test an integration between modules we are dealing with asynchronous communication. Due to 
asynchrony, our test must wait for the result at certain times. For this purpose the Sampling technique is used.

> An asynchronous test must wait for success and use timeouts to detect failure. This implies that every tested activity
> must have an observable effect: a test must affect the system so that its observable state becomes different. This 
> sounds obvious, but it drives how we think about writing asynchronous tests. If an activity has no observable effect, 
> there is nothing the test can wait for, and therefore no way for the test to synchronize with the system it is testing.
> There are two ways a test can observe the system: by sampling its observable state or by listening for events that it
> sends out.
> 
> **[Growing Object-Oriented Software, Guided by Tests](https://www.amazon.com/Growing-Object-Oriented-Software-Guided-Tests/dp/0321503627)** 
> by Steve Freeman

The implementation can be found in `App.IntegrationTests.SeedWork.TestBase`.

### Architecture tests

To ensure that architecture within particular modules and in the whole project is kept, architecture test were written.
Basically they check dependencies between modules, its layers, and ensuring some naming standards and encapsulation. 

Each module has its own architecture tests, but we also have global tests that check dependencies within whole application.

You can also check the article about **[Architecture Tests](https://blogs.oracle.com/javamagazine/post/unit-test-your-architecture-with-archunit)**.