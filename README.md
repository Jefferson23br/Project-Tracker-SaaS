Project Tracker SaaS – README
Visão Geral
O Project Tracker SaaS é uma plataforma de acompanhamento de projetos com controle de acesso por perfis, gestão de tarefas com cronômetro por atividade, painel de indicadores e um “mini-repositório” de arquivos com log de cópias (downloads). O objetivo é oferecer uma visão clara de progresso do projeto, produtividade (horas), status de tarefas e interação dos envolvidos, com uma experiência fluida de quadro Kanban (arrastar e soltar).

Perfis e permissões:

Admin
Cria/edita projetos, define quem pode acessar/ver cada projeto
Designa Desenvolvedores e Contratantes por projeto
Participa como desenvolvedor (criando tarefas, atuando e publicando “Publicação”)
Edita e publica a atividade especial “Publicação” (20% do progresso)
Desenvolvedor
Atua nos projetos designados
Cria tarefas, registra andamento, altera status
Usa cronômetro por tarefa
Comenta/atualiza a thread de descrição da tarefa (formato “chat”)
Contratante
Apenas visualiza projetos e tarefas associadas
Acompanha indicadores, status e logs
Principais funcionalidades:

Múltiplos projetos, cada um com inúmeras tarefas
Quadro Kanban com 3 colunas: “Não iniciada”, “Em andamento”, “Finalizada”, com drag and drop
Cartões de tarefa com: autor/criador, quem está trabalhando (multi-dev), data de abertura, última atualização, horas trabalhadas, data de finalização
Cronômetro por tarefa que continua contando mesmo ao sair da página (até o dev parar)
Painel do projeto com KPIs: horas totais, número de tarefas, número de devs, último acesso do contratante
Cálculo de progresso do projeto baseado em pesos por status e atividade “Publicação”
Repositório de arquivos por projeto (estilo simples de GitHub) com log de cópia (download) por usuário
Controle de acesso por usuário: cada usuário vê apenas o que for designado a ele
Arquitetura
Stack sugerida
Front-end:
React + TypeScript
State management: Redux Toolkit ou Zustand
UI: TailwindCSS + Headless UI (ou MUI)
Drag and drop: @dnd-kit/core ou react-beautiful-dnd
Data fetching: React Query (TanStack Query)
Back-end:
Node.js + TypeScript
Framework: NestJS (ou Express + Zod para validação)
Autenticação/Autorização: JWT + RBAC (Role-Based Access Control)
WebSockets (NestJS Gateway ou Socket.IO) para atualizações em tempo real (cronômetro, Kanban, comentários)
Banco de Dados:
PostgreSQL (via Prisma ORM)
Redis para cache e locks de cronômetro (evitar contagens duplicadas)
Storage de arquivos:
S3 compatível (AWS S3, MinIO, ou Cloud provider) com metadados de download
Observabilidade:
Log: Winston/Pino
Métricas: Prometheus + Grafana (opcional)
Infraestrutura:
Docker/Docker Compose para dev
CI/CD: GitHub Actions
Deploy: Kubernetes ou serviços gerenciados (Railway/Render/Fly.io/Heroku-like para POC)
Modelagem de Domínio
Entidades principais:

User
id, name, email, password_hash
roles globais? Preferível RBAC por projeto (ProjectUserRole)
last_login_at
Project
id, name, description
created_by, created_at, updated_at
special_activity_publication_status (para o item “Publicação”)
ProjectUserRole
id, project_id, user_id, role (ADMIN | DEVELOPER | CLIENT)
visibility flags (ex.: ativo/inativo)
Task
id, project_id, title, description
status (NOT_STARTED | IN_PROGRESS | DONE)
created_by, assigned_to (opcional para “dono” principal)
opened_at, updated_at, finished_at (nulo se não finalizada)
hours_tracked (acumulado em segundos)
TaskContributor
id, task_id, user_id, first_access_at, last_access_at
active_session_count (controle de sessões simultâneas, se necessário)
TimeEntry
id, task_id, user_id, started_at, stopped_at (nulo se em andamento)
duration_seconds (redundância preenchida ao stop)
TaskComment
id, task_id, user_id, message (markdown ou rich text), created_at
parent_id (para threads, opcional)
FileAsset
id, project_id, path/key, name, size_bytes, content_type
uploaded_by, uploaded_at, checksum (opcional para deduplicação)
FileDownloadLog
id, file_id, user_id, downloaded_at, ip, user_agent
Índices:

Índices por project_id em Task, FileAsset, ProjectUserRole
Índices por task_id em TimeEntry, TaskComment, TaskContributor
Índices por user_id em ProjectUserRole e TimeEntry
Índices por status em Task para KPIs
Regras de Negócio
Acesso e visibilidade:
Usuário só enxerga projetos nos quais possui ProjectUserRole.
RBAC por projeto:
ADMIN: CRUD do projeto, gerenciamento de acesso, tarefas, “Publicação”
DEVELOPER: CRUD de tarefas do projeto; iniciar/parar cronômetro; comentar; mover no Kanban
CLIENT: somente leitura
Kanban:
Tarefa é arrastável entre colunas; mudança de status gera log de atualização e atualiza updated_at.
Ao mover para “Finalizada”, define finished_at (se não definido) e trava cronômetros ativos.
Cronômetro:
Desenvolvedor pode iniciar múltiplas tarefas? Por padrão, permitir apenas uma ativa por usuário por projeto (configurável).
Iniciar cronômetro cria TimeEntry com started_at.
Parar cronômetro define stopped_at, calcula duration_seconds, soma em Task.hours_tracked.
Persistência resiliente: se o dev fecha a página, cronômetro segue ativo (servidor mantém estado). Use Redis lock/heartbeat para sessões.
Prevenir sessões zumbis: heartbeat do cliente; se perder sinal por muito tempo, opcional notificar ou auto-stop após X horas (configurável por projeto).
Progresso do projeto:
Peso por status:
Tarefa finalizada: 1.00% cada
Tarefa em andamento: 0.50% cada
Tarefa não iniciada: 0.25% cada
Atividade “Publicação” (campo/flag do projeto): 20% adicionais quando marcada como “concluída” pelo Admin.
Fórmula exemplo:
Seja:
F = tarefas DONE
P = tarefas IN_PROGRESS
N = tarefas NOT_STARTED
Pub = 20 se publicação concluída, senão 0
Progresso base = F1 + P0.5 + N*0.25
Progresso total (%) = min(100, Progresso base + Pub)
Observação: Defina estratégia para evitar ultrapassar 100%. Recomendado normalizar por uma “meta” ou limitar pelo min(100, ...). Outra opção: calcular percentuais relativos com base em total de tarefas e aplicar a publicação como componente separado:
Parte Tarefas (%) = ((F1 + P0.5 + N0.25) / (T1)) * 80
Parte Publicação (%) = 20 se concluída
Total (%) = Parte Tarefas + Parte Publicação
Esta segunda fórmula é mais estável, pois reserva 80% às tarefas e 20% à Publicação.
Comments/Atualizações:
Thread tipo chat por tarefa; registrar quem postou, horário e edição (se permitido).
Ao um dev abrir a tarefa, criar/atualizar TaskContributor e exibir avatares dos devs ativos/atuantes.
Repositório de arquivos (com log de cópias):
Upload com metadados; controle de acesso por projeto.
Cada download gera FileDownloadLog com user_id, timestamp e client info.
Último acesso do contratante:
Atualizar ProjectUserRole.last_access_at quando um CLIENT visualizar a página do projeto.
Fluxos Principais (E2E)
Onboarding
Admin cria projeto e define acesso (Developers e Clients).
Admin pode criar primeira “Publicação” (flag de status no projeto).
Gestão de tarefas
Developer cria tarefa → aparece em “Não iniciada”.
Developer inicia cronômetro → TimeEntry aberto, status pode ir para “Em andamento”.
Comentários/descrições são adicionados em formato de chat.
Ao finalizar, move para “Finalizada” → finished_at setado, cronômetro fechado se ativo.
Progresso e Painel
KPIs atualizados em tempo real via WebSocket.
Gráficos/indicadores: horas totais, tarefas por status, número de devs, último acesso do cliente, progresso total.
Arquivos
Upload de documentos/códigos relacionados.
A cada download, um log persiste o autor da cópia.
Design da API (sugestão REST + WebSocket)
Base: /api/v1

Auth
POST /auth/login
POST /auth/refresh
Users
GET /users/me
Projects
GET /projects (apenas visíveis ao usuário)
POST /projects (Admin)
GET /projects/:id
PATCH /projects/:id (Admin)
GET /projects/:id/kpis
POST /projects/:id/publication (Admin: marcar/atualizar status de “Publicação”)
Access
GET /projects/:id/members
POST /projects/:id/members (Admin: adicionar role)
PATCH /projects/:id/members/:userId (alterar role/estado)
Tasks
GET /projects/:id/tasks?status=...
POST /projects/:id/tasks (Dev/Admin)
GET /tasks/:taskId
PATCH /tasks/:taskId (status, título, etc.)
POST /tasks/:taskId/move (Kanban drag: status + posição)
Time Tracking
POST /tasks/:taskId/timer/start (Dev)
POST /tasks/:taskId/timer/stop (Dev)
GET /tasks/:taskId/time-entries
Comments
GET /tasks/:taskId/comments
POST /tasks/:taskId/comments
Contributors
GET /tasks/:taskId/contributors
Files
GET /projects/:id/files
POST /projects/:id/files (upload via presigned URL)
GET /files/:fileId/download (registra log e redireciona para URL)
GET /files/:fileId/download-logs (Admin)
WebSocket canais:

project:{id}: atualizações de KPIs, tarefas, movimentos no Kanban, publicação
task:{id}: comentários em tempo real, cronômetro, contribuintes ativos
Estrutura de Pastas (sugestão)
frontend/
src/
app/ (rotas, layout)
components/
features/
auth/
projects/
tasks/
files/
dashboard/
hooks/
lib/ (api clients, ws clients)
store/
styles/
package.json
vite.config.ts ou next.config.js (se Next.js)
backend/
src/
main.ts
app.module.ts
auth/
users/
projects/
tasks/
time-tracking/
comments/
files/
websockets/
common/ (guards, interceptors, dto, utils)
prisma/ (schema.prisma)
prisma/
migrations/
schema.prisma
package.json
infra/
docker-compose.yml
k8s/ (manifests)
ci/
github-actions/
backend.yml
frontend.yml
docs/
api.md
architecture.md
Setup de Desenvolvimento
Pré-requisitos:

Node.js LTS
PNPM ou Yarn
Docker + Docker Compose
PostgreSQL e Redis (via docker-compose)
AWS credenciais (ou MinIO) para storage, em dev use MinIO
Passos:

Clone o repositório
Configurar envs:
backend/.env
DATABASE_URL=postgresql://user:pass@localhost:5432/project_tracker
REDIS_URL=redis://localhost:6379
JWT_SECRET=...
STORAGE_ENDPOINT=http://localhost:9000
STORAGE_ACCESS_KEY=...
STORAGE_SECRET_KEY=...
STORAGE_BUCKET=project-tracker
frontend/.env
VITE_API_BASE_URL=http://localhost:3000/api/v1
VITE_WS_URL=ws://localhost:3000
Subir infra:
docker compose up -d (Postgres, Redis, MinIO)
Backend:
cd backend
pnpm install
pnpm prisma migrate dev
pnpm start:dev
Frontend:
cd frontend
pnpm install
pnpm dev
Acesse:
Frontend: http://localhost:5173 (ou porta do Vite/Next)
Backend: http://localhost:3000/api/v1
MinIO: http://localhost:9000 (console http://localhost:9001)
Seeds (opcional):

Criar comando pnpm seed no backend para criar usuário Admin e um projeto de exemplo.
Segurança e Compliance
Senhas com hash (Argon2/bcrypt).
JWT curto + refresh token.
RBAC estrito por projeto.
Rate limiting em endpoints sensíveis.
Auditoria básica: registrar mudanças de status de tarefas e alterações de acesso.
Sanitização de inputs e validação DTO (Zod ou class-validator).
CORS configurado.
Logs de download de arquivos com IP e user agent.
Política para encerramento automático de cronômetro após N horas (configurável).
Backups do banco.
Métricas e KPIs do Painel
Projetos:
Horas totais trabalhadas (soma de TimeEntry)
Total de tarefas por status
Número de desenvolvedores ativos (no período)
Último acesso do contratante
Progresso (%) conforme fórmula
Tarefas:
Tempo total trabalhado
Tempo médio por tarefa
Lead time (abertura → finalização)
Throughput (tarefas finalizadas por período)
UX do Quadro Kanban e Cartões de Tarefa
Colunas:

Não iniciada
Em andamento
Finalizada
Cartão exibe:

Título
Criador
Devs atuando (avatares múltiplos)
Data de abertura
Última atualização
Horas trabalhadas (HH:MM)
Data de finalização (se houver)
Ações:

Arrastar entre colunas
Abrir detalhe: visão completa, comentários (chat), iniciar/parar cronômetro
Comentários (chat):

Listagem reversa por tempo
Menções @usuario (opcional)
Upload de anexos no comentário (opcional)
Estratégia do Cronômetro
Ao iniciar:
Verifica se usuário já tem um TimeEntry ativo (mesmo projeto) se política for “um ativo por projeto”
Cria entrada com started_at
Emite evento WS para task:{id}
Ao parar:
Calcula duração
Atualiza Task.hours_tracked
Emite evento WS
Resiliência:
Heartbeat do cliente a cada 15–30s
Se sem heartbeat por > X minutos, manter ativo mas alertar; opcional auto-stop após Y horas
Conflitos:
Locks Redis por chave timer:user:{id}:task:{id}
Repositório de Arquivos e Log de Cópia
Upload via URL pré-assinada
Metadados salvos no DB
Download via endpoint que:
Valida permissão
Registra FileDownloadLog
Redireciona para a URL do arquivo
Listar logs (Admin do projeto) para auditoria
Testes
Unitários:
Regras de progresso
Cronômetro (start/stop, acúmulo, edge cases)
RBAC por projeto
Integração:
Fluxos Auth → Projetos → Tarefas → Timer → Comentários
Upload/Download com logs
E2E:
Cypress/Playwright no front
Pact (opcional) para contratos entre FE/BE
CI/CD
GitHub Actions:
Lint + Testes (frontend/backend)
Build imagens Docker
Deploy para ambiente de staging (opcional)
Versionamento semântico
Migrations automáticas em deploy (com cuidado)
Roadmap
v0.1 (MVP)
Auth + RBAC por projeto
CRUD de projeto e membros
CRUD de tarefas + Kanban
Cronômetro básico (start/stop)
Comentários em tarefas
KPIs simples
v0.2
WebSockets em tempo real
Repositório de arquivos com logs de download
Fórmula de progresso com “Publicação”
v0.3
Métricas avançadas e relatórios exportáveis (CSV)
Menções e notificações
Auto-stop configurável do cronômetro
v1.0
Hardening de segurança, auditoria e trilhas
Multi-idioma
Integrações (issue trackers, Git providers – opcional)