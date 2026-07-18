# Create React Feature Prompt

Create a complete, production-ready React feature following the standards
defined in this repository.

---

## Instructions

Generate a fully functional React feature for the feature described below.

Follow all standards defined in:
- `.github/instructions/react.md`
- `.github/instructions/naming.md`
- `.github/instructions/coding-style.md`

---

## What to generate

### Folder Structure

```
src/features/{feature-name}/
├── components/
│   ├── {FeatureName}List.tsx
│   └── {FeatureName}Card.tsx
├── hooks/
│   └── use{FeatureName}.ts
├── services/
│   └── {featureName}Api.ts
├── models/
│   └── {featureName}.model.ts
└── index.ts
```

### Model

- Use TypeScript `interface` for object shapes
- Use `type` for unions and aliases
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
  items: OrderLineItem[];
}
```

### API Service

- Encapsulate all Axios calls in the service
- Use the shared `apiClient` instance
- Return typed responses

```typescript
// services/orderApi.ts
import { apiClient } from '../../../api/axiosInstance';
import type { Order, CreateOrderRequest } from '../models/order.model';

export const orderApi = {
  getAll: async (): Promise<Order[]> => {
    const { data } = await apiClient.get<Order[]>('/orders');
    return data;
  },

  getById: async (id: string): Promise<Order> => {
    const { data } = await apiClient.get<Order>(`/orders/${id}`);
    return data;
  },

  create: async (request: CreateOrderRequest): Promise<Order> => {
    const { data } = await apiClient.post<Order>('/orders', request);
    return data;
  },

  delete: async (id: string): Promise<void> => {
    await apiClient.delete(`/orders/${id}`);
  },
};
```

### React Query Hooks

- Use React Query for all server state
- Export one file of hooks per feature

```typescript
// hooks/useOrders.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { orderApi } from '../services/orderApi';

export const ORDERS_QUERY_KEY = ['orders'] as const;

export const useOrders = () =>
  useQuery({
    queryKey: ORDERS_QUERY_KEY,
    queryFn: orderApi.getAll,
  });

export const useOrder = (id: string) =>
  useQuery({
    queryKey: [...ORDERS_QUERY_KEY, id],
    queryFn: () => orderApi.getById(id),
    enabled: !!id,
  });

export const useCreateOrder = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: orderApi.create,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ORDERS_QUERY_KEY });
    },
  });
};
```

### Components

- Functional Components with TypeScript
- Use the custom hooks for data
- Handle loading and error states
- No API calls directly in components

```tsx
// components/OrderList.tsx
import { useOrders } from '../hooks/useOrders';
import { OrderCard } from './OrderCard';

export const OrderList: React.FC = () => {
  const { data: orders, isLoading, isError } = useOrders();

  if (isLoading) return <div>Loading orders...</div>;
  if (isError) return <div>Failed to load orders.</div>;

  return (
    <ul>
      {orders?.map((order) => (
        <OrderCard key={order.id} order={order} />
      ))}
    </ul>
  );
};
```

### Page Component

- Create a page component in `src/pages/`
- Add the route in the router configuration

---

## Requirements

- Use the latest stable React with TypeScript.
- Use React Query for all server state — no manual `useEffect` for fetching.
- Axios calls in `services/` — not in components.
- TypeScript strict mode.
- No `any` types.
- Handle loading and error states from React Query.
- Named exports preferred.
- `readonly` for immutable model fields.

---

## Feature Description

[Describe the feature here — e.g., "An orders feature with a list of all orders showing status and total, with the ability to create a new order and delete an existing one."]
