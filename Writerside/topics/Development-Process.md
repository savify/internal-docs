# Development Process

On this page you will find instructions and tips how to develop and maintain backend application.

## Jira

We use **[Jira](https://savify.atlassian.net/jira/software/projects/SAV/boards/1)** to keep track on all tasks and issues.

Below you can find actual workflow:

![Jira Workflow](jira-workflow.png)

1. **TODO**: issue is refined and ready for development.
2. **In progress**: developer works on this issue.
3. **Code review**: code part is finished and Pull Request was created. PR is waiting for other developers approvals.
4. **Done**: issue is completed.

There are several types of tasks that we use:

- **Epic** - describes a big feature or module that we want to deliver. Epics are used also for keeping track of tasks
related to specific topic (e.g. Infrastructure, UI/UX).
- **Story** - describes a small feature that delivers a business value. Must be a part of *Epic*.
- **Task** - describes an issue that not necessarily should deliver a business value, but can contain some refactor, cleanup
or improvements to existing codebase.
- **Bug** - obviously, describes bug that should be fixed.
- **Sub-Task** - used when a *Story*, *Task* or *Bug* scope is pretty big, and it should be split into smaller issues.

## Git repository

We use Git as our version control system. Repository can be found **[here](https://github.com/savify/savify)**.

For every issue that requires code modification in repository we create a new branch. We **DO NOT** push changes directly
to `main` branch - use Pull Requests instead. 

All branches, commits and pull request should follow the [Conventional Commits](#conventional-commits) rules.

### Conventional Commits

All commits, branches and pull requests should be named according to **[Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)** specifications.

#### Example:

- You have a task named **SAV-1 "Initialize User Access module"**
- You create a branch with name `feature/SAV-1-user-access-module-init`
- Commit messages examples:
  - `feat(SAV-1): add domain project`
  - `chore(SAV-1): add application project`
  - `chore(SAV-1): add infrastructure project`
  - `test(SAV-1): add unit and integration tests`
  - `refactor(SAV-1): module cleanup`
- Pull request from `feature/SAV-1-user-access-module-init` into `main` branch: `feat(SAV-1): user access module initialization`

> **When merging pull request remember about squashing commits from your branch.**
{style="warning"}

### Code standards

We use `.editorconfig` to ensure the same code standards on all local environments. Make sure you have `dotnet-format`
tool installed; Run:
```
dotnet tool install -g dotnet-format --version "7.*" --add-source https://pkgs.dev.azure.com/dnceng/public/_packaging/dotnet7/nuget/v3/index.json
```
to install it. Before each commit run `make codestyle-fix` to align all changed files to .editorconfig settings.
