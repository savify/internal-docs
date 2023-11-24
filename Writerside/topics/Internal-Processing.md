# Internal Processing

The main principle of this system is that you can change its state only by calling a specific Command.

Commands can be called not only by the API, but by the processing module itself. The main use case which implements this
mechanism is data processing in eventual consistency mode when we want to process something in a different process and 
transaction. This applies, for example, to Inbox processing because we want to do something (calling a Command) based on
an Integration Event from the Inbox.

Implementation of internal processing is very similar to implementation of the Outbox and Inbox. One SQL table and one 
background worker for processing. Each internally processing Command must inherit from InternalCommandBase class:

```C#
public abstract class InternalCommandBase : ICommand
{
    public Guid Id { get; }

    public Guid CorrelationId { get; }

    protected InternalCommandBase(Guid id, Guid correlationId)
    {
        Id = id;
        CorrelationId = correlationId;
    }
}
```

This is important because the UnitOfWorkCommandHandlerDecorator must mark an internal Command as processed during committing
as was told **[here](Request-Processing.md#unit-of-work)**. 