# Projeto Miranda - Sistema de Comunicação Institucional

Plataforma integrada de comunicação institucional e relacionamento direto com cidadãos, voltada para prefeituras e órgãos da administração pública.

---

## Visão Geral

O **Projeto Miranda** é uma solução desenvolvida para centralizar e profissionalizar a emissão de comunicados oficiais por entidades governamentais. A plataforma permite a criação, segmentação e publicação de informes com anexos multimídia, disparo automatizado de notificações push para dispositivos móveis de munícipes, controle rigoroso de acessos administrativos, trilha imutável de auditoria e conformidade técnica com a Lei Geral de Proteção de Dados (LGPD).

---

## Arquitetura

O sistema opera em uma arquitetura desacoplada e distribuída, otimizada para alto desempenho, custo eficiente e isolamento de responsabilidades:

```text
       Usuário (Cidadão / Gestor Público)
                       │
                       ▼
       ┌───────────────────────────────┐
       │       Cloudflare Pages        │
       │        (React + Vite)         │
       └───────────────┬───────────────┘
                       │ HTTPS / REST
                       ▼
       ┌───────────────────────────────┐
       │            Vercel             │
       │    (Django REST Framework)    │
       └───────┬───────────────┬───────┘
               │               │
      Conexão  │ SSL           │ Armazenamento / CDN
               ▼               ▼
       ┌───────────────┐┌───────────────┐
       │  Aiven MySQL  ││  Cloudinary   │
       │ (Banco Dados) ││(Mídia/Vídeos) │
       └───────────────┘└───────────────┘
               ▲
               │ Notificações Push
               ▼
       ┌───────────────┐
       │   Firebase    │
       │ (Cloud Msg)   │
       └───────────────┘
```

### Serviços de Apoio e Contingência

- **Cloudinary:** Armazenamento externo de arquivos de mídia, logotipos, brasões e vídeos otimizados.
- **Firebase Cloud Messaging (FCM):** Plataforma para entrega de notificações push em dispositivos cadastrados.
- **Render (Workers e Rollback):** Ambiente de apoio que hospeda a instância de Redis e os serviços Celery (`celery-worker` e `celery-beat`) para tarefas assíncronas em lote e agendamentos periódicos, servindo adicionalmente como infraestrutura de contingência e *rollback* para o backend web.

---

## Stack Tecnológica

### Frontend
- **Linguagem & Framework:** React 19, TypeScript, Vite
- **Roteamento:** React Router DOM
- **Cliente HTTP:** Axios
- **Ícones:** Lucide React
- **Estilização:** CSS modular e layouts responsivos

### Backend
- **Framework Principal:** Django 4.2+ (LTS)
- **API REST:** Django REST Framework (DRF)
- **Documentação de API:** DRF Spectacular (OpenAPI 3.0 / Swagger UI)
- **Interface Administrativa:** Django Admin com tema Django Jazzmin
- **Processamento de Imagens e Vídeo:** Pillow, imageio-ffmpeg
- **Arquivos Estáticos:** WhiteNoise
- **Servidor WSGI/ASGI:** Gunicorn (para execução em container/servidor dedicado)

### Banco de Dados
- **SGBD:** MySQL 8
- **Hospedagem Gerenciada:** Aiven MySQL
- **Driver e Criptografia:** PyMySQL, Cryptography (negociação SSL/TLS obrigatória)

### Infraestrutura & Deploy
- **Frontend SPA:** Cloudflare Pages
- **API Serverless:** Vercel
- **Filas e Agendador de Tarefas:** Celery e Redis (Render)
- **Ambiente de Contingência/Rollback:** Render

### Storage & Notificações
- **Armazenamento de Mídia:** Cloudinary API
- **Push Notifications:** Firebase Admin SDK (FCM)

### Ferramentas de Qualidade
- **Linters:** ESLint (Frontend), Flake8 / Django Check (Backend)
- **Compilação e Checagem:** TypeScript Compiler (`tsc`), Python `compileall`

---

## Estrutura do Projeto

```text
projeto_miranda/
├── frontend/                     # Aplicação Single Page Application (SPA)
│   ├── src/
│   │   ├── common/               # Clientes de API, serviços e utilitários compartilhados
│   │   ├── feature/              # Módulos funcionais da aplicação
│   │   │   ├── announcementsEdition/ # Edição e publicação de comunicados
│   │   │   ├── auth/                 # Login, formulários e sessão
│   │   │   ├── communication/        # Feed de comunicados institucionais
│   │   │   ├── controlAcess/         # Gestão de permissões de usuários
│   │   │   ├── conversation/         # Sistema de mensagens e chat interno
│   │   │   ├── dashboard/            # Painel com gráficos e métricas
│   │   │   ├── identification/       # Gestão da identidade visual institucional
│   │   │   ├── register/             # Cadastro de novos cidadãos
│   │   │   └── splash/               # Tela de apresentação inicial
│   │   ├── pages/                # Componentes de página e visualizações principais
│   │   └── routes/               # Definição e proteção de rotas (públicas e privadas)
│   └── package.json              # Dependências e scripts do frontend
│
├── backend/                      # API REST e motor de regras de negócio
│   ├── core/                     # Módulo central do Django
│   │   ├── settings.py           # Configurações gerais, segurança, logging e integrações
│   │   ├── urls.py               # Roteamento central e endpoints de documentação
│   │   ├── celery.py             # Configuração da instância Celery
│   │   ├── wsgi.py               # Ponto de entrada WSGI para servidores web
│   │   └── asgi.py               # Ponto de entrada ASGI
│   ├── api/                      # Aplicação Django com lógica de negócio
│   │   ├── models.py             # Modelos relacionais do sistema
│   │   ├── views.py              # ViewSets e endpoints da API REST
│   │   ├── serializers.py        # Serializadores DRF
│   │   ├── authentication.py     # Estratégia de autenticação e expiração de tokens
│   │   ├── tasks.py              # Tarefas assíncronas do Celery
│   │   ├── delivery.py           # Mecanismo de entrega e rastreio de notificações
│   │   ├── media_processing.py   # Compressão de vídeo e otimização de anexos
│   │   ├── audit.py              # Gravação de eventos na trilha de auditoria
│   │   ├── privacy.py            # Tratamento de solicitações de privacidade e LGPD
│   │   ├── tests.py              # Suíte completa de testes automatizados
│   │   └── management/           # Comandos personalizados do Django (CLI)
│   ├── docs/                     # Documentações técnicas e operacionais
│   │   ├── AIVEN_SSL_TROUBLESHOOTING.md # Resolução de conexões SSL com a Aiven
│   │   ├── BACKUP_RESTORE.md            # Procedimentos de backup e restore
│   │   ├── LGPD_POLICY.md               # Política técnica e operacional de privacidade
│   │   └── PRODUCTION_DEPLOYMENT.md     # Guia detalhado de deploy e infraestrutura
│   ├── scripts/                  # Scripts utilitários de diagnóstico e setup
│   └── requirements.txt          # Dependências Python do backend
│
├── render.yaml                   # Definição de infraestrutura como código (Render)
├── setup-backend.bat             # Script utilitário para inicialização em ambiente Windows
└── .gitignore                    # Regras de exclusão do repositório Git
```

---

## Funcionalidades

- **Autenticação Segura e Controle de Perfis:**
  - Login e registro com validação de credenciais e emissão de tokens de acesso.
  - Distinção clara entre perfis de Cidadão e Gestor Administrativo (`Profile`).
  - Política de rotação de tokens para gestores e controle de tempo de expiração (`TTL`).
- **Gestão de Comunicados Oficiais:**
  - Ciclo de vida completo: rascunho, publicação agendada, publicação ativa e arquivamento.
  - Associação de anexos multimídia (documentos, fotos e vídeos).
- **Segmentação Geográfica e Temática:**
  - Criação de segmentos (`Segment`) vinculados a bairros ou áreas de interesse.
  - Disparo de informes para públicos específicos ou envio geral para toda a base.
- **Notificações Push com Rastreabilidade:**
  - Registro de dispositivos móveis de munícipes (`PushDevice`).
  - Criação de registros individuais de entrega (`DeliveryLog`) para monitoramento de sucesso e falhas.
  - Disparo síncrono ou assíncrono integrado ao Firebase Cloud Messaging.
- **Compressão e Validação de Mídia:**
  - Validação estrita de formatos (imagens e vídeos MP4, MOV, WebM).
  - Compressão automática de vídeos via FFmpeg com codec H.264/AAC, limitação dimensional e otimização de streaming (`faststart`).
  - Armazenamento em nuvem via Cloudinary.
- **Identidade Visual Institucional:**
  - Customização de dados da instituição (prefeitura, secretaria ou autarquia), cores primária e secundária, brasão e logotipo oficial.
- **Canal de Comunicação e Mensagens:**
  - Módulo de chat interno com upload de anexos, listagem de contatos e marcação de mensagens lidas.
- **Trilha de Auditoria e Painel de Métricas:**
  - Registro de ações críticas executadas por gestores (publicações, remoções, alterações de permissão).
  - Dashboard analítico com métricas de comunicados, dispositivos ativos e taxas de entrega.
- **Conformidade LGPD:**
  - Abertura e tramitação de solicitações de privacidade pelo próprio cidadão (anonimização, exportação e exclusão de dados).
  - Opção de desativação de conta pelo usuário.

---

## Segurança

O sistema adota medidas técnicas de proteção em múltiplas camadas, confirmadas diretamente na implementação do código:

- **HTTPS Obrigatório:** Redirecionamento compulsório para tráfego criptografado em produção (`SECURE_SSL_REDIRECT = True`).
- **Proteção HSTS:** Configuração de *HTTP Strict Transport Security* com vigência de um ano (`SECURE_HSTS_SECONDS = 31536000`), incluindo subdomínios e política de *preload*.
- **Cookies Protegidos:** Cookies de sessão e CSRF configurados estritamente com as flags `Secure` e `HttpOnly`.
- **Controle Rígido de CORS e CSRF:**
  - Domínios autorizados limitados explicitamente à origem oficial do frontend (`https://nexa-ads.pages.dev`).
  - Bloqueio de inicialização caso `CORS_ALLOW_ALL_ORIGINS` seja definido como verdadeiro em ambiente de produção (`DEBUG = False`).
- **Rate Limiting (Throttling):**
  - Limites de requisições por IP e usuário autenticado via Django REST Framework.
  - Políticas dedicadas para rotas sensíveis: limitação para tentativas de login, cadastro de novas contas e solicitações de redefinição de senha.
- **Criptografia e SSL no Banco de Dados:**
  - Conexão com o banco MySQL exigindo modo SSL obrigatório (`ssl-mode=REQUIRED`).
  - Suporte a verificação de certificado da autoridade certificadora (`DB_CA_CERT`).
- **Validação de Senhas:**
  - Validação de complexidade, tamanho mínimo e verificação contra sequências comuns via validadores nativos do Django.
- **Compatibilidade Serverless Segura:**
  - Sistema de logs redirecionado para `stdout`/`stderr` em ambiente Vercel, impedindo tentativas de escrita no sistema de arquivos somente leitura da nuvem.

---

## Configuração do Ambiente Local

### Pré-requisitos
- Node.js (versão 18 ou superior) e npm
- Python (versão 3.12 recomendada)
- Instância do MySQL 8 em execução (ou container local)

---

### 1. Configurando o Backend

1. Acesse o diretório do backend:
   ```bash
   cd backend
   ```

2. Crie e ative um ambiente virtual:
   ```bash
   # Linux / macOS
   python3 -m venv venv
   source venv/bin/activate

   # Windows
   python -m venv venv
   venv\Scripts\activate
   ```

3. Instale as dependências:
   ```bash
   pip install -r requirements.txt
   ```

4. Crie o arquivo de variáveis de ambiente `.env` dentro da pasta `backend/` preenchendo as variáveis necessárias (consulte a seção [Variáveis de Ambiente](#variáveis-de-ambiente)).

5. Execute as verificações e inicie o servidor:
   ```bash
   python manage.py check
   python manage.py runserver
   ```

> **Nota para Windows:** Caso prefira, execute o script utilitário `setup-backend.bat` a partir da raiz para preparar as dependências e iniciar o servidor.

---

### 2. Configurando o Frontend

1. Acesse o diretório do frontend:
   ```bash
   cd frontend
   ```

2. Instale as dependências:
   ```bash
   npm install
   ```

3. Inicie o servidor de desenvolvimento:
   ```bash
   npm run dev
   ```

4. Acesse a aplicação no navegador pelo endereço informado no terminal (geralmente `http://localhost:5173`).

---

## Variáveis de Ambiente

As credenciais do sistema são gerenciadas exclusivamente via variáveis de ambiente, nunca commitadas no controle de versão. Abaixo estão os nomes das variáveis organizados por módulo:

### Configurações do Django
- `SECRET_KEY`
- `DEBUG`
- `ALLOWED_HOSTS`
- `TIME_ZONE`
- `LANGUAGE_CODE`

### Banco de Dados (MySQL / Aiven)
- `DB_HOST`
- `DB_PORT`
- `DB_NAME`
- `DB_USER`
- `DB_PASSWORD`
- `DB_CA_CERT`
- `DB_SSL_REQUIRED`
- `DATABASE_URL` (formato consolidado alternativo)

### Armazenamento de Mídia (Cloudinary)
- `CLOUDINARY_CLOUD_NAME`
- `CLOUDINARY_API_KEY`
- `CLOUDINARY_API_SECRET`

### Notificações Push (Firebase)
- `FIREBASE_ENABLED`
- `FIREBASE_PROJECT_ID`
- `FIREBASE_CLIENT_EMAIL`
- `FIREBASE_PRIVATE_KEY`
- `PUSH_DISPATCH_ON_PUBLISH`
- `PUSH_DISPATCH_ASYNC`

### Segurança e Acesso
- `CORS_ALLOWED_ORIGINS`
- `CSRF_TRUSTED_ORIGINS`
- `FRONTEND_URL`
- `MANAGER_TOKEN_TTL_SECONDS`
- `MANAGER_TOKEN_ROTATE_ON_LOGIN`
- `THROTTLE_ANON_RATE`
- `THROTTLE_USER_RATE`
- `AUTH_LOGIN_THROTTLE_RATE`

### Filas, Cache e Tarefas (Render / Celery)
- `REDIS_URL`
- `CELERY_BROKER_URL`
- `CELERY_RESULT_BACKEND`
- `CELERY_TASK_ALWAYS_EAGER`

### Frontend (Vite)
- `VITE_API_URL`

---

## Banco de Dados

O banco de dados relacional oficial do projeto é o **MySQL 8**. 

- **Em Produção:** Hospedado no serviço de nuvem gerenciada **Aiven MySQL**, com criptografia de tráfego TLS/SSL ativa e obrigatória em todas as conexões.
- **Em Desenvolvimento:** Pode ser utilizado qualquer servidor MySQL local compatível, com a flag de exigência SSL ajustada conforme o ambiente.
- **Migrações:** O gerenciamento do esquema relacional é realizado estritamente pelo sistema de migrações do Django (`backend/api/migrations/`).

---

## Deploy

A infraestrutura de produção é dividida nos seguintes provedores:

| Componente | Plataforma | Tipo de Execução | URL de Produção |
| :--- | :--- | :--- | :--- |
| **Frontend** | Cloudflare Pages | Hospedagem Estática / SPA | `https://nexa-ads.pages.dev` |
| **Backend API** | Vercel | Serverless Functions (Python) | `https://projeto-miranda.vercel.app` |
| **Banco de Dados** | Aiven | MySQL Gerenciado (SSL Obrigatório) | *(Host Privado)* |
| **Storage / Mídia** | Cloudinary | Armazenamento de Objetos / CDN | *(CDN Global)* |
| **Workers / Contingência** | Render | Background Workers & Redis | *(Ambiente de Apoio / Rollback)* |

### Fluxo de Deploy
1. **Frontend:** Disparado automaticamente na Cloudflare Pages a cada push na branch de produção, executando `npm run build`.
2. **Backend:** Integrado à Vercel, com detecção automática do framework Django. As rotas são processadas sob demanda com logs estruturados via console.
3. **Tarefas de Segundo Plano (Workers):** Mantidas no Render a partir do manifesto `render.yaml`, executando o worker do Celery e o agendador de tarefas periódicas (*Beat*).

---

## Testes e Validação de Qualidade

Para verificar a integridade do código e a ausência de problemas estruturais, utilize os comandos abaixo:

### Backend
```bash
# Checagem de integridade estrutural do Django
python manage.py check

# Verificação de compilação de código Python
python -m compileall .

# Execução da suíte de testes automatizados
python manage.py test api
```

### Frontend
```bash
# Checagem de tipagem com TypeScript e build do Vite
npm run build

# Validação com linter
npm run lint
```

---

## API e Documentação Interativa

A API REST é estruturada de acordo com as especificações do Django REST Framework. O projeto conta com documentação interativa gerada automaticamente com **OpenAPI 3.0** via **DRF Spectacular**:

- **Interface Swagger UI:** `https://projeto-miranda.vercel.app/api/docs/`
- **Esquema OpenAPI (JSON):** `https://projeto-miranda.vercel.app/api/schema/`
- **Health Check da Aplicação:** `https://projeto-miranda.vercel.app/health/`
- **Health Check Detalhado:** `https://projeto-miranda.vercel.app/health/detailed/`

---

## Status do Projeto

O **Projeto Miranda** está em fase ativa de evolução e implantação institucional. A arquitetura de produção encontra-se estabilizada com a API principal em funcionamento na Vercel, o frontend distribuído na Cloudflare Pages e a persistência assegurada no Aiven MySQL.
