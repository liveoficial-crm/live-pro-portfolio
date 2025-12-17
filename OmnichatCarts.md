# Resumo das Rotas - CartAudienceController

## Estrutura de Rotas

```php
Route::prefix('audiences')->group(function () {
    Route::post('/cart/create', [CartAudienceController::class, 'dispatchCarts']);
    Route::get('/cart/contacts', [CartAudienceController::class, 'getProcessedCarts']);
    Route::post('/cart/omnichat/create', [CartAudienceController::class, 'dispatchOmnichatCarts']);
    
    Route::get('cart/worker/status', [CartAudienceController::class, 'getWorkerStatus']);

    Route::get('/omnichat/{audienceId}/contacts/total', [CartAudienceController::class, 'getTotalContacts']);
    Route::get('/omnichat/{audienceId}/contacts', [CartAudienceController::class, 'getContacts']);
});
```

## Lista de Endpoints

| Método | Rota | Controller | Função | Descrição |
|--------|------|------------|--------|-----------|
| `POST` | `/api/audiences/cart/create` | `dispatchCarts` | Processar Carrinhos | Cria processo de carrinhos para período específico |
| `GET` | `/api/audiences/cart/contacts` | `getProcessedCarts` | Carrinhos Processados | Lista carrinhos já processados |
| `POST` | `/api/audiences/cart/omnichat/create` | `dispatchOmnichatCarts` | Carrinhos Omnichat | Processa carrinhos para integração Omnichat |
| `GET` | `/api/audiences/cart/worker/status` | `getWorkerStatus` | Status Worker | Consulta status de workers assíncronos |
| `GET` | `/api/audiences/omnichat/{audienceId}/contacts/total` | `getTotalContacts` | Total Contatos | Obtém total de contatos de audiência |
| `GET` | `/api/audiences/omnichat/{audienceId}/contacts` | `getContacts` | Contatos | Lista contatos de audiência com paginação |

## Padrões de URL

- **Prefix:** `/api/audiences`
- **Carrinhos:** `/cart/*`
- **Workers:** `/cart/worker/*`
- **Omnichat:** `/omnichat/{audienceId}/*`
- **Contatos:** `/contacts` ou `/contacts/total`

---

