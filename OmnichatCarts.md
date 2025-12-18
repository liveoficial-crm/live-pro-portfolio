# API's Carrinhos Abandonados

## Visão Geral
API para gerenciamento de audiências de carrinho, permitindo criar, processar e consultar carrinhos abandonados, além de integração com Omnichat.

**Base URL:** `/api/admin/audiences`

---

## Endpoints

### 1. Criar Audiência de Carrinhos
Dispara o processamento de carrinhos para um período específico.

**Endpoint:** `POST /cart/create`

**Parâmetros:**
| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `date_start` | date | Sim | Data inicial do período |
| `date_end` | date | Sim | Data final do período |
| `sellers` | array | Não | Lista de vendedores para filtrar |
| `sellers.*` | string | Não | ID do vendedor |

**Exemplo de Request:**
```json
{
  "date_start": "2024-01-01",
  "date_end": "2024-01-31",
  "sellers": ["seller123", "seller456"],
}
```

**Response:** `200 OK`
```json
{
  "status": "success",
  "data": { ... }
}
```

---

### 2. Obter Carrinhos Processados
Retorna carrinhos que foram processados em um período específico.

**Endpoint:** `GET /cart/contacts`

**Parâmetros:**
| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `date_start` | date | Sim | Data inicial do período |
| `date_end` | date | Sim | Data final do período |
| `next_page` | string | Não | Token para próxima página |
| `batch_size` | integer | Não | Quantidade de registros (1-500, padrão: 100) |
| `seller` | string | Não | Filtrar por vendedor específico |

**Exemplo de Request:**
```
GET /cart/contacts?date_start=2024-01-01&date_end=2024-01-31&batch_size=100
```

**Response:** `200 OK`
```json
{
  "data": [ ... ],
  "next_page": "token_xyz",
  "total": 1500
}
```

---

### 3. Criar Audiência Omnichat
Dispara o processamento de carrinhos para integração com Omnichat.

**Endpoint:** `POST /cart/omnichat/create`

**Parâmetros:**
| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `date_start` | date | Sim | Data inicial do período |
| `date_end` | date | Sim | Data final do período |

**Exemplo de Request:**
```json
{
  "date_start": "2024-01-01",
  "date_end": "2024-01-31"
}
```

**Response:** `200 OK`
```json
{
  "status": "success",
  "data": { ... }
}
```

---

### 4. Status do Worker
Verifica o status de processamento dos workers.

**Endpoint:** `GET /cart/worker/status`

**Parâmetros:**
| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `key` | string | Sim | Chave do worker: `carts_processing` ou `omnichat_carts_processing` |

**Exemplo de Request:**
```
GET /cart/worker/status?key=carts_processing
```

**Response:** `200 OK`
```json
{
  "status": "running",
  "progress": 75,
  "details": { ... }
}
```

---

### 5. Total de Contatos Omnichat
Retorna o total de contatos em uma audiência específica do Omnichat.

**Endpoint:** `GET /omnichat/{audienceId}/contacts/total`

**Parâmetros de Rota:**
| Campo | Tipo | Descrição |
|-------|------|-----------|
| `audienceId` | string | ID da audiência no Omnichat |

**Parâmetros de Query:**
| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `batch_size` | integer | Não | Tamanho do lote (1-500, padrão: 100) |

**Exemplo de Request:**
```
GET /omnichat/aud_123456/contacts/total?batch_size=100
```

**Response:** `200 OK`
```json
{
  "total": 5000,
  "audience_id": "aud_123456"
}
```

---

### 6. Listar Contatos Omnichat
Lista os contatos de uma audiência específica do Omnichat com paginação.

**Endpoint:** `GET /omnichat/{audienceId}/contacts`

**Parâmetros de Rota:**
| Campo | Tipo | Descrição |
|-------|------|-----------|
| `audienceId` | string | ID da audiência no Omnichat |

**Parâmetros de Query:**
| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `page_size` | integer | Não | Quantidade por página (1-500, padrão: 100) |
| `page_token` | string | Não | Token para próxima página |

**Exemplo de Request:**
```
GET /omnichat/aud_123456/contacts?page_size=50&page_token=token_abc
```

**Response:** `200 OK`
```json
{
  "contacts": [ ... ],
  "next_page_token": "token_def",
  "total": 5000
}
```

---

## Códigos de Status HTTP

| Código | Descrição |
|--------|-----------|
| `200` | Requisição bem-sucedida |
| `400` | Erro de validação nos parâmetros |
| `401` | Não autenticado |
| `403` | Não autorizado |
| `404` | Recurso não encontrado |
| `422` | Erro de validação |
| `500` | Erro interno do servidor |

---

## Notas
- Todas as datas devem estar no formato `Y-m-d` (ex: 2024-01-31)
- Os valores de `batch_size` e `page_size` são limitados entre 1 e 500
- A paginação é feita através dos tokens `next_page` e `page_token`
- Para workers válidos, usar: `carts_processing` ou `omnichat_carts_processing`
