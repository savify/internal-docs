# Architecture Decision Log

> An **architecture decision record (ADR)** is a document that captures an important architectural decision made along with its context and consequences.
> 
> An **architecture decision (AD)** is a software design choice that addresses a significant requirement.
> 
> An **architecture decision log (ADL)** is the collection of all ADRs created and maintained for a particular project (or organization).

In **Architecture Decision Log (ADL)** section we keep all the architecture decision that were agreed upon by the developers.
More information about ADL can be found [here](https://github.com/joelparkerhenderson/architecture-decision-record)

## Architecture Decision Record Template

Use the following template to create new ADR.

```
# ADR title

{style="narrow"}
Status:
: **Accepted|Declined|Pending**

Deciders:
: **Names & Surnames of deciders**

Date:
: **DD-MM-YYYY**

## Context and Problem Statement

Some description of context and problem.

## Decision drivers

- Architecture driver 1
- Architecture driver 2

## Considered Options

- Option 1
- Option 2

## Decision Outcome

**Chosen option:** "Option 1", because ...

### Positive Consequences

- Positive consequence 1
- Positive consequence 2

### Negative Consequences

- Negative consequence 1
- Negative consequence 2

## Pros and Cons of the Options

### Option 1

- Good, because ...
- Bad, because ...

### Option 2

- Good, because ...
- Bad, because ...
```

## Writerside custom template configuration

In Writerside you also have ability to create custom template with the structure provided above, so you won't need to 
copy/paste the same template multiple times. Below you can find an information how to configure it.

![Create an ADR with custom template](create-topic-with-custom-template.png)

![Templates customization](customize-templates.png)