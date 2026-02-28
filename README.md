# 💳 PaymentGateway API

> API REST de alta disponibilidade para processamento de pagamentos, construída com **Clean Architecture**, **DDD** e **CQRS** em .NET 8.

![.NET](https://img.shields.io/badge/.NET-8.0-512BD4?style=flat-square&logo=dotnet)
![ASP.NET Core](https://img.shields.io/badge/ASP.NET_Core-8.0-512BD4?style=flat-square)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-336791?style=flat-square&logo=postgresql)
![Redis](https://img.shields.io/badge/Redis-7-DC382D?style=flat-square&logo=redis)
![RabbitMQ](https://img.shields.io/badge/RabbitMQ-3.12-FF6600?style=flat-square&logo=rabbitmq)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat-square&logo=docker)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)
![Coverage](https://img.shields.io/badge/Coverage-≥85%25-brightgreen?style=flat-square)

---

## 📋 Índice

- [Sobre o Projeto](#-sobre-o-projeto)
- [Stack de Tecnologias](#-stack-de-tecnologias)
- [Arquitetura](#-arquitetura)
- [Endpoints da API](#-endpoints-da-api)
- [Exemplos de Uso](#-exemplos-de-uso)
- [Padrões e Diferenciais Técnicos](#-padrões-e-diferenciais-técnicos)
- [Como Rodar Localmente](#-como-rodar-localmente)
- [Testes](#-testes)
- [CI/CD](#-cicd)
- [Configurações](#-configurações)

---

## 🚀 Sobre o Projeto

O **PaymentGateway API** é uma API REST que simula um gateway de pagamentos completo. Desenvolvido com foco em demonstrar domínio de arquitetura de software, boas práticas de engenharia e padrões avançados do ecossistema .NET.

O sistema processa transações com **cartão de crédito**, **débito** e **PIX**, gerencia estornos, entrega notificações via webhook e garante consistência dos dados através do **Outbox Pattern**. Toda a stack foi pensada para escalar horizontalmente em produção.

### ✨ Destaques

- 🏗️ Arquitetura em 4 camadas com separação total de responsabilidades
- 🔑 Idempotência nativa em todas as mutações (seguro para retries automáticos)
- ⚡ Circuit Breaker e Retry com Polly para chamadas externas
- 📬 Eventos de domínio com Outbox Pattern + RabbitMQ
- 📊 Observabilidade completa com OpenTelemetry, Serilog e Seq
- 🧪 Testes unitários, de integração e E2E com Testcontainers

---

## 🛠️ Stack de Tecnologias

| Camada | Tecnologias |
|---|---|
| **Runtime** | ASP.NET Core 8 · .NET 8 · C# 12 |
| **ORM / Banco** | Entity Framework Core 8 · PostgreSQL 16 |
| **Cache** | Redis 7 · IDistributedCache |
| **Mensageria** | RabbitMQ · MediatR (CQRS) |
| **Validação** | FluentValidation · ErrorOr (Result Pattern) |
| **Resiliência** | Polly (Retry · Circuit Breaker · Timeout) |
| **Autenticação** | JWT Bearer · Refresh Token · ASP.NET Identity |
| **Observabilidade** | OpenTelemetry · Serilog · Seq · Health Checks |
| **Testes** | xUnit · Moq · FluentAssertions · Testcontainers |
| **Infra** | Docker · Docker Compose · GitHub Actions |
| **Docs** | Swagger · Scalar · XML Comments |

---

## 🏛️ Arquitetura

O projeto segue **Clean Architecture** com 4 camadas bem definidas, garantindo separação de responsabilidades e facilidade de manutenção:

```
┌─────────────────────────────────────────┐
│               API Layer                 │  ← Controllers, Middleware, Contracts
├─────────────────────────────────────────┤
│          Application Layer              │  ← Commands, Queries, Handlers, Behaviors
├─────────────────────────────────────────┤
│           Domain Layer                  │  ← Entities, Value Objects, Events, Errors
├─────────────────────────────────────────┤
│        Infrastructure Layer             │  ← EF Core, Redis, RabbitMQ, Outbox
└─────────────────────────────────────────┘
```

### Pipeline CQRS

Todo Command passa pelo seguinte pipeline antes de chegar ao Handler:

```
Command ──► ValidationBehavior ──► LoggingBehavior ──► IdempotencyBehavior ──► Handler
```

Queries utilizam projeções otimizadas com SQL direto via Dapper para máxima performance em leituras.

### Estrutura de Pastas

<details>
<summary>Ver estrutura completa</summary>

```
PaymentGateway/
│
├── src/
│   ├── PaymentGateway.Domain/
│   │   ├── Entities/          # Payment, Refund, Merchant
│   │   ├── Enums/             # PaymentStatus, PaymentMethod
│   │   ├── Events/            # PaymentCreatedEvent, PaymentProcessedEvent...
│   │   ├── Errors/            # DomainErrors
│   │   ├── Repositories/      # Interfaces de repositório
│   │   └── ValueObjects/      # Money, CardInfo, Address
│   │
│   ├── PaymentGateway.Application/
│   │   ├── Payments/
│   │   │   ├── Commands/      # CreatePayment, CapturePayment, RefundPayment
│   │   │   └── Queries/       # GetPaymentById, ListPayments
│   │   ├── Common/
│   │   │   ├── Behaviors/     # ValidationBehavior, LoggingBehavior, IdempotencyBehavior
│   │   │   └── Interfaces/    # IPaymentProcessor, IWebhookService
│   │   └── DependencyInjection.cs
│   │
│   ├── PaymentGateway.Infrastructure/
│   │   ├── Persistence/       # AppDbContext, Repositories, Configurations
│   │   ├── Outbox/            # OutboxMessage, OutboxProcessor
│   │   ├── Processors/        # StripePaymentProcessor
│   │   ├── Caching/           # RedisCacheService
│   │   ├── Webhooks/          # WebhookService
│   │   └── DependencyInjection.cs
│   │
│   └── PaymentGateway.Api/
│       ├── Controllers/       # Payments, Refunds, Merchants, Webhooks
│       ├── Contracts/         # Requests e Responses
│       ├── Middleware/        # ErrorHandling, Idempotency
│       ├── Program.cs
│       └── appsettings.json
│
├── tests/
│   ├── PaymentGateway.Domain.Tests/
│   ├── PaymentGateway.Application.Tests/
│   └── PaymentGateway.Integration.Tests/
│
├── docker-compose.yml
└── .github/workflows/
    ├── ci.yml
    └── cd.yml
```

</details>

---

## 📡 Endpoints da API

> Todos os endpoints (exceto `/api/auth`) exigem `Authorization: Bearer <token>`.

### 🔐 Autenticação — `/api/auth`

| Método | Endpoint | Descrição | Auth |
|---|---|---|---|
| `POST` | `/api/auth/register` | Cadastro de novo merchant | ✗ |
| `POST` | `/api/auth/login` | Login e geração de JWT + Refresh Token | ✗ |
| `POST` | `/api/auth/refresh` | Renovar access token | ✗ |
| `POST` | `/api/auth/revoke` | Revogar refresh token | ✓ |

### 💳 Pagamentos — `/api/payments`

| Método | Endpoint | Descrição | Auth |
|---|---|---|---|
| `POST` | `/api/payments` | Criar nova transação de pagamento | ✓ |
| `GET` | `/api/payments` | Listar pagamentos com paginação e filtros | ✓ |
| `GET` | `/api/payments/{id}` | Buscar pagamento por ID | ✓ |
| `POST` | `/api/payments/{id}/capture` | Capturar pagamento pré-autorizado | ✓ |
| `POST` | `/api/payments/{id}/cancel` | Cancelar pagamento pendente | ✓ |
| `GET` | `/api/payments/{id}/events` | Histórico de eventos do pagamento | ✓ |

### 💸 Estornos — `/api/refunds`

| Método | Endpoint | Descrição | Auth |
|---|---|---|---|
| `POST` | `/api/payments/{id}/refunds` | Solicitar estorno total ou parcial | ✓ |
| `GET` | `/api/payments/{id}/refunds` | Listar estornos de um pagamento | ✓ |
| `GET` | `/api/refunds/{id}` | Buscar estorno por ID | ✓ |

### 🏪 Merchants — `/api/merchants`

| Método | Endpoint | Descrição | Auth |
|---|---|---|---|
| `GET` | `/api/merchants/me` | Dados do merchant autenticado | ✓ |
| `PUT` | `/api/merchants/me` | Atualizar dados do merchant | ✓ |
| `GET` | `/api/merchants/me/stats` | Estatísticas de transações | ✓ |

### 🪝 Webhooks — `/api/webhooks`

| Método | Endpoint | Descrição | Auth |
|---|---|---|---|
| `POST` | `/api/webhooks` | Registrar URL de webhook | ✓ |
| `GET` | `/api/webhooks` | Listar webhooks cadastrados | ✓ |
| `DELETE` | `/api/webhooks/{id}` | Remover webhook | ✓ |
| `GET` | `/api/webhooks/{id}/logs` | Logs de disparos do webhook | ✓ |

### 🏥 Saúde e Diagnóstico

| Método | Endpoint | Descrição | Auth |
|---|---|---|---|
| `GET` | `/health` | Health check completo (DB, Redis, RabbitMQ) | ✗ |
| `GET` | `/health/live` | Liveness probe (Kubernetes) | ✗ |
| `GET` | `/health/ready` | Readiness probe (Kubernetes) | ✗ |

---

## 💡 Exemplos de Uso

### `POST /api/payments` — Criar Pagamento

**Request:**
```json
{
  "amount": 150.00,
  "currency": "BRL",
  "paymentMethod": "CreditCard",
  "capture": true,
  "idempotencyKey": "order-12345-v1",
  "card": {
    "number": "4111111111111111",
    "holderName": "João Silva",
    "expirationMonth": 12,
    "expirationYear": 2027,
    "cvv": "123"
  },
  "customer": {
    "name": "João Silva",
    "email": "joao@email.com",
    "document": "123.456.789-00"
  },
  "metadata": {
    "orderId": "12345",
    "productName": "Assinatura Premium"
  }
}
```

**Response `201 Created`:**
```json
{
  "id": "pay_01J8XKZP3M7N4Q5R6T8V9W0Y",
  "status": "Captured",
  "amount": 150.00,
  "currency": "BRL",
  "paymentMethod": "CreditCard",
  "authorizationCode": "AUTH-789XYZ",
  "idempotencyKey": "order-12345-v1",
  "createdAt": "2024-01-15T10:30:00Z",
  "capturedAt": "2024-01-15T10:30:01Z",
  "_links": {
    "self": "/api/payments/pay_01J8XKZP3M7N4Q5R6T8V9W0Y",
    "refund": "/api/payments/pay_01J8XKZP3M7N4Q5R6T8V9W0Y/refunds",
    "events": "/api/payments/pay_01J8XKZP3M7N4Q5R6T8V9W0Y/events"
  }
}
```

### `POST /api/payments/{id}/refunds` — Estorno Parcial

**Request:**
```json
{
  "amount": 50.00,
  "reason": "Produto com defeito"
}
```

**Response `201 Created`:**
```json
{
  "id": "ref_02K9YLAQ4N8O5R6S7U9X1Z2W",
  "paymentId": "pay_01J8XKZP3M7N4Q5R6T8V9W0Y",
  "amount": 50.00,
  "status": "Succeeded",
  "reason": "Produto com defeito",
  "createdAt": "2024-01-16T09:00:00Z"
}
```

### Resposta de Erro — `422 Unprocessable Entity`

```json
{
  "type": "https://tools.ietf.org/html/rfc9457",
  "title": "Validation Error",
  "status": 422,
  "errors": {
    "card.number": ["Número de cartão inválido."],
    "amount": ["O valor mínimo é R$ 1,00."]
  }
}
```

---

## ⚙️ Padrões e Diferenciais Técnicos

### 🔑 Idempotência

Toda requisição `POST` aceita um header `Idempotency-Key` ou o campo `idempotencyKey` no body. Requisições duplicadas com a mesma chave retornam a resposta original sem reprocessar a transação, garantindo segurança em cenários de retry automático.

```
POST /api/payments
Idempotency-Key: order-12345-v1
```

Respostas servidas do cache incluem o header `Idempotency-Replayed: true`.

---

### 📬 Outbox Pattern

Eventos de domínio são persistidos **atomicamente** junto com a transação no banco de dados. Um `BackgroundService` independente processa esses eventos de forma assíncrona:

```
[Transação DB] ──► [OutboxMessages] ──► [OutboxProcessor] ──► [RabbitMQ] ──► [Consumers]
```

Isso garante que nenhum evento seja perdido mesmo em caso de falha após o commit da transação.

---

### 🔄 Circuit Breaker com Polly

Chamadas ao processador externo são protegidas por um pipeline de resiliência completo:

```csharp
services.AddHttpClient<IPaymentProcessor, StripePaymentProcessor>()
    .AddPolicyHandler(GetRetryPolicy())           // 3 tentativas, backoff exponencial
    .AddPolicyHandler(GetCircuitBreakerPolicy())  // Abre após 5 falhas consecutivas
    .AddPolicyHandler(GetTimeoutPolicy());        // Timeout de 10s por chamada
```

---

### 🎯 Result Pattern com ErrorOr

Nenhuma exceção é lançada para fluxos de negócio. Erros de domínio são tipados e mapeados para HTTP automaticamente:

```csharp
public async Task<ErrorOr<PaymentResult>> Handle(CreatePaymentCommand command, ...)
{
    if (command.Amount <= 0)
        return DomainErrors.Payment.InvalidAmount; // Mapeado para 422

    // ...
    return new PaymentResult(payment); // 201 Created
}
```

---

## 🖥️ Como Rodar Localmente

### Pré-requisitos

- [.NET 8 SDK](https://dotnet.microsoft.com/download)
- [Docker](https://www.docker.com/) e Docker Compose

### Passos

```bash
# 1. Clonar o repositório
git clone https://github.com/seu-usuario/payment-gateway
cd payment-gateway

# 2. Subir a infraestrutura (PostgreSQL, Redis, RabbitMQ, Seq)
docker-compose up -d

# 3. Aplicar migrations
dotnet ef database update \
  --project src/PaymentGateway.Infrastructure \
  --startup-project src/PaymentGateway.Api

# 4. Rodar a API
dotnet run --project src/PaymentGateway.Api
```

### Serviços Disponíveis

| Serviço | URL |
|---|---|
| API | http://localhost:5000 |
| Swagger UI | http://localhost:5000/swagger |
| Scalar | http://localhost:5000/scalar |
| Seq (logs) | http://localhost:5341 |
| RabbitMQ Management | http://localhost:15672 |

---

## 🧪 Testes

```bash
# Todos os testes
dotnet test

# Somente unitários
dotnet test tests/PaymentGateway.Domain.Tests
dotnet test tests/PaymentGateway.Application.Tests

# Integração (requer Docker)
dotnet test tests/PaymentGateway.Integration.Tests

# Com relatório de cobertura
dotnet test --collect:"XPlat Code Coverage"
reportgenerator -reports:**/coverage.cobertura.xml -targetdir:coverage -reporttypes:Html
```

### Cobertura por Camada

| Camada | Meta |
|---|---|
| Domain | ≥ 90% |
| Application | ≥ 85% |
| Infrastructure | ≥ 70% |
| **Total** | **≥ 85%** |

Os testes de integração utilizam **Testcontainers** para subir instâncias reais de PostgreSQL e Redis durante o CI, garantindo confiabilidade sem mocks de infraestrutura.

---

## 🔁 CI/CD

### Pipeline CI (`ci.yml`)

Executado em todo push e pull request:

1. Restore de dependências
2. `dotnet format --verify-no-changes`
3. Build
4. Testes unitários e de integração
5. Geração de relatório de cobertura (Codecov)
6. Gate de qualidade: bloqueia merge se coverage < 85%

### Pipeline CD (`cd.yml`)

| Gatilho | Ação |
|---|---|
| Push em `main` | Build Docker + push GHCR + deploy em **staging** |
| Tag `v*.*.*` | Deploy em **produção** |

---

## ⚙️ Configurações

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Database=paymentdb;Username=postgres;Password=postgres",
    "Redis": "localhost:6379"
  },
  "Jwt": {
    "Secret": "your-super-secret-key-minimum-256-bits",
    "Issuer": "PaymentGateway",
    "Audience": "PaymentGateway.Client",
    "ExpiryMinutes": 60,
    "RefreshTokenExpiryDays": 7
  },
  "PaymentProcessor": {
    "Provider": "Stripe",
    "ApiKey": "sk_test_...",
    "TimeoutSeconds": 10
  },
  "RabbitMQ": {
    "Host": "localhost",
    "Username": "guest",
    "Password": "guest"
  },
  "Outbox": {
    "IntervalSeconds": 5,
    "BatchSize": 20
  }
}
```

> ⚠️ Nunca commite segredos reais no repositório. Use variáveis de ambiente ou um gerenciador de segredos em produção.

---

## 📄 Licença

Distribuído sob a licença MIT. Veja [LICENSE](LICENSE) para mais detalhes.

---

<div align="center">
  <p>Feito com ❤️ e .NET 8</p>
</div>
