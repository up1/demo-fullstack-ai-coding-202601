# ReactJS Rules

## Responsibility
Implement ReactJS components based strictly on the frontend specification.

## Rules
- Do NOT change frontend specification.
- Do NOT assume backend behavior.
- Use functional components only.
- Use hooks, not classes.
- Use explicit props typing.
- UI must be testable via provided test selectors.
- Accessibility is mandatory.

## Project structure
* Use feature-based structure
```
src/
  features/
    orders/
      components/
        OrderForm.tsx
      pages/
        CreateOrderPage.tsx
      hooks/
        useCreateOrder.ts
      api/
        ordersApi.ts
      types/
        orderTypes.ts
      __tests__/
```

## Component Rules
### Must
- One component = one responsibility
- Pure UI components whenever possible
- Controlled inputs
- Predictable render paths

### Must NOT
- Side effects in render
- API calls directly in components
- Hidden state coupling

## Hooks Rules
- Do hooks for side effects
- Do one hook per use case
- No hooks inside conditions
- No shared mutable state

## API Layer Rules
- API calls isolated
- No fetch/axios in components
- Explicit request/response types
- Deterministic error handling

## Testability Rules (CRITICAL)
- Every interactive element MUST expose with `data-testid="..."`
- Test IDs must match frontend spec
- No dynamic test IDs

## Accessibility Rules
- Labels for inputs
- Button type specified
- Keyboard navigation works
- Error messages are screen-reader friendly

## Validation Rules
- Client-side validation mirrors backend rules
- Never replace server validation
- Validation messages must match spec

## Code example

Folder Structure
```
src/features/orders/
├── api/ordersApi.ts
├── hooks/useCreateOrder.ts
├── components/OrderForm.tsx
├── pages/CreateOrderPage.tsx
├── types/orderTypes.ts
```

types/orderTypes.ts
```
export interface CreateOrderRequest {
  quantity: number;
}

export interface CreateOrderResponse {
  orderId: string;
}

export interface ApiError {
  code: string;
  message: string;
}
```

api/ordersApi.ts
```
import { CreateOrderRequest, CreateOrderResponse } from "../types/orderTypes";

export async function createOrder(
  request: CreateOrderRequest
): Promise<CreateOrderResponse> {
  const response = await fetch("/api/orders", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(request),
  });

  if (!response.ok) {
    const error = await response.json();
    throw error;
  }

  return response.json();
}
```

hooks/useCreateOrder.ts
```
import { useState } from "react";
import { createOrder } from "../api/ordersApi";
import { CreateOrderResponse, ApiError } from "../types/orderTypes";

export function useCreateOrder() {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<ApiError | null>(null);
  const [data, setData] = useState<CreateOrderResponse | null>(null);

  const submit = async (quantity: number) => {
    setLoading(true);
    setError(null);

    try {
      const result = await createOrder({ quantity });
      setData(result);
    } catch (err: any) {
      setError(err);
    } finally {
      setLoading(false);
    }
  };

  return { submit, loading, error, data };
}
```

components/OrderForm.tsx
```
import { useState } from "react";
import { useCreateOrder } from "../hooks/useCreateOrder";

export function OrderForm() {
  const [quantity, setQuantity] = useState<number>(1);
  const { submit, loading, error, data } = useCreateOrder();

  const onSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (quantity <= 0) return;
    submit(quantity);
  };

  return (
    <form onSubmit={onSubmit}>
      <label htmlFor="quantity">Quantity</label>
      <input
        id="quantity"
        type="number"
        min={1}
        value={quantity}
        onChange={(e) => setQuantity(Number(e.target.value))}
        data-testid="quantity-input"
      />

      <button
        type="submit"
        disabled={loading}
        data-testid="submit-order"
      >
        Create Order
      </button>

      {error && (
        <div role="alert" data-testid="order-error">
          {error.message}
        </div>
      )}

      {data && (
        <div data-testid="order-success">
          Order created: {data.orderId}
        </div>
      )}
    </form>
  );
}
```

pages/CreateOrderPage.tsx
```
import { OrderForm } from "../components/OrderForm";

export function CreateOrderPage() {
  return (
    <main>
      <h1>Create Order</h1>
      <OrderForm />
    </main>
  );
}
```
