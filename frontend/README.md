# Frontend - Projeto Miranda

Aplicação Single Page Application (SPA) para comunicação institucional e relacionamento direto com cidadãos.

## Tecnologias

- **React 19**
- **TypeScript**
- **Vite**
- **React Router DOM**
- **Axios**
- **Lucide React**

## Scripts Disponíveis

Na pasta `frontend/`, você pode executar:

### `npm install`
Instala todas as dependências do projeto.

### `npm run dev`
Inicia o servidor de desenvolvimento local com hot-reloading (geralmente em `http://localhost:5173`).

### `npm run build`
Executa a checagem de tipos do TypeScript (`tsc -b`) e gera a compilação otimizada de produção no diretório `dist/`.

### `npm run lint`
Executa o linter ESLint para validação de estilo e regras do código.

### `npm run preview`
Inicia um servidor local para visualizar o build de produção gerado.

## Variáveis de Ambiente

Crie um arquivo `.env` ou `.env.local` na pasta `frontend/`:

```env
VITE_API_URL=http://localhost:8000
```

Em produção, o frontend está publicado na Cloudflare Pages:
`https://nexa-ads.pages.dev`
