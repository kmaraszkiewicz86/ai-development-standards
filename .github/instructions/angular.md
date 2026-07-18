# Angular Standards

## Purpose

This document defines engineering standards for all Angular projects in this
repository. Every Angular application must follow these conventions for
architecture, components, services, and state management.

---

## Guidelines

### Target Framework

Always use the latest stable Angular version.

### Preferred Technologies

| Category | Technology |
|----------|-----------|
| Components | Standalone Components |
| State | Signals |
| Reactivity | RxJS |
| HTTP | `HttpClient` |
| Routing | Lazy loading with feature routes |
| Forms | Reactive Forms |
| Validation | Angular Validators + custom validators |
| Styles | SCSS per component |

---

## Folder Structure

```
src/
├── app/
│   ├── core/            ← Singleton services, guards, interceptors, app-level providers
│   ├── shared/          ← Shared components, directives, pipes, models used across features
│   ├── features/        ← Feature modules / standalone feature components
│   │   └── orders/
│   │       ├── components/
│   │       ├── services/
│   │       ├── models/
│   │       └── orders.routes.ts
│   ├── models/          ← Global shared models and interfaces
│   ├── services/        ← Global services not tied to a specific feature
│   ├── guards/          ← Route guards
│   ├── interceptors/    ← HTTP interceptors
│   └── layout/          ← Layout components (header, footer, sidebar)
└── environments/
```

---

## Standalone Components

Always use Standalone Components — do not use NgModules for feature code.

```typescript
// Good — standalone component
@Component({
  selector: 'app-order-list',
  standalone: true,
  imports: [CommonModule, RouterModule],
  templateUrl: './order-list.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class OrderListComponent {
  private readonly orderService = inject(OrderService);

  orders = this.orderService.getOrders();
}

// Bad — module-based component
@NgModule({
  declarations: [OrderListComponent],
  imports: [CommonModule],
})
export class OrdersModule {}
```

---

## Signals

Prefer Signals for reactive state management:

```typescript
import { signal, computed, effect } from '@angular/core';

export class OrderListComponent {
  private readonly orderService = inject(OrderService);

  orders = signal<Order[]>([]);
  isLoading = signal(false);
  totalCount = computed(() => this.orders().length);

  async loadOrders(): Promise<void> {
    this.isLoading.set(true);

    try {
      const result = await firstValueFrom(this.orderService.getAll());
      this.orders.set(result);
    } finally {
      this.isLoading.set(false);
    }
  }
}
```

---

## Services

- Define service interfaces when testing is important.
- Keep services single-purpose.
- Use `inject()` for dependency injection inside services and components.
- Provide services at the appropriate level (`root`, feature, or component).
- Never place HTTP calls directly in components.

```typescript
// Good — service encapsulates HTTP
@Injectable({ providedIn: 'root' })
export class OrderService {
  private readonly http = inject(HttpClient);
  private readonly baseUrl = '/api/orders';

  getAll(): Observable<Order[]> {
    return this.http.get<Order[]>(this.baseUrl);
  }

  getById(id: string): Observable<Order> {
    return this.http.get<Order>(`${this.baseUrl}/${id}`);
  }

  create(request: CreateOrderRequest): Observable<Order> {
    return this.http.post<Order>(this.baseUrl, request);
  }
}
```

---

## Lazy Loading

Always configure lazy loading for feature routes:

```typescript
// app.routes.ts
export const appRoutes: Routes = [
  {
    path: 'orders',
    loadChildren: () => import('./features/orders/orders.routes').then(m => m.ORDERS_ROUTES),
  },
];

// features/orders/orders.routes.ts
export const ORDERS_ROUTES: Routes = [
  {
    path: '',
    component: OrderListComponent,
  },
  {
    path: ':id',
    component: OrderDetailComponent,
  },
];
```

---

## HTTP Interceptors

Use interceptors for cross-cutting HTTP concerns:

```typescript
// core/interceptors/auth.interceptor.ts
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.getToken();

  if (token) {
    req = req.clone({
      setHeaders: { Authorization: 'Bearer ' + token },
    });
  }

  return next(req);
};
```

---

## Error Handling

Handle HTTP errors in services or via a global interceptor:

```typescript
// core/interceptors/error.interceptor.ts
export const errorInterceptor: HttpInterceptorFn = (req, next) =>
  next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      if (error.status === 401) {
        inject(AuthService).logout();
      }
      return throwError(() => error);
    })
  );
```

---

## Best Practices

- Use `ChangeDetectionStrategy.OnPush` on all components.
- Use `inject()` instead of constructor injection in modern Angular.
- Use Signals for component state and `computed()` for derived state.
- Use `AsyncPipe` or `toSignal()` to handle observables in templates.
- Use Reactive Forms for complex forms.
- Keep templates free of business logic — delegate to components or services.
- Apply route guards for protected routes.
- Use `environment.ts` for environment-specific configuration — never hardcode URLs.

---

## Bad Practices

- Placing HTTP calls directly in components.
- Using NgModules for new feature code.
- Using `any` as a type.
- Using `subscribe()` without unsubscribing in components (prefer `AsyncPipe` or `toSignal()`).
- Putting business logic inside templates.
- Accessing DOM directly without `@ViewChild` or renderer.
- Using `BehaviorSubject` for state when Signals are available.

---

## Examples

### Feature Component with Signal

```typescript
@Component({
  selector: 'app-orders',
  standalone: true,
  imports: [CommonModule, RouterModule, OrderCardComponent],
  template: `
    <div *ngIf="isLoading()">Loading...</div>
    <ul>
      @for (order of orders(); track order.id) {
        <app-order-card [order]="order" />
      }
    </ul>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class OrdersComponent implements OnInit {
  private readonly orderService = inject(OrderService);

  orders = signal<Order[]>([]);
  isLoading = signal(false);

  async ngOnInit(): Promise<void> {
    this.isLoading.set(true);
    this.orders.set(await firstValueFrom(this.orderService.getAll()));
    this.isLoading.set(false);
  }
}
```

---

## Common Mistakes

- Subscribing to observables in `ngOnInit` without unsubscribing.
- Not using `OnPush` change detection.
- Placing route logic inside components instead of guards.
- Not handling HTTP errors.
- Not using `trackBy` (or `track` in block syntax) in `*ngFor`/`@for`.

---

## Checklist

- [ ] Latest stable Angular version used
- [ ] Standalone Components used — no NgModules for feature code
- [ ] Signals used for reactive state
- [ ] `ChangeDetectionStrategy.OnPush` applied
- [ ] Lazy loading configured for all feature routes
- [ ] HTTP calls encapsulated in services
- [ ] `inject()` used for dependency injection
- [ ] No business logic in templates
- [ ] Error interceptor configured
- [ ] Auth interceptor configured
- [ ] No `any` types
- [ ] Environment variables used — no hardcoded URLs
