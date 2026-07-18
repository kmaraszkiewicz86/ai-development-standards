# React Standards

## Purpose

This document defines engineering standards for all React projects in this
repository. Every React application must follow these conventions for
architecture, components, hooks, and data fetching.

---

## Guidelines

### Target Framework

Always use the latest stable React version with TypeScript.

### Preferred Technologies

| Category | Technology |
|----------|-----------|
| Language | TypeScript |
| Components | Functional Components |
| State | React hooks (`useState`, `useReducer`) |
| Server state | React Query (`@tanstack/react-query`) |
| HTTP | Axios |
| Routing | React Router |
| Forms | React Hook Form |
| Validation | Zod |
| Styling | CSS Modules or Tailwind CSS |

---

## Folder Structure

```
src/
├── api/             ← Axios instances, API base configuration
├── features/        ← Feature-based organisation (each feature is self-contained)
│   └── orders/
│       ├── components/
│       ├── hooks/
│       ├── services/
│       ├── models/
│       └── index.ts
├── components/      ← Shared UI components used across features
├── hooks/           ← Shared custom hooks
├── services/        ← Shared services not tied to a specific feature
├── pages/           ← Page-level components (route targets)
├── layouts/         ← Layout components (headers, sidebars, wrappers)
├── models/          ← Shared types and interfaces
├── utils/           ← Pure utility functions
└── app/             ← App entry, routing configuration, providers
```

---

## Components

Always use Functional Components with TypeScript:

```tsx
// features/orders/components/OrderCard.tsx
interface OrderCardProps {
  order: Order;
  onDelete: (id: string) => void;
}

export const OrderCard: React.FC<OrderCardProps> = ({ order, onDelete }) => {
  return (
    <div className={styles.card}>
      <h3>{order.customerName}</h3>
      <p>Total: {order.total}</p>
      <button onClick={() => onDelete(order.id)}>Delete</button>
    </div>
  );
};
```

### Rules

- Always type component props with an interface or type.
- Keep components small and focused on rendering.
- Never put business logic directly in components.
- Avoid prop drilling — use context or state management for deeply nested state.
- Keep component files short — extract sub-components when needed.

---

## React Query

Always use React Query (TanStack Query).

Use `useQuery()` and `useMutation()` directly inside React components whenever the logic is simple and used only by that component.

Extract a custom hook only when:

- the logic is reused,
- the query becomes complex,
- multiple components share the same behavior,
- it significantly improves readability.

Do not create custom hooks for every API endpoint by default.

Prefer simplicity over unnecessary abstraction.

---

## Axios

Use Axios with typed instances for all HTTP communication:

```typescript
// api/axiosInstance.ts
import axios from 'axios';

export const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  headers: { 'Content-Type': 'application/json' },
});

apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = 'Bearer ' + token;
  }
  return config;
});

apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // Handle unauthorised
    }
    return Promise.reject(error);
  }
);
```

```typescript
// features/orders/services/orderApi.ts
import { apiClient } from '../../../api/axiosInstance';
import type { Order, CreateOrderRequest } from '../models/order';

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

---

## Custom Hooks

Extract reusable logic into custom hooks:

```tsx
// hooks/useDebounce.ts
import { useState, useEffect } from 'react';

export const useDebounce = <T>(value: T, delay: number): T => {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const handler = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
};
```

---

## TypeScript

- Enable `strict` mode in `tsconfig.json`.
- Always type function parameters and return types.
- Prefer `interface` for object shapes and `type` for unions and aliases.
- Avoid `any` — use `unknown` and type guards when the type is not known.
- Use `readonly` for immutable properties.

```typescript
// Good — typed model
export interface Order {
  readonly id: string;
  readonly customerId: string;
  status: OrderStatus;
  total: number;
  readonly createdAt: string;
}

export type OrderStatus = 'Pending' | 'Processing' | 'Shipped' | 'Delivered' | 'Cancelled';
```

---

## Best Practices

- Always define TypeScript types for component props, API responses, and state.
- Use React Query for all server-side data — avoid manual `useEffect` for fetching.
- Keep API logic in `api/` or feature `services/` — not inside components.
- Use `useCallback` for event handlers passed as props to prevent unnecessary re-renders.
- Use `useMemo` for expensive calculations — not for every derived value.
- Use React Router for all navigation.
- Use environment variables (`.env`) for configuration — never hardcode URLs.
- Prefer named exports over default exports.

---

## Bad Practices

- Fetching data directly inside components with `useEffect` and `fetch`.
- Using `any` as a type.
- Storing server state in `useState` when React Query handles it.
- Prop drilling more than two levels — use context or state management.
- Mutating state directly.
- Using `index` as a key in lists when items can reorder.
- Placing API calls outside the `api/` or `services/` layer.

---

## Common Mistakes

- Forgetting to handle loading and error states from React Query.
- Not invalidating queries after mutations.
- Using `useEffect` with a missing dependency.
- Not memoising selector functions when used with context.
- Forgetting to configure `QueryClientProvider` at the root.
