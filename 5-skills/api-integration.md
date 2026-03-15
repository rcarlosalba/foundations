# Skill: Resilient API Integration

**When to use:** Any time the app connects to an external HTTP service (payment gateway, email provider, third-party data API, etc.).

---

## Procedure

1. **Secrets via config** — read API keys from environment using the active stack's standard (`decouple.config()` for Django, `ConfigService` for NestJS). Never hardcode.
2. **Dedicated wrapper class** — do not call external URLs from services or controllers directly. Create one wrapper per external API.
3. **Timeout** — always set a request timeout (5s default).
4. **Retry with backoff** — retry on 5xx and network errors only. Never retry on 4xx (those are client errors). Use exponential backoff: 1s → 2s → 4s.
5. **Mock for tests** — the wrapper must be mockable. Inject it as a dependency so tests can swap it.

---

## Django Pattern (httpx + tenacity)

```python
# services/external/[provider]_client.py
import httpx
from tenacity import retry, stop_after_attempt, wait_exponential, retry_if_exception_type
from decouple import config
import logging

logger = logging.getLogger(__name__)


class ProviderAPIError(Exception):
    """Raised when the external provider returns a non-retryable error."""
    pass


class ProviderClient:
    def __init__(self) -> None:
        self._client = httpx.Client(
            base_url=config("PROVIDER_API_URL"),
            headers={"Authorization": f"Bearer {config('PROVIDER_API_KEY')}"},
            timeout=5.0,
        )

    @retry(
        stop=stop_after_attempt(3),
        wait=wait_exponential(multiplier=1, min=1, max=8),
        retry=retry_if_exception_type(httpx.HTTPStatusError),
        reraise=True,
    )
    def get_resource(self, resource_id: int) -> dict:
        try:
            response = self._client.get(f"/resources/{resource_id}")
            response.raise_for_status()
            return response.json()
        except httpx.HTTPStatusError as e:
            if e.response.status_code < 500:
                raise ProviderAPIError(f"Client error: {e.response.status_code}") from e
            logger.warning("Provider 5xx on attempt, will retry. Status: %s", e.response.status_code)
            raise
        except httpx.TimeoutException as e:
            logger.error("Provider request timed out for resource %s", resource_id)
            raise ProviderAPIError("Request timed out") from e

    def close(self) -> None:
        self._client.close()
```

**Test mock (pytest):**
```python
# tests/test_some_service.py
from unittest.mock import MagicMock, patch

def test_process_uses_provider_data(db):
    mock_client = MagicMock()
    mock_client.get_resource.return_value = {"id": 1, "status": "active"}

    with patch("apps.orders.services.ProviderClient", return_value=mock_client):
        result = process_order(order_id=1)

    mock_client.get_resource.assert_called_once_with(1)
    assert result.status == "active"
```

---

## NestJS Pattern (HttpService + RxJS)

```typescript
// modules/[provider]/[provider].client.ts
import { Injectable, ServiceUnavailableException } from '@nestjs/common';
import { HttpService } from '@nestjs/axios';
import { ConfigService } from '@nestjs/config';
import { firstValueFrom, retry, catchError, timer } from 'rxjs';
import { AxiosError } from 'axios';

@Injectable()
export class ProviderClient {
    constructor(
        private readonly httpService: HttpService,
        private readonly config: ConfigService,
    ) {}

    async getResource(resourceId: number): Promise<ResourceDto> {
        const baseUrl = this.config.get<string>('PROVIDER_API_URL');

        return firstValueFrom(
            this.httpService
                .get<ResourceDto>(`${baseUrl}/resources/${resourceId}`, {
                    headers: {
                        Authorization: `Bearer ${this.config.get('PROVIDER_API_KEY')}`,
                    },
                    timeout: 5000,
                })
                .pipe(
                    retry({
                        count: 3,
                        delay: (_, retryCount) => timer(2 ** retryCount * 1000),
                    }),
                    catchError((error: AxiosError) => {
                        throw new ServiceUnavailableException(
                            `Provider error: ${error.message}`,
                        );
                    }),
                ),
        ).then((res) => res.data);
    }
}
```

**Test mock (Jest):**
```typescript
// modules/[provider]/[provider].client.spec.ts
const mockHttpService = { get: jest.fn() };

const client = new ProviderClient(
    mockHttpService as any,
    { get: () => 'mock-value' } as any,
);

it('returns resource on success', async () => {
    mockHttpService.get.mockReturnValue(of({ data: { id: 1, status: 'active' } }));
    const result = await client.getResource(1);
    expect(result.status).toBe('active');
});
```
