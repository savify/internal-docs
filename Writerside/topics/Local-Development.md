# How to run project locally

In order to start development process, you need to configure your local environment. Follow instruction on this page to 
setup it.

## Requirements

- **[Docker](https://docker.com/)** and **[docker-compose](https://docs.docker.com/compose/install/)**. They both can be 
installed by one tool - **[Docker Desktop](https://www.docker.com/products/docker-desktop/)**
- **[.Net CLI](https://learn.microsoft.com/en-us/dotnet/core/tools/)** and **[EF Tools](https://learn.microsoft.com/en-us/ef/core/cli/dotnet)**

## How to run

Follow next steps to run backend application locally

1. Clone Git repository:
    ```bash
    git clone git@github.com:savify/savify.git
    ```
2. Copy `.env.dist` file to `.env`
3. Run docker containers:
    ```bash
    make up
    ```
4. Run database migrations:
    ```bash
    make db-update
    ```
5. Seed database with initial data:
    ```bash
    make seed-database
    ```
6. For making API calls to external providers, you should set all required [secrets](https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets?view=aspnetcore-7.0&tabs=linux) for a project. Run
    ```bash
   dotnet user-secrets set "SaltEdge:AppId" "**AppId**" --project src/API
   dotnet user-secrets set "SaltEdge:AppSecret" "**AppSecret**" --project src/API
   ```
7. Build and run project (API)

> To get TPP application secrets ask **Andrii Vozniuk**
{style="note"}

## Available hosts

After running the application you can use the following links to access the application:

- [API - http://localhost:8080/](http://localhost:8080/) - backend application API entrypoint .
- [Swagger](http://localhost:8080/swagger/index.html) - API documentation (auto-generated).
- [Mailcatcher - http://localhost:8888/](http://localhost:8888/) - here you can find all emails that were caught during
application execution.
- [Kibana - http://localhost:5601/](http://localhost:5601/) - Data analytics and logs visualisation tool.
- [Adminer - http://localhost:8880/](http://localhost:8880/) - you can use it as database UI if you don't have any installed.
- [Webhook Catcher - webhook.site](https://webhook.site/#!/eaf8199c-24b6-4d29-8f24-2781def88187/) - for now SaltEdge Webhooks 
are pointed to this tool.