# CooperVote - Especificação do Sistema de Votação Cooperativista

## Visão Geral
Sistema completo para gestão de assembleias cooperativistas, composto por uma API REST (Spring Boot) para gerenciamento de sessões de votação e uma interface web (React) para interação dos usuários. Permite cadastro de pautas, abertura de sessões com duração configurável, recebimento de votos (Sim/Não), validação de CPF via client externo, apuração de resultados, além de dashboard com estatísticas, lista de pautas e controle de sessões.

## Arquitetura

### Backend (API)
```
src/main/java/com/coopervote/
├── CoopervoteApplication.java     # Classe principal
├── config/                       # Configurações
├── application/
│   ├── service/                 # Lógica de negócio
│   └── exception/              # Exceções
├── domain/
│   ├── model/                  # Entidades JPA
│   └── repository/             # Repositórios
├── infrastructure/
│   └── cpf/                   # Client de validação CPF
└── presentation/
    └── rest/
        ├── controller/         # REST Controllers
        └── dto/                # Data Transfer Objects
```

### Frontend (React)
```
src/
├── components/           # Componentes reutilizáveis
│   ├── ConfirmModal.tsx
│   ├── Header.tsx
│   ├── Layout.tsx
│   ├── Sidebar.tsx
│   └── Toast.tsx
├── hooks/                # Custom hooks
│   └── useCpfMask.ts
├── pages/                # Páginas
│   ├── Dashboard.tsx
│   ├── PautaCreate.tsx
│   ├── PautaDetail.tsx
│   ├── PautaList.tsx
│   └── Voting.tsx
├── services/             # Chamadas API
│   ├── agendaService.ts
│   ├── sessionService.ts
│   └── voteService.ts
├── stores/               # Estado global (Zustand)
│   └── appStore.ts
├── types/                # Tipos TypeScript
│   └── index.ts
├── App.tsx               # Componente principal
└── main.tsx              # Entry point
```

## Stack Tecnológica
| Camada | Tecnologia | Versão |
|--------|-----------|--------|
| Backend | Java | 21 |
| Backend | Spring Boot | 3.4.5 |
| Backend | Spring Data JPA | - |
| Backend | PostgreSQL | 16+ |
| Backend | Flyway | - |
| Backend | Springdoc OpenAPI | 2.7.0 |
| Backend | Lombok | - |
| Frontend | React | 19 |
| Frontend | TypeScript | 5 |
| Frontend | TailwindCSS | 4 |
| Frontend | Zustand | - |
| Frontend | Framer Motion | - |
| Frontend | Vite | 8 |
| Infra | Docker | - |
| Infra | Nginx | - |

## Especificação de Funcionalidades

### Backend (API)

#### 1. Cadastrar Pauta
Cria uma nova pauta para votação em assembleia.
- **Endpoint:** `POST /api/v1/agendas`
- **Request:**
```json
{
  "title": "Aumento do Capital Social",
  "description": "Proposta de aumento do capital social em 10%"
}
```
- **Response:** `201 Created`
```json
{
  "id": 1,
  "title": "Aumento do Capital Social",
  "description": "Proposta de aumento do capital social em 10%",
  "createdAt": "2026-04-26T10:00:00"
}
```

#### 2. Listar Pautas
Retorna todas as pautas cadastradas.
- **Endpoint:** `GET /api/v1/agendas`

#### 3. Abrir Sessão de Votação
Abre uma nova sessão de votação para uma pauta existente.
- **Endpoint:** `POST /api/v1/sessions/agenda/{agendaId}`
- **Request:**
```json
{
  "durationMinutes": 5
}
```
- **Regras:**
  - Duração padrão: 1 minuto
  - Máximo: 60 minutos
  - Apenas uma sessão por pauta

#### 4. Registrar Voto
Registra o voto de um associado.
- **Endpoint:** `POST /api/v1/votes/session/{sessionId}`
- **Request:**
```json
{
  "cpf": "12345678901",
  "voteValue": true
}
```
- **Validações:**
  - CPF deve ter 11 dígitos
  - CPF é validado via client externo
  - Apenas um voto por CPF por sessão
  - Sessão deve estar ativa

#### 5. Obter Resultado da Votação
Retorna o resultado da votação.
- **Endpoint:** `GET /api/v1/votes/session/{sessionId}/result`
- **Response:**
```json
{
  "sessionId": 1,
  "agendaId": 1,
  "yesVotes": 15,
  "noVotes": 5,
  "totalVotes": 20
}
```

#### 6. Encerrar Sessão
Encerra manualmente uma sessão de votação.
- **Endpoint:** `POST /api/v1/sessions/{id}/close`

### Frontend (Interface Web)

#### 1. Dashboard
- Visualização geral das pautas
- Indicadores de status
- Links rápidos para ações

#### 2. Lista de Pautas
- Visualização de todas as pautas
- Status da sessão (Pendente/Ativo/Encerrado)
- Ações rápidas

#### 3. Criar Pauta
- Formulário para nova pauta (título e descrição)
- Redirect para Dashboard após criação

#### 4. Detalhes da Pauta
- Visualização completa da pauta
- Status da sessão
- Botão para abrir/encerrar sessão
- Resultado da votação

#### 5. Votação
- Campo CPF com máscara (000.000.000-00)
- Botões SIM/NÃO
- Modal de confirmação
- Feedback visual

#### 6. Componentes UI
##### Toast Notifications
- Posição: topo direito
- Tipos: success, error, warning, info
- Animações com Framer Motion
- Fechamento automático (4s)

##### Modal de Confirmação
- Confirmação de voto
- Confirmação de encerramento de sessão

##### Sidebar
- Navegação entre páginas
- Links: Dashboard, Pautas, Nova Pauta

## Especificações da API

### Endpoints
| Método | Endpoint | Descrição |
|--------|----------|---------|
| GET | `/api/v1/agendas` | Listar pautas |
| POST | `/api/v1/agendas` | Criar pauta |
| GET | `/api/v1/agendas/{id}` | Buscar pauta |
| POST | `/api/v1/sessions/agenda/{id}` | Abrir sessão |
| GET | `/api/v1/sessions/{id}` | Buscar sessão |
| POST | `/api/v1/sessions/{id}/close` | Encerrar sessão |
| POST | `/api/v1/votes/session/{id}` | Registrar voto |
| GET | `/api/v1/votes/session/{id}/result` | Ver resultado |

### Códigos de Resposta HTTP
| Código | Significado |
|--------|-------------|
| 201 | Criado com sucesso |
| 200 | Sucesso |
| 400 | Erro de validação |
| 404 | Recurso não encontrado |
| 409 | Conflito (voto duplicado) |
| 422 | Entidade não processável |
| 429 | Muitas requisições |
| 500 | Erro interno |

### Mensagens de Erro (Português)
| Erro | HTTP | Mensagem |
|------|------|----------|
| Pauta não encontrada | 404 | Pauta nao encontrada com ID: {id} |
| Sessão não encontrada | 404 | Sessao de votacao nao encontrada com ID: {id} |
| Sessão já existe | 409 | Ja existe uma sessao de votacao para esta pauta: {id} |
| Sessão encerrada | 422 | Sessao de votacao encerrada: {id} |
| Voto duplicado | 409 | Associado com CPF {cpf} ja votou nesta sessao ({sessionId}) |
| CPF inválido | 422 | CPF com formato invalido |
| CPF não pode votar | 422 | Status do CPF: UNABLE_TO_VOTE |

## Banco de Dados (PostgreSQL)

### Tabelas
| Tabela | Descrição |
|--------|-----------|
| `agenda` | Pautas de votação |
| `voting_session` | Sessões de votação |
| `vote` | Votos registrados |

### Índices e Restrições
- Unique: `uk_agenda_title` (título único)
- Unique: `uk_vote_cpf_session` (um voto por CPF por sessão)
- Index: `idx_vote_session_id` (busca por sessão)
- Index: `idx_voting_session_agenda_id` (busca por pauta)

## Regras de Validação

### CPF
- Formato: 000.000.000-00 (máscara automática no frontend)
- 11 dígitos numéricos
- Validação via client externo no backend

### Voto
- CPF obrigatório
- Uma votação por CPF por sessão
- Sessão deve estar ativa (status Ativo)

### Status de Sessão
| Status | Cor | Descrição |
|--------|-----|-----------|
| Pendente | Amarelo | Sessão não iniciada |
| Ativo | Verde | Votação em andamento |
| Encerrado | Cinza | Votação finalizada |

## Design System (Frontend)
| Cor | Hex | Uso |
|------|-----|-----|
| Primary | #0677F9 | Botões, links |
| Success | #22C55E | Mensagens positivas |
| Danger | #D92626 | Erros, botões negativo |
| Warning | #F59E0B | Avisos |
| Text | #171A1C | Texto principal |
| Text Muted | #91969C | Texto secundário |

## Infraestrutura e Deploy

### Rate Limiting (API)
- Limite: 60 requisições por minuto por IP
- Retorno: HTTP 429 quando excedido

### Flyway Migrations
- Localização: `src/main/resources/db/migration/`
- Executadas automaticamente ao iniciar a aplicação

### Docker
- Multi-stage build para frontend (Vite + Nginx)
- Docker Compose para stack completa (API + PostgreSQL + Frontend)
- Quick start: `docker-compose up -d --build`

### Variáveis de Ambiente
#### Backend
| Variável | Descrição | Padrão |
|----------|----------|-------|
| `SPRING_DATASOURCE_URL` | URL de conexão PostgreSQL | jdbc:postgresql://localhost:5432/coopervote |
| `SPRING_DATASOURCE_USERNAME` | Usuário do banco | coopervote |
| `SPRING_DATASOURCE_PASSWORD` | Senha do banco | coopervote123 |
| `SPRING_PROFILES_ACTIVE` | Perfil Spring | prod |

#### Frontend
| Variável | Descrição | Padrão |
|----------|----------|-------|
| `VITE_API_URL` | URL da API backend | http://localhost:8080 |

## Licença
**© 2026 Lucas Porto. Todos os direitos reservados.**
