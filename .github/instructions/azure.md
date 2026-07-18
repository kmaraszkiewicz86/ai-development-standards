# Azure Standards

## Purpose

This document defines engineering standards for all Azure cloud services used in
this repository. Every Azure integration must follow these conventions for
security, reliability, and maintainability.

---

## Guidelines

### Core Principle: Managed Identity First

Prefer Managed Identity over secrets for all Azure service communication.
Never store service account credentials, connection strings, or API keys in
application configuration or source code when Managed Identity is available.

```csharp
// Good — Managed Identity
var credential = new DefaultAzureCredential();
var secretClient = new SecretClient(new Uri(keyVaultUri), credential);

// Bad — hardcoded credentials
var secretClient = new SecretClient(new Uri(keyVaultUri),
    new ClientSecretCredential("tenantId", "clientId", "clientSecret"));
```

---

## Azure Key Vault

- Store all secrets, connection strings, and certificates in Key Vault.
- Never store sensitive values in `appsettings.json` or environment variables directly.
- Use `DefaultAzureCredential` for authentication.
- Reference Key Vault from configuration using the Key Vault configuration provider.

```csharp
// Program.cs
var keyVaultUri = builder.Configuration["KeyVault:Uri"]!;
builder.Configuration.AddAzureKeyVault(new Uri(keyVaultUri), new DefaultAzureCredential());
```

---

## Azure Functions

### Structure

```
MyFunctions/
├── Functions/           ← Trigger functions (HTTP, Timer, Service Bus, etc.)
├── Services/            ← Business logic / orchestration
├── Models/              ← Function-specific models
├── Extensions/          ← DI registration extensions
└── Program.cs           ← Minimal — uses ConfigurationLayer
```

### Rules

- Use the isolated worker model for all new Azure Functions.
- Use Dependency Injection — register services in `Program.cs` via `AddProjectDependencies()`.
- Keep function trigger code thin — delegate to services.
- Use `ILogger<T>` for structured logging.
- Always handle exceptions and return appropriate HTTP responses for HTTP triggers.

```csharp
// Functions/ProcessOrderFunction.cs
public class ProcessOrderFunction(
    IAsyncCommandDispatcher dispatcher,
    ILogger<ProcessOrderFunction> logger)
{
    [Function("ProcessOrder")]
    public async Task RunAsync(
        [ServiceBusTrigger("orders", Connection = "ServiceBusConnection")] OrderMessage message,
        CancellationToken cancellationToken)
    {
        logger.LogInformation("Processing order {OrderId}", message.OrderId);
        await dispatcher.DispatchAsync(new ProcessOrderCommand(message.OrderId), cancellationToken);
    }
}
```

---

## Azure App Service

- Use App Service managed certificates for TLS.
- Always enable HTTPS-only in App Service settings.
- Use App Service Managed Identity for Azure resource access.
- Store application settings in App Service Configuration — not in deployed files.
- Use deployment slots for zero-downtime deployments.
- Enable Application Insights for monitoring.

---

## Azure AI Foundry and Azure OpenAI

- Use `Azure.AI.OpenAI` SDK for all OpenAI integrations.
- Authenticate using Managed Identity — never hardcode API keys.
- Store deployment names and endpoint URIs in Key Vault or App Configuration.
- Always handle rate limiting and throttling with exponential back-off retry.
- Log token usage for cost monitoring.

```csharp
// Good — Managed Identity authentication
var client = new AzureOpenAIClient(
    new Uri(configuration["AzureOpenAI:Endpoint"]!),
    new DefaultAzureCredential());

// Bad — API key in code
var client = new AzureOpenAIClient(
    new Uri("https://my-instance.openai.azure.com"),
    new AzureKeyCredential("sk-..."));
```

---

## Azure AI Search

- Use Managed Identity for authentication.
- Define index schemas as strongly-typed C# classes.
- Use the Azure AI Search SDK (`Azure.Search.Documents`).
- Paginate search results — never return unbounded result sets.
- Configure semantic ranking where appropriate.

---

## Blob Storage

- Use Managed Identity authentication via `DefaultAzureCredential`.
- Always specify the content type when uploading blobs.
- Use SAS tokens with minimal permissions and short expiry for external access.
- Store container names in configuration — not hardcoded.
- Enable soft delete and versioning for production containers.

```csharp
// Good — Managed Identity
var blobServiceClient = new BlobServiceClient(
    new Uri($"https://{accountName}.blob.core.windows.net"),
    new DefaultAzureCredential());
```

---

## Application Insights

- Configure Application Insights in every application.
- Use structured logging with `ILogger<T>`.
- Add custom telemetry for business events.
- Use correlation IDs for distributed tracing.
- Set sampling rates to control costs in high-volume applications.

```csharp
// ConfigurationLayer — register Application Insights
services.AddApplicationInsightsTelemetry(configuration["ApplicationInsights:ConnectionString"]);
```

---

## Azure Service Bus

- Use Managed Identity for Service Bus authentication.
- Use `ServiceBusProcessor` for message processing.
- Always implement dead-lettering for failed messages.
- Use message sessions for ordered processing.
- Set appropriate lock durations and max delivery counts.

---

## Best Practices

- Apply the principle of least privilege to all Managed Identities.
- Tag all Azure resources with project, environment, and owner tags.
- Use Azure Policy to enforce organisation-wide standards.
- Use Private Endpoints to restrict access to Azure PaaS services.
- Enable diagnostic logging on all resources.
- Use Azure Monitor alerts for critical thresholds.
- Keep resource names consistent across environments using a naming convention.
- Use Bicep or Terraform for infrastructure as code.

---

## Bad Practices

- Storing secrets in app settings files committed to source control.
- Using personal credentials for service-to-service communication.
- Disabling TLS or using self-signed certificates in production.
- Allowing public access to databases or storage accounts when not required.
- Not implementing retry policies for transient failures.
- Not monitoring costs and usage.
- Using wildcard CORS policies in production APIs.

---

## Examples

### Key Vault with DefaultAzureCredential

```csharp
// ConfigurationLayer/InfrastructureConfiguration.cs
public static IServiceCollection AddInfrastructureServices(
    this IServiceCollection services,
    IConfiguration configuration)
{
    services.AddAzureClients(clientBuilder =>
    {
        clientBuilder.AddBlobServiceClient(
            new Uri($"https://{configuration["Storage:AccountName"]}.blob.core.windows.net"));

        clientBuilder.AddSecretClient(
            new Uri(configuration["KeyVault:Uri"]!));

        clientBuilder.UseCredential(new DefaultAzureCredential());
    });

    return services;
}
```

---

## Common Mistakes

- Not handling `RequestFailedException` from Azure SDKs.
- Forgetting to assign the Managed Identity the required RBAC role.
- Not configuring connection resiliency for Service Bus and Event Hub.
- Over-provisioning — using premium tiers where standard is sufficient.
- Not enabling zone redundancy for production workloads.

---

## Checklist

- [ ] Managed Identity used for all Azure service communication
- [ ] All secrets stored in Azure Key Vault
- [ ] No credentials in `appsettings.json` or source code
- [ ] Application Insights configured
- [ ] HTTPS-only enforced on App Service
- [ ] Private Endpoints used where applicable
- [ ] Retry policies configured for transient failures
- [ ] Diagnostic logging enabled on all resources
- [ ] Least privilege applied to Managed Identities
- [ ] Infrastructure defined as code (Bicep/Terraform)
