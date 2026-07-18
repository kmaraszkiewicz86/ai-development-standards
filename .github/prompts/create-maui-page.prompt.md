# Create MAUI Page Prompt

Create a complete, production-ready .NET MAUI page following the MVVM standards
defined in this repository.

---

## Instructions

Generate a fully functional MAUI page for the feature described below.

Follow all standards defined in:
- `.github/instructions/maui.md`
- `.github/instructions/naming.md`
- `.github/instructions/coding-style.md`

---

## What to generate

### View (XAML + Code-Behind)

- `Views/{PageName}Page.xaml` — XAML layout
- `Views/{PageName}Page.xaml.cs` — code-behind (minimal — only BindingContext assignment)

Rules:
- Do not use `x:DataType`
- Do not use Compiled Bindings
- Assign `BindingContext` in code-behind only
- Use standard XAML bindings: `{Binding PropertyName}`
- Keep XAML readable — extract complex content into sub-views if needed

```xml
<!-- Views/OrderListPage.xaml -->
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="MyApp.Presentation.Views.OrderListPage"
             Title="{Binding Title}">
    <VerticalStackLayout>
        <ActivityIndicator IsRunning="{Binding IsLoading}"
                           IsVisible="{Binding IsLoading}" />
        <CollectionView ItemsSource="{Binding Orders}">
            <CollectionView.ItemTemplate>
                <DataTemplate>
                    <VerticalStackLayout Padding="16">
                        <Label Text="{Binding CustomerName}" FontSize="16" />
                        <Label Text="{Binding Total, StringFormat='Total: {0:C}'}" />
                    </VerticalStackLayout>
                </DataTemplate>
            </CollectionView.ItemTemplate>
        </CollectionView>
        <Button Text="Refresh" Command="{Binding LoadCommand}" />
    </VerticalStackLayout>
</ContentPage>
```

```csharp
// Views/OrderListPage.xaml.cs
namespace MyApp.Presentation.Views;

public partial class OrderListPage : ContentPage
{
    public OrderListPage(OrderListPageViewModel viewModel)
    {
        InitializeComponent();
        BindingContext = viewModel;
    }
}
```

### ViewModel

- `ViewModels/{PageName}PageViewModel.cs`
- Inherit from `ObservableObject`
- Use primary constructor for DI
- Use explicit `SetProperty()` for all observable properties — not `[ObservableProperty]`
- Use `IAsyncRelayCommand` for async commands — not `[RelayCommand]`
- No business logic in the ViewModel — delegate to services
- No direct database or HTTP access

```csharp
// ViewModels/OrderListPageViewModel.cs
namespace MyApp.Presentation.ViewModels;

public partial class OrderListPageViewModel(
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

    public string Title => "Orders";

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

### Navigation Registration

- Add the page route to `AppShell.xaml`
- Register the View and ViewModel in `ConfigurationLayer`

### Localization

- Add any new string keys to `AppResources.en.resx` and `AppResources.pl.resx`

---

## Requirements

- No `[ObservableProperty]` source generator — explicit `SetProperty()` only.
- No `[RelayCommand]` source generator — explicit `IAsyncRelayCommand` only.
- No business logic in ViewModel — delegate to services.
- `BindingContext` assigned in code-behind — not XAML.
- No `x:DataType` or Compiled Bindings.
- Shell Navigation used.
- All services injected — never instantiated directly.
- Use `Kmaraszkiewicz86.Maui.Behaviors` instead of custom Behaviors where possible.

---

## Page Description

[Describe the page here — e.g., "An order list page that displays all orders for the current user, with a refresh button and loading indicator, navigating to an order detail page on tap."]
