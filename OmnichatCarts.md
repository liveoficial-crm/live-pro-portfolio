# Documentação da API - CartAudienceController

## Visão Geral

Esta API gerencia operações relacionadas a carrinhos de compras e integrações com o serviço Omnichat. A API permite processar carrinhos, obter contatos e monitorar o status de workers assíncronos.

**Base URL:** `/api`  
**Controller:** `App\Http\Controllers\Api\CartAudienceController`

---

## Endpoints

### 1. Processar Carrinhos (dispatchCarts)

**Endpoint:** `POST /api/cart-audience/dispatch-carts`

**Descrição:** Processa carrinhos de compras em um período específico, com opção de sincronização.

**Parâmetros:**

| Campo | Tipo | Obrigatório | Validação | Descrição |
|-------|------|-------------|-----------|-----------|
| `date_start` | string | ✅ | ISO Date (YYYY-MM-DD) | Data de início do período |
| `date_end` | string | ✅ | ISO Date (YYYY-MM-DD) | Data de fim do período |
| `sellers` | array | ❌ | array de strings | Lista de vendedores específicos |
| `sellers.*` | string | ❌ | string | Identificador do vendedor |
| `sync` | boolean | ❌ | boolean | Processamento síncrono (padrão: false) |

**Exemplo de Requisição:**
```json
{
  "date_start": "2025-12-01",
  "date_end": "2025-12-18",
  "sellers": ["vendedor1", "vendedor2"],
  "sync": false
}
```

**Resposta:** JSON com resultado do processamento.

---

### 2. Obter Carrinhos Processados (getProcessedCarts)

**Endpoint:** `GET /api/cart-audience/processed-carts`

**Descrição:** Recupera carrinhos já processados com paginação.

**Parâmetros:**

| Campo | Tipo | Obrigatório | Validação | Descrição |
|-------|------|-------------|-----------|-----------|
| `date_start` | string | ✅ | ISO Date (YYYY-MM-DD) | Data de início do período |
| `date_end` | string | ✅ | ISO Date (YYYY-MM-DD) | Data de fim do período |
| `next_page` | string | ❌ | string | Token de paginação |
| `batch_size` | integer | ❌ | 1-500 | Tamanho do lote (padrão: 100) |
| `seller` | string | ❌ | string | Filtrar por vendedor específico |

**Exemplo de Requisição:**
```json
{
  "date_start": "2025-12-01",
  "date_end": "2025-12-18",
  "batch_size": 50,
  "seller": "vendedor1"
}
```

**Resposta:** JSON com carrinhos processados e informações de paginação.

---

### 3. Processar Carrinhos Omnichat (dispatchOmnichatCarts)

**Endpoint:** `POST /api/cart-audience/dispatch-omnichat-carts`

**Descrição:** Processa especificamente carrinhos para integração com Omnichat.

**Parâmetros:**

| Campo | Tipo | Obrigatório | Validação | Descrição |
|-------|------|-------------|-----------|-----------|
| `date_start` | string | ✅ | ISO Date (YYYY-MM-DD) | Data de início do período |
| `date_end` | string | ✅ | ISO Date (YYYY-MM-DD) | Data de fim do período |

**Exemplo de Requisição:**
```json
{
  "date_start": "2025-12-01",
  "date_end": "2025-12-18"
}
```

**Resposta:** JSON com resultado do processamento para Omnichat.

---

### 4. Status do Worker (getWorkerStatus)

**Endpoint:** `GET /api/cart-audience/worker-status`

**Descrição:** Consulta o status de workers assíncronos específicos.

**Parâmetros:**

| Campo | Tipo | Obrigatório | Validação | Descrição |
|-------|------|-------------|-----------|-----------|
| `key` | string | ✅ | `carts_processing` ou `omnichat_carts_processing` | Identificador do worker |

**Exemplo de Requisição:**
```json
{
  "key": "carts_processing"
}
```

**Resposta:** JSON com status do worker consultado.

---

### 5. Total de Contatos (getTotalContacts)

**Endpoint:** `GET /api/cart-audience/contacts/total/{audienceId}`

**Descrição:** Obtém o total de contatos de uma audiência específica.

**Parâmetros:**

| Campo | Tipo | Obrigatório | Validação | Descrição |
|-------|------|-------------|-----------|-----------|
| `audienceId` | string | ✅ | string | ID da audiência |
| `batch_size` | integer | ❌ | 1-500 | Tamanho do lote (padrão: 100) |

**Exemplo de Requisição:**
```json
{
  "batch_size": 100
}
```

**Resposta:** JSON com total de contatos na audiência.

---

### 6. Obter Contatos (getContacts)

**Endpoint:** `GET /api/cart-audience/contacts/{audienceId}`

**Descrição:** Recupera contatos de uma audiência específica com paginação.

**Parâmetros:**

| Campo | Tipo | Obrigatório | Validação | Descrição |
|-------|------|-------------|-----------|-----------|
| `audienceId` | string | ✅ | string | ID da audiência |
| `page_size` | integer | ❌ | 1-500 | Tamanho da página (padrão: 100) |
| `page_token` | string | ❌ | string | Token de paginação |

**Exemplo de Requisição:**
```json
{
  "page_size": 50,
  "page_token": "next_page_token_here"
}
```

**Resposta:** JSON com lista de contatos e informações de paginação.

---

## Serviços Utilizados

### CartsAudienceService
- Gerencia processamento de carrinhos
- Integração com Redis para status de workers
- Operações de carrinhos para Omnichat

### OmnichatApiService
- Integração com API externa do Omnichat
- Gerenciamento de audiências e contatos
- Paginação de resultados

---

## Códigos de Status HTTP

| Código | Descrição |
|--------|-----------|
| 200 | Sucesso |
| 400 | Requisição inválida (erro de validação) |
| 422 | Dados fornecidos não passaram na validação |
| 500 | Erro interno do servidor |

---

## Exemplos de Uso

### Processar Carrinhos
```bash
curl -X POST /api/cart-audience/dispatch-carts \
  -H "Content-Type: application/json" \
  -d '{
    "date_start": "2025-12-01",
    "date_end": "2025-12-18",
    "sellers": ["vendedor1"],
    "sync": false
  }'
```

### Consultar Status do Worker
```bash
curl -X GET "/api/cart-audience/worker-status?key=carts_processing"
```

### Obter Contatos
```bash
curl -X GET "/api/cart-audience/contacts/audience123?page_size=50"
```

---

## Observações Importantes

1. **Validação:** Todos os parâmetros são validados conforme as regras especificadas
2. **Paginação:** Endpoints de listagem suportam paginação via tokens
3. **Processamento Assíncrono:** Carrinhos são processados de forma assíncrona por padrão
4. **Cache:** Status de workers é armazenado em Redis
5. **Integração Externa:** Contatos são obtidos via API externa do Omnichat

---

*Documentação gerada em: 2025-12-18*  
*Autor: MiniMax Agent*
