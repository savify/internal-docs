# Request Processing

Each module has the same interface signature exposed to the API. It contains 3 methods: command with result, 
command without result and query.

```C#
public interface IUserAccessModule
{
    Task ExecuteCommandAsync(ICommand command);

    Task<TResult> ExecuteCommandAsync<TResult>(ICommand<TResult> command);

    Task<TResult> ExecuteQueryAsync<TResult>(IQuery<TResult> query);
}
```

## CQRS Concepts

Processing of Commands and Queries is separated by applying the architectural style/pattern 
**[Command Query Responsibility Segregation (CQRS)](https://docs.microsoft.com/en-us/azure/architecture/patterns/cqrs)**.

### Commands

Commands are processed using Write Model which is implemented using DDD tactical patterns and EntityFramework:

![Command Processing](command-processing.png)

```C#
internal class RegisterNewUserCommandHandler : ICommandHandler<RegisterNewUserCommand, Guid>
{
    private readonly IUserRegistrationRepository _userRegistrationRepository;

    private readonly IUsersCounter _usersCounter;

    public RegisterNewUserCommandHandler(IUserRegistrationRepository userRegistrationRepository, IUsersCounter usersCounter)
    {
        _userRegistrationRepository = userRegistrationRepository;
        _usersCounter = usersCounter;
    }

    public async Task<Guid> Handle(RegisterNewUserCommand command, CancellationToken cancellationToken)
    {
        var userRegistration = UserRegistration.RegisterNewUser(
            command.Email,
            PasswordHasher.HashPassword(command.Password),
            command.Name,
            Country.From(command.Country),
            Language.From(command.PreferredLanguage),
            ConfirmationCode.Generate(),
            _usersCounter);

        await _userRegistrationRepository.AddAsync(userRegistration);

        return userRegistration.Id.Value;
    }
}
```
{collapsible="true" default-state="collapsed" collapsed-title="CommandHandler Example"}

### Queries

Queries are processed using Read Model which is implemented by executing raw SQL statements on database views with Dapper:

![Query Processing](query-processing.png)

```C#
internal class GetUserRegistrationQueryHandler : IQueryHandler<GetUserRegistrationQuery, UserRegistrationDto?>
{
    private readonly ISqlConnectionFactory _sqlConnectionFactory;

    public GetUserRegistrationQueryHandler(ISqlConnectionFactory sqlConnectionFactory)
    {
        _sqlConnectionFactory = sqlConnectionFactory;
    }

    public async Task<UserRegistrationDto?> Handle(GetUserRegistrationQuery query, CancellationToken cancellationToken)
    {
        var connection = _sqlConnectionFactory.GetOpenConnection();

        var sql = $"""
                   SELECT id, email, name, status, valid_till as validTill
                   FROM {DatabaseConfiguration.Schema.Name}.user_registrations
                   WHERE id = @UserRegistrationId
                   """;

        return await connection.QuerySingleOrDefaultAsync<UserRegistrationDto>(sql, new { query.UserRegistrationId });
    }
}
```
{collapsible="true" default-state="collapsed" collapsed-title="QueryHandler Example"}

## Cross-Cutting Concerns

The implementation of cross-cutting concerns is done using the **[Decorator Pattern](https://en.wikipedia.org/wiki/Decorator_pattern)**. 
Each Command Handler is decorated by 4 decorators: business rule exception localization, logging, validation and unit of work.

![Cross-cutting concerns](cross-cutting-concerns.png)

### Business Rules Exception Localization

This decorator localizes exception messages that comes from Domain Business
Rules.

```C#
internal class BusinessRuleExceptionLocalizationCommandHandlerDecorator<T, TResult> : ICommandHandler<T, TResult> where T : ICommand<TResult>
{
    private readonly ICommandHandler<T, TResult> _decorated;

    private readonly IStringLocalizer _localizer;

    public BusinessRuleExceptionLocalizationCommandHandlerDecorator(
        ICommandHandler<T, TResult> decorated,
        ILocalizerProvider localizerProvider)
    {
        _decorated = decorated;
        _localizer = localizerProvider.GetLocalizer();
    }

    public async Task<TResult> Handle(T command, CancellationToken cancellationToken)
    {
        try
        {
            return await _decorated.Handle(command, cancellationToken);
        }
        catch (BusinessRuleValidationException exception)
        {
            var localizedMessage = _localizer[
                exception.BrokenRule.MessageTemplate,
                exception.BrokenRule.MessageArguments ?? Array.Empty<object>()
            ];

            throw new LocalizedBusinessRuleValidationException(exception.BrokenRule, localizedMessage);
        }
    }
}
```
{collapsible="true" default-state="collapsed" collapsed-title="BusinessRuleExceptionLocalizationCommandHandlerDecorator"}

### Logging

Logging decorator logs execution, arguments and processing of each Command. This way each log inside a processor has the log context of the processing command.

```C#
internal class LoggingCommandHandlerDecorator<T, TResult> : ICommandHandler<T, TResult> where T : ICommand<TResult>
{
    private readonly ICommandHandler<T, TResult> _decorated;

    private readonly ILogger _logger;

    private readonly IExecutionContextAccessor _executionContextAccessor;

    public LoggingCommandHandlerDecorator(
        ICommandHandler<T, TResult> decorated,
        ILoggerProvider loggerProvider,
        IExecutionContextAccessor executionContextAccessor)
    {
        _decorated = decorated;
        _logger = loggerProvider.GetLogger();
        _executionContextAccessor = executionContextAccessor;
    }

    public async Task<TResult> Handle(T command, CancellationToken cancellationToken)
    {
        using (LogContext.Push(
                   new RequestLogEnricher(_executionContextAccessor),
                   new CommandLogEnricher(command)))
        {
            try
            {
                _logger.Information("Executing command {@Command}", command);

                var result = await _decorated.Handle(command, cancellationToken);

                _logger.Information("Command processed successful, result {Result}", result);

                return result;
            }
            catch (Exception exception)
            {
                _logger.Error(exception, "Command processing failed; {Message}", exception.Message);
                throw;
            }
        }
    }

    private class CommandLogEnricher : ILogEventEnricher
    {
        private readonly ICommand<TResult> _command;

        public CommandLogEnricher(ICommand<TResult> command)
        {
            _command = command;
        }

        public void Enrich(LogEvent logEvent, ILogEventPropertyFactory propertyFactory)
        {
            logEvent.AddOrUpdateProperty(new LogEventProperty("Context", new ScalarValue($"Command:{_command.Id.ToString()}")));
        }
    }
}
```
{collapsible="true" default-state="collapsed" collapsed-title="LoggingCommandHandlerDecorator"}

### Validation

The Validation decorator performs Command data validation. It checks rules against Command arguments using the 
FluentValidation library.

```C#
internal class ValidationCommandHandlerDecorator<T, TResult> : ICommandHandler<T, TResult> where T : ICommand<TResult>
{
    private readonly ICommandHandler<T, TResult> _decorated;

    private readonly IEnumerable<IValidator<T>> _validators;

    private readonly IStringLocalizer _localizer;

    public ValidationCommandHandlerDecorator(
        ICommandHandler<T, TResult> decorated,
        IEnumerable<IValidator<T>> validators,
        ILocalizerProvider localizerProvider)
    {
        _decorated = decorated;
        _validators = validators;
        _localizer = localizerProvider.GetLocalizer();
    }

    public async Task<TResult> Handle(T command, CancellationToken cancellationToken)
    {
        Validate(command, _validators, _localizer);

        return await _decorated.Handle(command, cancellationToken);
    }
}

internal static class ValidationCommandHandlerDecorator
{
    internal static void Validate<T>(T command, IEnumerable<IValidator<T>> validators, IStringLocalizer localizer)
    {
        var errors = validators
            .Select(v => v.Validate(command))
            .SelectMany(result => result.Errors)
            .Where(error => error != null)
            .ToList();

        if (errors.Any())
        {
            var errorList = new Dictionary<string, List<string>>();

            foreach (var error in errors)
            {
                var fieldErrors = new List<string>();

                if (errorList.ContainsKey(error.PropertyName))
                {
                    fieldErrors = errorList[error.PropertyName];
                }

                fieldErrors.Add(localizer[error.ErrorMessage]);
                errorList[error.PropertyName] = fieldErrors;
            }

            throw new InvalidCommandException(errorList);
        }
    }
}
```
{collapsible="true" default-state="collapsed" collapsed-title="ValidationHandlerDecorator"}

### Unit of Work

All Command processing has side effects. To avoid calling commit on every handler, UnitOfWorkCommandHandlerDecorator is 
used. It additionally marks InternalCommand as processed (if it is Internal Command) and dispatches all Domain Events 
(as part of Unit Of Work).

```C#
internal class UnitOfWorkCommandHandlerDecorator<T> : ICommandHandler<T> where T : ICommand
{
    private readonly ICommandHandler<T> _decorated;
    private readonly IUnitOfWork<UserAccessContext> _unitOfWork;
    private readonly UserAccessContext _userAccessContext;

    public UnitOfWorkCommandHandlerDecorator(
        ICommandHandler<T> decorated,
        IUnitOfWork<UserAccessContext> unitOfWork,
        UserAccessContext userAccessContext)
    {
        _decorated = decorated;
        _unitOfWork = unitOfWork;
        _userAccessContext = userAccessContext;
    }

    public async Task Handle(T command, CancellationToken cancellationToken)
    {
        await _decorated.Handle(command, cancellationToken);

        if (command is InternalCommandBase)
        {
            await SetInternalCommandAsProcessedAsync(_userAccessContext, command.Id, cancellationToken);
        }

        await _unitOfWork.CommitAsync(cancellationToken);
    }
}

internal static class UnitOfWorkCommandHandlerDecorator
{
    internal static async Task SetInternalCommandAsProcessedAsync(UserAccessContext userAccessContext, Guid commandId, CancellationToken cancellationToken)
    {
        var internalCommand = await userAccessContext.InternalCommands.SingleOrDefaultAsync(
            x => x.Id == commandId,
            cancellationToken: cancellationToken);

        if (internalCommand != null)
        {
            internalCommand.ProcessedDate = DateTime.UtcNow;
        }
    }
}
```
{collapsible="true" default-state="collapsed" collapsed-title="UnitOfWorkHandlerDecorator"}
