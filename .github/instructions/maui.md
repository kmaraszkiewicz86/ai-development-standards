# .NET MAUI Standards

## Purpose

This document defines engineering standards for all .NET MAUI projects in this
repository. Every MAUI application must follow these conventions for
structure, architecture, and implementation.

---

## Guidelines

### Target Framework

Always target the latest stable .NET MAUI version.

### Solution Structure

Always generate this solution:

| Project | Responsibility |
|---------|---------------|
| `MyApp.UI` | App, AppShell, MauiProgram, bootstrap |
| `MyApp.Presentation` | Views, ViewModels, Behaviors, ValueConverters, Navigation, Localization, Styles, Resources |
| `MyApp.ApplicationLayer` | Use cases, Services, DTOs |
| `MyApp.DomainLayer` | Entities, Value Objects |
| `MyApp.InfrastructureLayer` | External integrations, Persistence |
| `MyApp.ConfigurationLayer` | Composition Root, dependency registration |

---

## Required Packages

### Presentation Project

```bash
dotnet add package Kmaraszkiewicz86.Maui.Behaviors --version 1.0.3
dotnet add package CommunityToolkit.Mvvm
dotnet add package CommunityToolkit.Maui
```

Reference: https://github.com/kmaraszkiewicz86/Kmaraszkiewicz86.Maui.Behaviors

Always prefer `Kmaraszkiewicz86.Maui.Behaviors` over creating custom Behaviors.

---

## MVVM

### Rules

- Always use MVVM.
- Business logic must never be placed in Views or code-behind.
- Resolve ViewModels through Dependency Injection — never instantiate directly.
- Assign `BindingContext` in code-behind only.
- Do not set `BindingContext` in XAML.
- Do not use `x:DataType` or Compiled Bindings.

### ViewModels

- Every ViewModel must inherit from `ObservableObject`.
- Import `using CommunityToolkit.Mvvm.ComponentModel;`
- Use primary constructors for dependency injection.
- Do not use the `[ObservableProperty]` source generator — use explicit `SetProperty()` properties.

```csharp
// Good — explicit property
private int _id;

public int Id
{
    get => _id;
    set => SetProperty(ref _id, value);
}
```

```csharp
// Bad — ObservableProperty source generator
[ObservableProperty]
private int _id;
```

### ViewModel Example

```csharp
// Presentation/ViewModels/MainPageViewModel.cs
namespace MyApp.Presentation.ViewModels;

public partial class MainPageViewModel(
    IOrderService orderService)
    : ObservableObject
{
    private IReadOnlyList<OrderSummaryDto> _orders = [];

    public IReadOnlyList<OrderSummaryDto> Orders
    {
        get => _orders;
        private set => SetProperty(ref _orders, value);
    }

    private bool _isLoading;

    public bool IsLoading
    {
        get => _isLoading;
        private set => SetProperty(ref _isLoading, value);
    }

    public IAsyncRelayCommand LoadCommand => new AsyncRelayCommand(LoadAsync);

    private async Task LoadAsync()
    {
        IsLoading = true;

        try
        {
            Orders = await orderService.GetOrdersAsync();
        }
        finally
        {
            IsLoading = false;
        }
    }
}
```

---

## Commands

- Prefer `AsyncRelayCommand` for asynchronous operations.
- Use `RelayCommand` only for synchronous operations.
- Do not use the `[RelayCommand]` source generator.
- Do not use the `[RelayCommand]` attribute.
- Expose commands as properties.
- Commands should invoke methods only — no business logic inside commands.

```csharp
// Good
public IAsyncRelayCommand LoadCommand => new AsyncRelayCommand(LoadAsync);
public IAsyncRelayCommand<Guid> DeleteCommand => new AsyncRelayCommand<Guid>(DeleteAsync);

// Bad — using source generator
[RelayCommand]
private async Task Load() { }
```

---

## Views

### Code-Behind

- Keep code-behind minimal.
- Only assign `BindingContext` and handle MAUI framework requirements.
- Never place business logic in code-behind.

```csharp
// Good
namespace MyApp.Presentation.Views;

public partial class MainPage : ContentPage
{
    public MainPage(MainPageViewModel viewModel)
    {
        InitializeComponent();
        BindingContext = viewModel;
    }
}
```

### XAML Bindings

- Use standard XAML bindings.
- Do not use `x:DataType`.
- Do not use Compiled Bindings.
- Keep bindings simple and readable.

```xml
<!-- Good -->
<Label Text="{Binding Title}" />
<ActivityIndicator IsRunning="{Binding IsLoading}" />

<!-- Bad — compiled binding -->
<Label x:DataType="viewmodels:MainPageViewModel" Text="{Binding Title}" />
```

---

## Navigation

- Use Shell Navigation.
- Define routes in `AppShell.xaml`.
- Navigate using `Shell.Current.GoToAsync()`.
- Pass parameters using query string syntax or `ShellNavigationQueryParameters`.

```csharp
// Navigate to a page
await Shell.Current.GoToAsync($"{nameof(OrderDetailPage)}?orderId={orderId}");

// Receive parameter
[QueryProperty(nameof(OrderId), "orderId")]
public partial class OrderDetailPageViewModel : ObservableObject
{
    private Guid _orderId;

    public Guid OrderId
    {
        get => _orderId;
        set => SetProperty(ref _orderId, value);
    }
}
```

---

## Localization

Always generate:

```
Resources/Languages/
├── AppResources.en.resx
└── AppResources.pl.resx
```

Configure localization for English and Polish.

---

## Initial Page

Every new application must contain a minimal initial page demonstrating the architecture:

```
Presentation/
├── Views/
│   ├── MainPage.xaml
│   └── MainPage.xaml.cs
└── ViewModels/
    └── MainPageViewModel.cs
```

The initial page must:

- Demonstrate MVVM correctly.
- Use a ViewModel resolved through Dependency Injection.
- Show at least one binding.
- Use `AsyncRelayCommand` for any actions.

---

## Services

- Every ViewModel communicates with services through interfaces.
- Never instantiate services directly inside ViewModels.
- Use Dependency Injection for all service resolution.
- Define service interfaces in ApplicationLayer.
- Implement services in InfrastructureLayer or ApplicationLayer as appropriate.

---

## Best Practices

- Use Shell for all navigation.
- Use `ObservableObject` as the base for all ViewModels.
- Use `IAsyncRelayCommand` for all async commands.
- Use `CommunityToolkit.Maui` for controls and helpers.
- Keep Views as thin as possible.
- Validate input in ViewModels, not in Views.
- Use `IConnectivity` for network state checks.
- Use `IPreferences` for local settings.
- Use `SecureStorage` for sensitive local data.

---

## Bad Practices

- Placing business logic in Views or code-behind.
- Instantiating ViewModels directly in Views.
- Using `x:DataType` or Compiled Bindings.
- Using the `[ObservableProperty]` or `[RelayCommand]` source generators.
- Accessing the database directly from a ViewModel.
- Making HTTP requests directly from a ViewModel.
- Navigating without Shell.

---

## Examples

### Minimal MainPage

```xml
<!-- Presentation/Views/MainPage.xaml -->
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="MyApp.Presentation.Views.MainPage"
             Title="{Binding Title}">
    <VerticalStackLayout Spacing="16" Padding="24">
        <Label Text="{Binding WelcomeMessage}"
               FontSize="24"
               HorizontalOptions="Center" />
        <ActivityIndicator IsRunning="{Binding IsLoading}"
                           IsVisible="{Binding IsLoading}" />
        <Button Text="Load Data"
                Command="{Binding LoadCommand}" />
    </VerticalStackLayout>
</ContentPage>
```

```csharp
// Presentation/Views/MainPage.xaml.cs
namespace MyApp.Presentation.Views;

public partial class MainPage : ContentPage
{
    public MainPage(MainPageViewModel viewModel)
    {
        InitializeComponent();
        BindingContext = viewModel;
    }
}
```

---

## Common Mistakes

- Forgetting to register Views and ViewModels in ConfigurationLayer.
- Calling async methods without `await` in commands.
- Not using `SetProperty()` when modifying observable properties.
- Navigating using `new Page()` instead of Shell navigation.
- Not disposing long-running operations when the page is destroyed.

---

## Checklist

- [ ] MVVM applied — no business logic in Views
- [ ] ViewModels inherit `ObservableObject`
- [ ] Primary constructors used for DI
- [ ] `[ObservableProperty]` source generator not used — explicit `SetProperty()` used
- [ ] `[RelayCommand]` source generator not used — explicit command properties used
- [ ] `BindingContext` assigned in code-behind — not XAML
- [ ] `x:DataType` and Compiled Bindings not used
- [ ] Shell Navigation used
- [ ] `Kmaraszkiewicz86.Maui.Behaviors` used instead of custom Behaviors
- [ ] Localization configured for English and Polish
- [ ] MainPage, MainPage.xaml.cs, and MainPageViewModel created
- [ ] All services injected — never instantiated directly
