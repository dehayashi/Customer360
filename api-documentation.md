API Documentation — Customer360 CRM Intelligence

Versão: v1
Base URL (dev): https://localhost:5000/api

Autenticação: OAuth 2.0 com Bearer JWT.

Envie o header Authorization: Bearer <token> em todas as requisições.

⸻

🔐 Autenticação

Obter Token (exemplo)

POST /auth/token
Content-Type: application/json
{
  "clientId": "your-client-id",
  "clientSecret": "your-client-secret",
  "scope": "customer.read customer.write insights.read sync.write"
}

200

{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600
}


⸻

🧾 Headers Padrão
	•	Authorization: Bearer <token>
	•	Content-Type: application/json
	•	Accept: application/json

🔁 Paginação

Use query params: ?page=1&pageSize=50
Resposta inclui page, pageSize, total, totalPages.

🚦 Rate Limiting
	•	Limite: 600 req/min por IP/Client.
	•	Headers de resposta: X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset.

❗ Modelo de Erro

{
  "traceId": "d5b1a2e4-...",
  "code": "VALIDATION_ERROR",
  "message": "Campo 'email' inválido",
  "details": { "email": "Formato inválido" }
}


⸻

Endpoints

1) Customers

GET /customers

Lista clientes integrados.

Query params: page, pageSize, search, sort=createdAt|name, order=asc|desc

200

{
  "page": 1,
  "pageSize": 50,
  "total": 1234,
  "totalPages": 25,
  "data": [
    {
      "id": "cstm_01HXY...",
      "externalId": "CRM-000312",
      "name": "Maria Souza",
      "email": "maria@example.com",
      "phone": "+55 11 99999-0000",
      "lifecycleStage": "Lead",
      "engagementScore": 72,
      "createdAt": "2025-10-10T12:30:00Z",
      "updatedAt": "2025-10-12T09:15:00Z"
    }
  ]
}

GET /customers/{id}

Obtém um cliente pelo id.

200

{
  "id": "cstm_01HXY...",
  "externalId": "CRM-000312",
  "name": "Maria Souza",
  "email": "maria@example.com",
  "phone": "+55 11 99999-0000",
  "billingAddress": {
    "street": "Rua A, 123",
    "city": "São Paulo",
    "state": "SP",
    "country": "BR",
    "zip": "01234-567"
  },
  "lifecycleStage": "Customer",
  "engagementScore": 72,
  "tags": ["ecommerce", "vip"],
  "customFields": { "crmOwner": "usr_0192..." },
  "createdAt": "2025-10-10T12:30:00Z",
  "updatedAt": "2025-10-12T09:15:00Z"
}

POST /customers

Cria um novo cliente.

Body

{
  "name": "João Pereira",
  "email": "joao@example.com",
  "phone": "+55 21 98888-7777",
  "lifecycleStage": "Lead",
  "tags": ["campaign-2025"],
  "customFields": { "utmSource": "meta_ads" }
}

201

{ "id": "cstm_01HZ..." }

PATCH /customers/{id}

Atualiza campos de um cliente.

Body (exemplo)

{ "lifecycleStage": "Opportunity", "tags": ["partner-referral"] }

200

{ "updated": true }


⸻

2) Insights

GET /insights

Retorna métricas agregadas e tendências.

Query params: from=2025-10-01, to=2025-10-15, groupBy=day|week|month

200

{
  "period": { "from": "2025-10-01", "to": "2025-10-15", "groupBy": "day" },
  "kpis": {
    "totalLeads": 1240,
    "conversionRate": 0.123,
    "avgSalesCycleDays": 18.4
  },
  "engagementDistribution": [
    { "scoreRange": "0-25", "count": 132 },
    { "scoreRange": "26-50", "count": 481 },
    { "scoreRange": "51-75", "count": 410 },
    { "scoreRange": "76-100", "count": 217 }
  ],
  "trend": [
    { "date": "2025-10-01", "newLeads": 22, "opportunities": 6, "closedWon": 1 },
    { "date": "2025-10-02", "newLeads": 31, "opportunities": 9, "closedWon": 2 }
  ]
}


⸻

3) Sync

POST /sync

Dispara sincronização com conectores Azure (Functions/Logic Apps). Pode aceitar filtros.

Body (exemplos)

{ "entity": "customers", "mode": "incremental", "since": "2025-10-10T00:00:00Z" }

{ "entity": "opportunities", "mode": "full" }

202

{ "jobId": "sync_01HZ7G...", "status": "queued" }

GET /sync/{jobId}

Status do job de sincronização.

200

{
  "jobId": "sync_01HZ7G...",
  "status": "completed",
  "startedAt": "2025-10-15T19:02:10Z",
  "finishedAt": "2025-10-15T19:04:33Z",
  "processed": 124,
  "errors": []
}


⸻

4) Webhooks

Os webhooks notificam eventos de CRM para consumidores externos.
	•	Endpoint do cliente (exemplo): POST https://partner.example.com/hooks/customer360
	•	Assinatura HMAC no header X-Signature (chave configurada no Key Vault).

Evento: customer.updated

{
  "event": "customer.updated",
  "timestamp": "2025-10-15T19:20:10Z",
  "data": {
    "id": "cstm_01HXY...",
    "changes": {
      "lifecycleStage": { "from": "Lead", "to": "Customer" },
      "engagementScore": { "from": 64, "to": 78 }
    }
  }
}


⸻

🧱 Schemas

Customer

{
  "id": "string",
  "externalId": "string",
  "name": "string",
  "email": "string",
  "phone": "string",
  "billingAddress": {
    "street": "string", "city": "string", "state": "string", "country": "string", "zip": "string"
  },
  "lifecycleStage": "Lead|Opportunity|Customer|Churn",
  "engagementScore": 0,
  "tags": ["string"],
  "customFields": { "any": "value" },
  "createdAt": "datetime",
  "updatedAt": "datetime"
}


⸻

🧪 Examples

cURL

# Listar clientes
curl -H "Authorization: Bearer $TOKEN" \
     "https://localhost:5000/api/customers?page=1&pageSize=20&search=maria"

# Criar cliente
curl -X POST -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
     -d '{"name":"João","email":"joao@example.com"}' \
     https://localhost:5000/api/customers

.NET (HttpClient)

using var http = new HttpClient { BaseAddress = new Uri("https://localhost:5000/api/") };
http.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);

var res = await http.GetAsync("customers?page=1&pageSize=10");
res.EnsureSuccessStatusCode();
var json = await res.Content.ReadAsStringAsync();


⸻

📄 OpenAPI (trecho)

openapi: 3.0.3
info:
  title: Customer360 CRM Intelligence API
  version: v1
servers:
  - url: https://localhost:5000/api
paths:
  /customers:
    get:
      summary: List customers
      parameters:
        - in: query
          name: page
          schema: { type: integer, default: 1 }
        - in: query
          name: pageSize
          schema: { type: integer, default: 50, maximum: 200 }
      responses:
        '200':
          description: OK
  /insights:
    get:
      summary: Get insights
      responses:
        '200': { description: OK }


⸻

🔒 Security Notes
	•	Use short-lived JWTs e refresh tokens.
	•	Valide o audience/issuer e a assinatura do token.
	•	Ative CORS apenas para domínios confiáveis.
	•	Registre traceId em todos os logs e devolva-o nas respostas de erro.

⸻

✅ Checklist de Produção
	•	Rotinas de rotação de chaves no Key Vault
	•	Alertas no Application Insights
	•	Backups do SQL Server e DR plan
	•	Testes de carga ≥ p95 ≤ 300ms
	•	Política de throttling e retry exponencial nos clientes
