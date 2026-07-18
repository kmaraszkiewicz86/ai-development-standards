# Create Angular Feature Prompt

Create a complete, production-ready Angular feature following the standards
defined in this repository.

---

## Instructions

Generate a fully functional Angular feature for the feature described below.

Follow all standards defined in:
- `.github/instructions/angular.md`
- `.github/instructions/naming.md`
- `.github/instructions/coding-style.md`

---

## What to generate

### Folder Structure

```
src/app/features/{feature-name}/
├── components/
│   ├── {feature-name}-list/
│   │   ├── {feature-name}-list.component.ts
│   │   ├── {feature-name}-list.component.html
│   │   └── {feature-name}-list.component.scss
│   └── {feature-name}-detail/
│       ├── {feature-name}-detail.component.ts
│       ├── {feature-name}-detail.component.html
│       └── {feature-name}-detail.component.scss
├── services/
│   └── {feature-name}.service.ts
├── models/
│   └── {feature-name}.model.ts
├── {feature-name}.routes.ts
└── index.ts
```

### Model

- Define TypeScript interfaces in `models/`
- Use `readonly` for immutable fields

```typescript
// models/order.model.ts
export interface Order {
  readonly id: string;
  readonly customerId: string;
  status: OrderStatus;
  total: number;
  readonly createdAt: string;
}

export type OrderStatus = 'Pending' | 'Processing' | 'Shipped' | 'Delivered' | 'Cancelled';

export interface CreateOrderRequest {
  customerId: string;
  items: OrderItem[];
}
```

### Service

- Encapsulate all HTTP calls in the service
- Return typed `Observable<T>`
- Use `inject(HttpClient)` for DI
- No business logic in components

```typescript
// services/order.service.ts
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

### Components

- Use Standalone Components
- Apply `ChangeDetectionStrategy.OnPush`
- Use Signals for state
- Use `inject()` for dependencies
- No HTTP calls in components

```typescript
@Component({
  selector: 'app-order-list',
  standalone: true,
  imports: [CommonModule, RouterModule],
  templateUrl: './order-list.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class OrderListComponent implements OnInit {
  private readonly orderService = inject(OrderService);

  orders = signal<Order[]>([]);
  isLoading = signal(false);
  totalCount = computed(() => this.orders().length);

  async ngOnInit(): Promise<void> {
    this.isLoading.set(true);
    this.orders.set(await firstValueFrom(this.orderService.getAll()));
    this.isLoading.set(false);
  }
}
```

### Routes

- Configure lazy-loaded routes for the feature
- Register routes in the feature `{feature-name}.routes.ts` file

```typescript
// orders.routes.ts
export const ORDERS_ROUTES: Routes = [
  { path: '', component: OrderListComponent },
  { path: ':id', component: OrderDetailComponent },
];
```

### Registration

- Add the feature routes to `app.routes.ts` with lazy loading

```typescript
{
  path: 'orders',
  loadChildren: () => import('./features/orders/orders.routes').then(m => m.ORDERS_ROUTES),
},
```

---

## Requirements

- Use the latest stable Angular.
- Use Standalone Components — no NgModules.
- Use Signals for state management.
- Apply `ChangeDetectionStrategy.OnPush`.
- Lazy load the feature routes.
- No HTTP calls directly in components — use services.
- Use `inject()` for all dependencies.
- No `any` types.
- Use `readonly` for immutable model fields.

---

## Feature Description

[Describe the feature here — e.g., "An orders feature with a list view showing all orders with status and total, and a detail view showing full order information."]
