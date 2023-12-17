# 0002. Communication between modules

{style="narrow"}
Status:
: **Accepted**

Deciders:
: **Andrii Vozniuk, Dmytro Lynda**

Date:
: **17-12-2023**

## Context and Problem Statement

We should reduce coupling between modules, however we cannot eliminate it completely - modules still should communicate
with each other. We need to decide which way of communication we should use in our project.

## Decision drivers

- Low Coupling
- Data consistency
- Communication complexity

## Considered Options

- Event-driven (asynchronous)
- Mixed Event-driven and synchronous queries

## Decision Outcome

**Chosen option:** "Event-driven (asynchronous)", because introducing sync queries will "blur" the boundaries between modules.

### Positive Consequences

- Lower coupling
- Modules autonomy

### Negative Consequences

- More complex communication
- Immediate consistency not supported 

## Pros and Cons of the Options

### Event-driven (asynchronous)

Each module will publish integration events. Other modules will subscribe to these events. No synchronous communication
allowed.

- Good, because it provides lower coupling between modules
- Good, because it introduces more modules autonomy
- Bad, because it increases communication implementation complexity
- Bad, because it does not support immediate data consistency

### Mixed Event-driven and synchronous queries

- Good, because it has benefits of both sync and async communication:
  - Supports immediate data consistency (sync)
  - Reduces coupling (async)
- Good, because for some solutions synchronous query will allow simpler communications solutions
- Bad, because synchronous queries increases coupling between modules
- Bad, because it decreases modules autonomy

