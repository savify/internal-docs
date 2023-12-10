# Domain Model

**Savify Application** is a comprehensive financial management solution that aims to help individuals effectively
manage their incomes, expenses, budgets, investments, and debts. The application provides a range of features
including budget planning, investment analysis, synchronisation with bank accounts, debt management, goal setting,
statistics analysis, insights, quizzes, and challenges. This document outlines the various flows, details, and
business rules associated with the application.

## 1. Definitions

> TPP - third-party provider. External service that is integrated by Savify. Provides data about accounts, balances and
> transactions for users that connected their bank with account.

## 2. Authentication & Authorization

The main business entity is a `User`. To be a `User`, `User Registration` is required and should be confirmed. `User`
can be registered by providing basic information - name, email and password. Additionally, `User` can be registered with
Apple or Google authentication.

`User`'s email address should be unique within the application.

During `User Registration` with email/password, `Confirmation Code` should be generated and sent to provided email address.
`User Registration` should be confirmed using sent `Confirmation Code` (by typing it or by scanning QR code).

`Confirmation Code` has a validity period (e.g. 10mins). After `Confirmation Code` expiration `User` should request new
one to be able to process `User Registration` process.

When `User` forgot his password, he can reset it. Email with `Confirmation Code` should be sent to `User`, and after
confirming password reset action, `User` can set new password.

Each `User` has assigned one or more `User Role`. Base `User Roles` are `Administrator` and `User`, but role list can
be extended in the future.

Each `User Role` has a set of `User Permission`. `User Permission` defines whether `User` can invoke a particular action.

`User` can authenticate to the application using registered email/password or one of other authentication methods
(Google/Apple). For authentication JTW Token is used.

Each `User` has right to delete his application account with all data related to this account. Due to GDPR requirements,
after `User`'s account deletion Savify can store only statistical data, all the personal data should be soft/hard deleted.

## 3. Accounts

### 3.1. General

Each `User` can have one or more `Accounts`. Each `Account` has various of properties, including:

- Title
- `Account Type`
- `Currency`
- Balance
- is included in total balance
- view metadata
    - icon
    - color

`Account Type` can be one of: `Cash`, `Debit`, `Credit`, `Investment`, `Crypto`.

`Account Types` specifications:

1. `Cash`
    - does not have ability to integrate with bank

2. `Debit`
    - is a standard account type with standard behavior
    - has IBAN/BBAN field

3. `Credit`
    - instead of balance hase two properties - credit limit and available balance. Credit limit can not be accessed through
      TPP, so it should be set manually by user.
    - optionally, `User` can set credit card billing cycle, so app will notify about needed paybacks

4. `Investment`
    - have a list of stocks/indexes/ETF's that `User` invested in. Each position has its symbol, purchase date and quantity.
    - data about current price will be fetched through financial data TPP.

5. `Crypto`
    -  have a list of cryptocurrencies. Each position has its symbol, purchase date and purchase price.

`User` can add new `Accounts`, modify existing `Accounts`, modify balances (only for `Cash`, `Debit` and `Credit` `Accounts` that are
not connected to the bank), hide `Accounts` and delete them.

### 3.2. Connecting accounts to banks

`Users` can connect their bank accounts to `Accounts` with `Debit` or `Credit` account types. Each `Account` can have only
one bank connection.

`User` cannot modify balance, IBAN/BBAN, currency on`Accounts` with bank connection. All this data will be fetched automatically
through **TPP**.

`User` can disconnect his bank from `Account` at any time.

When bank is connected to `Account`, balances and transactions will be fetched automatically when user accesses the application.

## 4. Transactions

Each `Account` has a list of `Transactions`. `Transaction` has one of types: `Expense`, `Income`, `Transfer`, `Debt` and `Goal`.

`Transaction` has information about source, destination, date, amount and various of information including description and
tags. `Transaction` can be connected to other `Transaction` to ensure that it was `Transfer` between different `Accounts`.
`Transaction` can be splitted, so the part of it can be treated as `Debt`.

### 4.1. Incomes

`Transaction` that is coming from somewhere to `User`'s `Account` is treated as `Income`. Income have several categories.
It can be added manually or automatically with bank connection mechanism.

### 4.2. Expenses

`Transaction` that is coming from `User`'s `Account` to somewhere is treated as `Expense`. Expense have several categories.
It can be added manually or automatically with bank connection mechanism.

### 4.2. Transfer

`Transaction` that is coming form `User`'s `Account` to another `User`'s `Account` is treated as `Transfer`. It can be added
manually, automatically with bank connection mechanism (identify `Transfer` by IBAN/BBAN), or by connecting detected `Expense`
and `Income` and identifying them as transfers between accounts.

### 4.3. Debt

`Debt` can be created when `User` don't want to identify `Transaction` as `Expense` or `Income`. With `Debt` mechanism,
transaction can be treated as `User`'s or someone else's debt. `Debt` can be added manually or automatically identified
from transaction that was performed from `User`'s `Account` to IBAN that is saved in system and is identifying as e.g.
`User`'s friend. More about `Debts` is described in [7. Debt Management](#7-debt-management) section.

### 4.4. Goal

`Goal` is a "transaction" that can be created from `Transfer` to `Account` that has `Goal` set up. `User` can set amount
that will be identified as a `Goal`, so the certain amount from `Account` with a `Goal` will be "frozen". More about `Goals`
is described in [8. Goals](#8-goals) section.

## 5. Categories

Each `Transaction` can have a `Category`. There are 9 main `Categories`, each of main categories has a set of subcategories.
`Categories` can be set automatically by categorization mechanisms or manually by `User`. `User` has ability to change
categories that were defined automatically.

## 6. Budget Planning

`User` has ability to plan his budget for each `Expense` `Category` or for overall spending. `Budgets` can be set for a
specific time period (e.g. monthly, yearly). Savify provides visual representations to track budget progress and send
notifications when nearing or exceeding budget limits.

## 7. Debt Management

`User` can track debts owed to and by friends. Savify provides options for `Users` to record debt transactions and send
reminders for repayment. `Debt` can be added manually or automatically identified from transaction that was performed
from `User`'s `Account` to IBAN that is saved in system and is identifying as e.g. `User`'s friend.

## 8. Goals

`User` can set financial `Goals` such as saving for a vacation, purchasing a car, or paying off a debt. `Goals` can have
target amounts, deadlines, and tracking mechanisms to monitor progress. Savify should provide visual representations
and notifications to keep users motivated.

## 9. Statistics Analysis

`User` can access detailed statistics and reports on income, expenses, investments, goals and debts. Savify
should provide insights and visualizations to help users understand their financial patterns and make informed decisions.

## 10. Insights, Quizzes and Challenges

`User` can read curated financial insights and articles provided by the application. Insights can cover topics such as
budgeting tips, expenses/incomes overview, investment strategies, and debt management advice.

`User` can participate in quizzes and challenges related to his personal finance. The application should provide rewards
or incentives for completing quizzes and challenges.

## 11. User Support and Feedback

`User` should have access to support channels to resolve issues, provide feedback, and suggest improvements. The application
should provide documentation and tutorials to help users understand and utilize its features effectively.
