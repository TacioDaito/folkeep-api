# Folkeep

Plataforma B2B de análise de pessoas para gestão e análise de dados de colaboradores, com foco em relatórios de headcount, turnover e distribuição organizacional.

---

## Visão Geral

Sistema multitenancy onde empresas gerenciam seus colaboradores e extraem insights estratégicos para o RH — sem ser um HRIS completo. O objetivo é transformar dados estruturados de pessoas em relatórios acionáveis.

---

## Stack

| Camada | Tecnologia |
|---|---|
| Backend | Laravel 12+ (API REST) |
| Frontend | React + TypeScript (SPA desacoplado) |
| Banco relacional | PostgreSQL |
| Logs / Eventos | MongoDB |
| Autenticação | Laravel Sanctum (token stateless) |

---

## Arquitetura

```
┌─────────────────────┐        HTTP / JSON        ┌──────────────────────┐
│                     │  ──────────────────────►  │                      │
│   React + TypeScript│                           │    Laravel API        │
│   (SPA)             │  ◄──────────────────────  │    (REST)             │
│                     │                           │                       │
└─────────────────────┘                           └──────────┬────────────┘
                                                             │
                                              ┌──────────────┴──────────────┐
                                              │                             │
                                     ┌────────▼────────┐        ┌──────────▼───────┐
                                     │   PostgreSQL     │        │     MongoDB       │
                                     │   (dados         │        │   (event logs /   │
                                     │    relacionais)  │        │    histórico)     │
                                     └─────────────────┘        └──────────────────┘
```

O frontend é um projeto independente que consome a API via token Sanctum. Não há acoplamento via Inertia ou Blade — os contratos de API são a única interface entre os dois projetos.

---

## Funcionalidades Planejadas

**MVP**
- Autenticação por token com suporte a multitenancy
- Cadastro de colaboradores com departamento, cargo, tipo de contrato e data de admissão
- Histórico de eventos do colaborador (promoções, mudanças de cargo, alterações salariais) armazenado no MongoDB
- Relatório de headcount por departamento
- Relatório de turnover mensal/trimestral
- Distribuição salarial por faixa e nível

**Expansões futuras**
- Notificações em tempo real (Laravel Reverb)
- Exportação de relatórios em PDF
- Upload de documentos com S3
- CI/CD com GitHub Actions
- Métricas de diversidade e inclusão

---

## Decisões Técnicas Relevantes

**Histórico de colaboradores com Slowly Changing Dimensions**
Colaboradores mudam de cargo, salário e departamento ao longo do tempo. Para viabilizar relatórios históricos precisos, as mudanças são registradas com `valid_from` / `valid_to` no PostgreSQL — não apenas o estado atual.

**MongoDB para eventos**
Cada ação relevante no sistema (admissão, promoção, desligamento, alteração salarial) gera um documento de evento com schema flexível no MongoDB, indexado por `tenant_id`, `employee_id` e `timestamp`.

**Multitenancy por schema**
Cada empresa (tenant) possui seu próprio schema no PostgreSQL, isolando os dados sem a complexidade de bancos separados.

**API versionada**
Todos os endpoints seguem o prefixo `/api/v1/` com envelope de resposta padronizado:
```json
{
  "data": {},
  "meta": {},
  "errors": []
}
```

---

## Estrutura dos Projetos

```
people-analytics/
├── api/          # Laravel — API REST
└── web/          # React + TypeScript — SPA
```

---

## Como rodar localmente

> Pré-requisitos: Docker, Node.js 20+, PHP 8.2+, Composer

```bash
# Clonar o repositório
git clone https://github.com/seu-usuario/people-analytics.git
cd people-analytics

# Backend
cd api
cp .env.example .env
composer install
php artisan key:generate
php artisan migrate --seed

# Frontend
cd ../web
cp .env.example .env.local
npm install
npm run dev
```

---

## Status

🚧 Em desenvolvimento — MVP em construção.
