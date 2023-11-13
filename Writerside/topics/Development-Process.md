# Development Process

On this page you will find instructions and tips how to develop and maintain backend application.

## Conventional Commits

All commits, branches and pull requests should be named according to [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) specifications.

### Example:

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