# Skill: Standardized Error Handling & Logging

**When to use:** Every Service (Django) or Service/Controller (NestJS) that contains business logic or external I/O.

---

## Procedure

1. **Custom exceptions** — define domain-specific exception classes. Never raise generic `Exception` or `Error`.
2. **Try/except around I/O only** — wrap database writes and external API calls. Do not wrap pure business logic.
3. **Log levels:**
   - `ERROR` — something broke and the operation stopped. Include the original error and context.
   - `WARN` — unexpected state but operation can continue.
   - `INFO` — significant lifecycle event (payment processed, user registered).
4. **User-facing messages** — translate technical errors into a human-readable response before they reach the API layer. The raw exception message must never be exposed directly to the client.

---

## Django Patterns

### Defining custom domain exceptions

```python
# apps/[app]/exceptions.py
class InsufficientStockError(Exception):
    """Raised when an order requests more stock than available."""
    def __init__(self, product_id: int, available: int, requested: int) -> None:
        self.product_id = product_id
        self.available = available
        self.requested = requested
        super().__init__(
            f"Product {product_id}: requested {requested}, available {available}."
        )


class OrderAlreadyProcessedError(Exception):
    """Raised when attempting to process an order that was already fulfilled."""
    pass
```

### Using exceptions in a service

```python
# apps/orders/services.py
import logging
from django.db import transaction
from .exceptions import InsufficientStockError, OrderAlreadyProcessedError
from .models import Order
from apps.inventory.selectors import get_stock_for_product

logger = logging.getLogger(__name__)


@transaction.atomic
def process_order(order_id: int) -> Order:
    order = Order.objects.select_for_update().get(id=order_id)

    if order.status == "processed":
        raise OrderAlreadyProcessedError(f"Order {order_id} is already processed.")

    stock = get_stock_for_product(order.product_id)
    if stock.quantity < order.quantity:
        raise InsufficientStockError(
            product_id=order.product_id,
            available=stock.quantity,
            requested=order.quantity,
        )

    try:
        order.mark_as_processed()  # DB write
        logger.info("Order %s processed successfully.", order_id)
    except Exception as e:
        logger.error("Failed to mark order %s as processed: %s", order_id, str(e), exc_info=True)
        raise

    return order
```

### Translating to HTTP in the view

```python
# apps/orders/views.py
from django.http import JsonResponse
from .services import process_order
from .exceptions import InsufficientStockError, OrderAlreadyProcessedError


def process_order_view(request, order_id: int) -> JsonResponse:
    try:
        order = process_order(order_id)
        return JsonResponse({"status": "ok", "order_id": order.id})
    except InsufficientStockError as e:
        return JsonResponse({"error": "Insufficient stock for this product."}, status=400)
    except OrderAlreadyProcessedError:
        return JsonResponse({"error": "This order has already been processed."}, status=409)
```

---

## NestJS Patterns

### Defining custom domain exceptions

```typescript
// common/exceptions/domain.exceptions.ts
import { BadRequestException, ConflictException } from '@nestjs/common';

export class InsufficientStockException extends BadRequestException {
    constructor(productId: number, available: number, requested: number) {
        super({
            code: 'INSUFFICIENT_STOCK',
            message: `Product ${productId}: requested ${requested}, available ${available}.`,
        });
    }
}

export class OrderAlreadyProcessedException extends ConflictException {
    constructor(orderId: number) {
        super({
            code: 'ORDER_ALREADY_PROCESSED',
            message: `Order ${orderId} has already been processed.`,
        });
    }
}
```

### Using exceptions in a service

```typescript
// modules/orders/orders.service.ts
import { Injectable, Logger } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Order } from './entities/order.entity';
import {
    InsufficientStockException,
    OrderAlreadyProcessedException,
} from '../../common/exceptions/domain.exceptions';

@Injectable()
export class OrdersService {
    private readonly logger = new Logger(OrdersService.name);

    constructor(
        @InjectRepository(Order)
        private readonly orderRepo: Repository<Order>,
    ) {}

    async processOrder(orderId: number): Promise<Order> {
        const order = await this.orderRepo.findOneOrFail({ where: { id: orderId } });

        if (order.status === 'processed') {
            throw new OrderAlreadyProcessedException(orderId);
        }

        if (order.stockAvailable < order.quantity) {
            throw new InsufficientStockException(
                order.productId,
                order.stockAvailable,
                order.quantity,
            );
        }

        try {
            order.status = 'processed';
            const saved = await this.orderRepo.save(order);
            this.logger.log(`Order ${orderId} processed successfully.`);
            return saved;
        } catch (error) {
            this.logger.error(`Failed to persist order ${orderId}`, (error as Error).stack);
            throw error;
        }
    }
}
```

> NestJS built-in exceptions (`NotFoundException`, `BadRequestException`, etc.) are automatically serialized to JSON by the global exception filter. Custom exceptions extending them inherit this behavior — no extra mapping needed in controllers.
