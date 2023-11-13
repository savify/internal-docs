# 0001. Dockerize a project

{style="narrow"}
Status:
: **Accepted**

Deciders:
: **Andrii Vozniuk, Dmytro Lynda**

Date:
: **21-03-2023**

## Context and Problem Statement

In order to have an ability to setup project on different development environments, to have a Continiuos Ingegration / 
Continious Delivery, we need to containerise our project. The most popular containerisation solution is 
[Docker](https://www.docker.com/).

## Decision drivers

- Easy deployment
- Scalability

## Considered Options

- Dockerize the whole application
- Dockerize only infrastructure services
- Do not use containerisation

## Decision Outcome

Chosen option: "Dockerize only infrastructure services", because for a starting stage we only need to have easy dev 
environments setup.

### Positive Consequences

- Easy configuration
- Easy development environment setup

### Negative Consequences

- Harder to configure CI/CD for .Net Core project

## Pros and Cons of the Options

### Dockerize the whole application

Create containers not only for infrastructure services (database, message broker etc.) but also a whole .Net application

- Good, because it allows us to deploy a project easily
- Good, because it allows us to implement additional features in CI more easily, such as running tests
- Bad, because the configuration is more complicated
- Bad, because running project locally will require additional steps (e.g. building the image for .Net application every
time before running the project)

### Dockerize only infrastructure services

Add docker containers only for infrastructure services such as database, message broker, smtp server.

- Good, because it's less complicated to configure

