# Domain Events Dispatching

Domain Events are side effects that come from Aggregates and Entities during performing some action on it. With this approach
we can react on publishing Domain Event synchronously (with corresponding Handler) or asynchronously by mapping it on 
Notification and sending it to Outbox.

As was mentioned in **[Request Processing](Request-Processing.md#unit-of-work)**, Domain Events are dispatched within 
Unit of Work. 

![Domain Events Dispatching](domain-events-dispatching.png)

```C#
public class DomainEventsDispatcher<TContext> : IDomainEventsDispatcher<TContext> where TContext : DbContext
{
    private readonly IMediator _mediator;

    private readonly IExecutionContextAccessor _executionContextAccessor;

    private readonly IOutbox<TContext> _outbox;

    private readonly IDomainEventsAccessor<TContext> _domainEventsAccessor;

    private readonly IDomainNotificationsMapper<TContext> _domainNotificationsMapper;

    public DomainEventsDispatcher(
        IMediator mediator,
        IExecutionContextAccessor executionContextAccessor,
        IOutbox<TContext> outbox,
        IDomainEventsAccessor<TContext> domainEventsAccessor,
        IDomainNotificationsMapper<TContext> domainNotificationsMapper)
    {
        _mediator = mediator;
        _executionContextAccessor = executionContextAccessor;
        _outbox = outbox;
        _domainEventsAccessor = domainEventsAccessor;
        _domainNotificationsMapper = domainNotificationsMapper;
    }

    public async Task DispatchEventsAsync()
    {
        var domainEvents = _domainEventsAccessor.GetAllDomainEvents();

        var domainEventNotifications = new List<IDomainEventNotification<IDomainEvent>>();
        foreach (var domainEvent in domainEvents)
        {
            Type? domainNotificationType = _domainNotificationsMapper.GetType(domainEvent.GetType().Name);
            object? domainNotification = null;

            if (domainNotificationType != null)
            {
                domainNotification = Activator.CreateInstance(
                    domainNotificationType,
                    domainEvent.Id,
                    _executionContextAccessor.CorrelationId,
                    domainEvent);
            }

            if (domainNotification != null)
            {
                domainEventNotifications.Add(domainNotification as IDomainEventNotification<IDomainEvent> ?? throw new InvalidOperationException());
            }
        }

        _domainEventsAccessor.ClearAllDomainEvents();

        foreach (var domainEvent in domainEvents)
        {
            await _mediator.Publish(domainEvent).ContinueWith(async _ =>
            {
                await this.DispatchEventsAsync();
            });
        }

        foreach (var domainEventNotification in domainEventNotifications)
        {
            var type = _domainNotificationsMapper.GetName(domainEventNotification.GetType());
            var data = JsonConvert.SerializeObject(domainEventNotification, new JsonSerializerSettings
            {
                ContractResolver = new AllPropertiesContractResolver()
            });

            var outboxMessage = new OutboxMessage(
                domainEventNotification.Id,
                domainEventNotification.DomainEvent.OccurredOn,
                type!,
                data);

            _outbox.Add(outboxMessage);
        }
    }
}
```
{collapsible="true" default-state="collapsed" collapsed-title="DomainEventsDispatcher"}

Firstly, it gets all Domain Events from all persisted Entities. `DomainEventsAccessor` is used for this purpose:

```C#
public class DomainEventsAccessor<TContext> : IDomainEventsAccessor<TContext> where TContext : DbContext
{
    private readonly TContext _context;

    public DomainEventsAccessor(TContext context)
    {
        _context = context;
    }

    public IReadOnlyCollection<IDomainEvent> GetAllDomainEvents()
    {
        var domainEntities = _context.ChangeTracker
            .Entries<Entity>()
            .Where(x => x.Entity.DomainEvents != null && x.Entity.DomainEvents.Any()).ToList();

        return domainEntities
            .SelectMany(x => x.Entity.DomainEvents!)
            .ToList();
    }

    public void ClearAllDomainEvents()
    {
        var domainEntities = _context.ChangeTracker
            .Entries<Entity>()
            .Where(x => x.Entity.DomainEvents != null && x.Entity.DomainEvents.Any()).ToList();

        domainEntities.ForEach(entity => entity.Entity.ClearDomainEvents());
    }
}
```
{collapsible="true" default-state="collapsed" collapsed-title="DomainEventsAccessor"}

The next step is to check if there are any `Notifications` that are mapped to found Domain Events. If yes, that `Notifications`
will be created from Domain Events.

Then, all Domain Events will be cleared to ensure that there will not be any conflicts, and those that were get will be
dispatched.

> Note: if during dispatching Domain Event any other Domain Event will be added, it will also be dispatched in the same 
> transaction scope.

After that, Notifications that were created will be saved to Outbox.

## Domain Notifications Mapping

To map Domain Event to Notification that will be processed through Outbox, next steps should be perfored:

1. Create Domain Event Notification - this class should extend `DomainEventNotificationBase<>`

    ```C#
    public class NewUserRegisteredNotification : DomainEventNotificationBase<NewUserRegisteredDomainEvent>
    {
        [JsonConstructor]
        public NewUserRegisteredNotification(Guid id, Guid correlationId, NewUserRegisteredDomainEvent domainEvent) : base(id, correlationId, domainEvent)
        {
        }
    }
    ```
   
2. To map this notification to actual Domain Event, you should register it in `App.Modules.[ModuleName].Infrastructure.Outbox.DomainNotificationsMap`:

    ```C#
    internal static class DomainNotificationsMap
    {
        internal static BiDictionary<string, Type> Build()
        {
            var domainNotificationsMap = new BiDictionary<string, Type>();
    
            domainNotificationsMap.Add(nameof(NewUserRegisteredDomainEvent), typeof(NewUserRegisteredNotification));
    
            return domainNotificationsMap;
        }
    }
    ```