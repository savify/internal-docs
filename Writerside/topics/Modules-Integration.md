# Modules Integration

Integration between modules is strictly asynchronous using **Integration Events** and the InMemory Event Bus as broker. 
In this way coupling between modules is minimal and exists only on the structure of Integration Events.

> **[Update: 2023-11-24]** For easy synchronous communication between modules some **Integration Query** mechanism may be introduced.  
{style="warning"}

Modules don't share data so it is not possible nor desirable to create a transaction which spans more than one module. 
To ensure maximum reliability, the **[Outbox / Inbox pattern](http://www.kamilgrzybek.com/design/the-outbox-pattern/)** 
is used. This pattern provides accordingly "At-Least-Once delivery" and "At-Least-Once processing".

![Modules Communication](modules-communication.png)

The Outbox and Inbox is implemented using two SQL tables and a background worker for each module. The background worker 
is implemented using the Quartz.NET library.

## Outbox

Outbox message is created during Unit Of Work domain events dispatching when `Notification` exists for particular Domain 
Event. 

Then the background worker `ProcessOutboxJob` will read messages from database, map them back on notifications and publish them.

If notification processing will not succeed, worker will publish it more than once until it will not be processed.

![Outbox Messages Processing](outbox-messages-processing.png)

## Inbox

When `IntegrationEvent` will be sent to another module, it will be handled by `IntegrationEventGenericHandler`. This handler
will save integration event to the inbox.

Then the background worker `ProcessInboxJob` will read messages from database, map them back on events and publish them.

If event processing will not succeed, worker will publish it more than once until it will not be processed.

![Inbox Messages Processing](inbox-messages-processing.png)

## CorrelationId

During request processing `CorrelationId` will be applied to domain event notifications, internal commands, and integration 
events to have ability to track down the request that will go through multiple modules.