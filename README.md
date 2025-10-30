<div align="center">
  <h3>Stack Principal do Projeto</h3>
  <p>
    <img src="https://img.shields.io/badge/React-61DAFB?style=for-the-badge&logo=react&logoColor=black" alt="React" />
    <img src="https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white" alt="TypeScript" />
    <img src="https://img.shields.io/badge/Redux-764ABC?style=for-the-badge&logo=redux&logoColor=white" alt="Redux" />
    <img src="https://img.shields.io/badge/Tailwind_CSS-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white" alt="Tailwind CSS" />
    <img src="https://img.shields.io/badge/TanStack_Query-FF4154?style=for-the-badge&logo=react-query&logoColor=white" alt="TanStack Query" />
    <img src="https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=node.js&logoColor=white" alt="Node.js" />
    <img src="https://img.shields.io/badge/NestJS-E0234E?style=for-the-badge&logo=nestjs&logoColor=white" alt="NestJS" />
    <img src="https://img.shields.io/badge/JWT-000000?style=for-the-badge&logo=jsonwebtokens&logoColor=white" alt="JWT" />
    <img src="https://img.shields.io/badge/Socket.io-010101?style=for-the-badge&logo=socket.io&logoColor=white" alt="Socket.io" />
    <img src="https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white" alt="PostgreSQL" />
    <img src="https://img.shields.io/badge/Prisma-2D3748?style=for-the-badge&logo=prisma&logoColor=white" alt="Prisma" />
    <img src="https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white" alt="Redis" />
    <img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker" />
    <img src="https://img.shields.io/badge/Amazon_S3-569A31?style=for-the-badge&logo=amazon-s3&logoColor=white" alt="AWS S3" />
    <img src="https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white" alt="GitHub Actions" />
  </p>
</div>
Apresentação do Projeto: Project Tracker SaaS (Para um Recrutador)
Olá! É um prazer apresentar o Project Tracker SaaS, um projeto que desenhei do zero para solucionar um problema comum: a falta de visibilidade e controle granular no acompanhamento de projetos de software.

Minha intenção não foi apenas criar "mais um" gerenciador de tarefas. Meu objetivo foi construir uma plataforma robusta, em tempo real e com foco em performance, que atende a diferentes perfis de usuários (Admins, Devs e Clientes) com uma experiência fluida e intuitiva.

A Visão: O Que Eu Quis Resolver
Eu queria uma ferramenta que oferecesse:

• Visão Clara do Progresso: KPIs e indicadores visuais.

• Produtividade Real: Um cronômetro por tarefa que fosse resiliente, continuando a contagem mesmo se o usuário fechasse a aba.

• Controle de Acesso: Uma hierarquia clara onde cada um vê apenas o que precisa.

• Colaboração: Um quadro Kanban interativo (drag-and-drop) e um "mini-repositório" de arquivos com log de downloads para auditoria.

O Cérebro da Operação: Perfis e Permissões
Para garantir a segurança e a usabilidade, estruturei um controle de acesso por projeto (RBAC) com três perfis distintos:

1. Admin (O Gestor):

•Tem o controle total. Ele cria os projetos e, o mais importante, define quem pode acessar cada um.

•Designa Desenvolvedores e Contratantes, garantindo que o cliente só veja o que lhe compete.

•Também atua no projeto e é o único que pode acionar a "Publicação", uma atividade especial que representa 20% do progresso total.

2. Desenvolvedor (A Força Tarefa):

• É quem faz acontecer. Atua nos projetos em que foi designado.

• Cria, atualiza e move tarefas no Kanban.

• Usa o cronômetro para registrar horas e interage na "thread" de comentários da tarefa, que estruturei para funcionar como um chat.

3. Contratante (O Cliente):

• Tem uma visão "read-only" do projeto.

• Acompanha o progresso, os indicadores e pode baixar os arquivos do repositório, deixando um log de sua atividade.

Funcionalidades-Chave: Onde a Mágica Acontece
Eu foquei em algumas funcionalidades que considero cruciais:

• Kanban Interativo: Um quadro com colunas "Não iniciada", "Em andamento" e "Finalizada". Usei dnd-kit (ou react-beautiful-dnd) para uma experiência de "arrastar e soltar" suave.

• Cronômetro Resiliente: O dev pode iniciar o cronômetro em uma tarefa e ele continua contando no servidor. Se ele fechar o navegador, a contagem não para. Isso é controlado via WebSockets e Redis, garantindo que o tempo não seja perdido.

• Painel de KPIs em Tempo Real: O Admin e o Cliente têm acesso a um dashboard com horas totais, status das tarefas, número de devs e o último acesso do contratante.

• Cálculo de Progresso Ponderado: O progresso não é só sobre tarefas. Eu criei uma fórmula onde 80% vêm das tarefas (com pesos diferentes para "Finalizada", "Em andamento" e "Não iniciada") e 20% vêm da atividade "Publicação" marcada pelo Admin.

• Repositório com Auditoria: Um sistema simples de arquivos por projeto. O diferencial é o log: cada download é registrado, mostrando quem baixou e quando.

Minha Escolha de Arquitetura (A Stack)
Para construir um SaaS robusto, minhas escolhas de tecnologia foram intencionais e focadas em escalabilidade, performance e Developer Experience.

Front-end: A Experiência do Usuário
• React + TypeScript: Minha escolha principal pela componentização e segurança de tipagem.

• Zustand (ou Redux Toolkit): Para um gerenciamento de estado global limpo e eficiente.

• TailwindCSS: Pela agilidade na estilização sem sair do TSX.

• React Query (TanStack): Para gerenciar o cache de dados do servidor, revalidação e mutações, mantendo a UI sempre sincronizada com o back-end.

Back-end: O Motor da Plataforma
• Node.js + NestJS + TypeScript: Escolhi o NestJS por sua arquitetura modular (inspirada no Angular), injeção de dependência e suporte nativo a WebSockets, o que foi crucial para as funcionalidades em tempo real.

• Autenticação: JWT com Refresh Tokens para segurança, implementando um RBAC (Role-Based Access Control) por projeto.

• WebSockets (Socket.IO): Para todas as atualizações em tempo real: movimento no Kanban, novos comentários e, claro, o cronômetro.

Banco de Dados e Infraestrutura
• PostgreSQL + Prisma: Usei PostgreSQL pela sua robustez e o Prisma ORM pela incrível produtividade e segurança de tipos que ele oferece (Typesafe queries).

• Redis: Essencial. Eu o utilizo não só para cache, mas como "lock" para o cronômetro, evitando contagens duplicadas ou sessões "zumbis".

• Storage (S3): Para o repositório de arquivos, optei por uma API compatível com S3 (como o MinIO para dev local ou AWS S3 para produção).

• Docker/Docker Compose: Todo o ambiente de desenvolvimento é containerizado, garantindo consistência e facilitando o setup.

• CI/CD (GitHub Actions): Configurei pipelines para rodar lint, testes e buildar as imagens Docker a cada push.

Regras de Negócio e Desafios que Solucionei
A parte mais complexa foi desenhar as regras de negócio:

• O Cronômetro: Como garantir que ele seja resiliente? Minha solução:

    1. Ao "iniciar", o cliente envia um evento WebSocket.

    2. O servidor (NestJS) cria uma entrada TimeEntry no banco (PostgreSQL) e armazena um "lock" no Redis (ex: timer:user:X:task:Y).

    3. O cliente envia "heartbeats" a cada 30 segundos para manter a sessão viva.

    4. Se o servidor parar de receber heartbeats, ele pode, opcionalmente, parar o cronômetro após um tempo (para evitar sessões zumbis).

    5. Ao "parar", o servidor calcula a duração, atualiza o total Task.hours_tracked e remove o lock do Redis.

• Progresso (Fórmula): A fórmula que criei (Baseada em 80% para tarefas e 20% para "Publicação") garante que o progresso seja justo e não ultrapasse 100%.

• Parte Tarefas (%) = ((F*1 + P*0.5 + N*0.25) / (Total de Tarefas * 1)) * 80

• Parte Publicação (%) = 20 (se concluída)

• Total (%) = Parte Tarefas + Parte Publicação

Planejamento de API e Estrutura
Para manter o projeto organizado, desenhei uma API RESTful (detalhada no api.md) e estruturei o projeto em monorepo (ou pastas separadas) frontend/ e backend/, cada um com sua lógica de testes, build e deploy.

Meu Foco em Qualidade: Segurança e Testes
A segurança foi uma prioridade desde o início:

• Hashing de senhas (Argon2/bcrypt).

• RBAC estrito por projeto (um usuário não pode sequer saber da existência de um projeto ao qual não pertence).

• Logs de auditoria em pontos críticos (como download de arquivos).

• Validação de todos os DTOs de entrada (com class-validator do NestJS ou Zod).

Eu também estruturei um plano de testes:

• Unitários: Para as regras de negócio (cálculo de progresso, lógica do cronômetro).

• Integração: Para os fluxos da API (Auth -> Projeto -> Tarefa).

• E2E (Cypress/Playwright): Para validar o fluxo do usuário no front-end.

Visão de Futuro (Roadmap)
Este projeto está estruturado para crescer. Os próximos passos que planejei incluem:

• v0.2: Implementar os WebSockets, o repositório de arquivos e a fórmula de progresso.

• v0.3: Adicionar métricas avançadas (gráficos de lead time), menções (@usuario) nos chats e notificações.

• v1.0: Focar em hardening de segurança e internacionalização (multi-idioma).