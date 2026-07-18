# Security Standards

## Purpose

This document defines security standards and practices that must be applied across
all projects, technologies, and layers in this repository. Security is a
cross-cutting concern and must be considered from the first line of code.

---

## Guidelines

### Authentication and Authorisation

- Use industry-standard protocols: OAuth 2.0, OpenID Connect, JWT.
- Never implement custom authentication mechanisms.
- Always validate tokens on the server — never trust client-supplied identity.
- Use role-based or policy-based authorisation.
- Apply the principle of least privilege to every user, service, and process.
- Never store passwords in plain text — always hash with bcrypt, Argon2, or PBKDF2.
- Use short-lived access tokens combined with refresh tokens.

### Secrets Management

- Never hardcode secrets, API keys, connection strings, or credentials in source code.
- Never commit secrets to version control — not even in development branches.
- Store secrets in environment variables, Azure Key Vault, AWS Secrets Manager, or a comparable service.
- Use Managed Identity on Azure to eliminate credential-based secret access entirely.
- Rotate secrets regularly and revoke compromised ones immediately.
- Scan commits for accidentally exposed secrets before merging.

```csharp
// Good — read from configuration / environment
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

// Bad — hardcoded
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer("Server=prod-server;******"));
```

### Input Validation

- Validate all input at every entry point — API endpoints, message consumers, file uploads.
- Use a validation library (FluentValidation for .NET, Zod/Yup for TypeScript).
- Reject unexpected input rather than sanitising it.
- Validate data types, lengths, formats, and ranges.
- Never trust user-supplied data for SQL, HTML, file paths, or shell commands.

### SQL Injection Prevention

- Always use parameterised queries or ORMs (EF Core).
- Never concatenate user input into SQL strings.
- If raw SQL is required, use `FromSqlRaw` with parameters, never string interpolation.

```csharp
// Good — parameterised
var orders = await _dbContext.Orders
    .Where(o => o.CustomerId == customerId)
    .ToListAsync(cancellationToken);

// Good — raw SQL with parameters
var orders = await _dbContext.Orders
    .FromSqlRaw("SELECT * FROM Orders WHERE CustomerId = {0}", customerId)
    .ToListAsync(cancellationToken);

// Bad — string interpolation in SQL
var orders = await _dbContext.Orders
    .FromSqlRaw($"SELECT * FROM Orders WHERE CustomerId = '{customerId}'")
    .ToListAsync(cancellationToken);
```

### Cross-Site Scripting (XSS) Prevention

- Encode all user-supplied content before rendering in HTML.
- Use frameworks that automatically encode output (Razor, Angular, React).
- Avoid `innerHTML`, `dangerouslySetInnerHTML`, and `bypassSecurityTrustHtml` unless absolutely necessary and explicitly sanitised.
- Apply a Content Security Policy (CSP) header.

### Cross-Site Request Forgery (CSRF) Prevention

- Use anti-forgery tokens for state-changing operations in web applications.
- For REST APIs using JWT with `Authorization` headers, CSRF is not typically required.
- Do not store tokens in cookies without `SameSite=Strict` or `SameSite=Lax`.

### Transport Security

- Enforce HTTPS in all environments — never allow plain HTTP in production.
- Use TLS 1.2 or higher.
- Apply HTTP Strict Transport Security (HSTS).
- Validate TLS certificates — never disable certificate validation.

### Logging and Monitoring

- Never log sensitive data: passwords, tokens, credit card numbers, personally identifiable information.
- Log authentication failures, authorisation denials, and unusual activity.
- Use structured logging with correlation IDs for traceability.
- Integrate with a monitoring platform (Application Insights, Datadog, etc.).

### Dependency Management

- Keep all dependencies up to date.
- Run automated vulnerability scanning on every build (e.g., `dotnet list package --vulnerable`).
- Remove unused dependencies.
- Pin dependency versions in production applications.
- Review transitive dependencies periodically.

### API Security

- Use rate limiting on all public-facing endpoints.
- Return consistent error responses — do not leak stack traces or internal details.
- Validate Content-Type headers on incoming requests.
- Use `[Authorize]` or equivalent middleware on all authenticated endpoints.
- Implement CORS policies explicitly — never use wildcard (`*`) in production.

---

## Azure-Specific Security

- Use Managed Identity for all Azure service communication — never store credentials.
- Store all secrets in Azure Key Vault.
- Enable Defender for Cloud and apply its recommendations.
- Use Private Endpoints for database and storage services.
- Apply network security groups and restrict public access to APIs.
- Enable audit logging on all Azure resources.

---

## Best Practices

- Apply security at every layer — do not rely solely on the presentation layer.
- Use OWASP Top 10 as a baseline security checklist.
- Perform threat modelling for new features.
- Apply defence in depth — multiple security controls at different layers.
- Keep dependencies patched — subscribe to CVE feeds for used packages.
- Use code scanning tools (GitHub Advanced Security, SonarQube) in CI.
- Never trust data from external systems without validation.

---

## Bad Practices

- Disabling SSL/TLS verification in any environment.
- Logging passwords, tokens, or PII.
- Using `admin`/`admin` or default credentials in any environment.
- Storing secrets in `appsettings.json` committed to source control.
- Using `[AllowAnonymous]` on endpoints that should be protected.
- Returning detailed exception messages to clients.
- Using MD5 or SHA-1 for password hashing.

---

## Examples

### Registering Key Vault in .NET

```csharp
// ConfigurationLayer/ServiceCollectionExtensions.cs
public static IServiceCollection AddProjectDependencies(
    this IServiceCollection services,
    IConfiguration configuration)
{
    services.AddAzureClients(clientBuilder =>
    {
        clientBuilder.AddSecretClient(new Uri(configuration["KeyVault:Uri"]!));
        clientBuilder.UseCredential(new DefaultAzureCredential());
    });

    // ... other registrations
    return services;
}
```

### FluentValidation Example

```csharp
public class CreateOrderCommandValidator : AbstractValidator<CreateOrderCommand>
{
    public CreateOrderCommandValidator()
    {
        RuleFor(x => x.CustomerId)
            .NotEmpty()
            .WithMessage("Customer ID is required.");

        RuleFor(x => x.Items)
            .NotEmpty()
            .WithMessage("At least one item is required.")
            .Must(items => items.Count <= 100)
            .WithMessage("Order cannot contain more than 100 items.");
    }
}
```

---

## Common Mistakes

- Exposing internal IDs (database primary keys) in public APIs — prefer opaque GUIDs.
- Using the same encryption key across environments.
- Trusting the `X-Forwarded-For` header without proper proxy configuration.
- Not limiting file upload size and type.
- Returning HTTP 200 for authentication failures instead of 401.

---

## Checklist

- [ ] No secrets committed to version control
- [ ] All secrets stored in Key Vault or environment variables
- [ ] Managed Identity used on Azure
- [ ] All inputs validated with a validation library
- [ ] Parameterised queries / ORM used everywhere
- [ ] HTTPS enforced in all environments
- [ ] Authentication and authorisation applied to all protected endpoints
- [ ] Sensitive data never logged
- [ ] Dependencies scanned for known vulnerabilities
- [ ] Error responses do not leak stack traces or internal details
- [ ] Rate limiting applied to public endpoints
- [ ] CORS configured explicitly — no wildcard in production
