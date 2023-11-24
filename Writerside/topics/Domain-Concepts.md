# Domain Model Principles

The Domain Model, which is the central and most critical part in the system, should be designed with special attention. 
Here are some key principles and attributes which are applied to Domain Models of each module:

1. **High level of encapsulation**

    All members are private by default, then internal - only public at the very edge.

2. **High level of PI (Persistence Ignorance)**

    No dependencies to infrastructure, databases, etc. All classes are POCOs.

3. **Rich in behavior**

    All business logic is located in the Domain Model. No leaks to the application layer or elsewhere.

4. **Low level of Primitive Obsession**

    Primitive attributes of Entites grouped together using ValueObjects.

5. **Business language**

    All classes, methods and other members are named in business language used in this Bounded Context.

6. **Testable**

    The Domain Model is a critical part of the system so it should be easy to test (Testable Design).

## Entities & Aggregate Roots

Each Aggregate should have its AggregateRoot - an entrypoint to the Aggregate. We communicate with business logic by calling
methods only on Aggregate Root. Every Aggregate Root should have an `IAggregateRoot` interface.

Each Entity should extend `Entity` abstract class. This class contains basic methods that help to operate an Aggregate, 
e.g. for checking Business Rules and managing its Domain Events.

Each Entity should have its own Id representation, that should extend `TypedIdValueBase`.
