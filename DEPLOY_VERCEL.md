# Deploy na Vercel — ELEGANCE (projeto multi-serviço)

Este repositório está configurado para rodar como **um único projeto Vercel com múltiplos serviços**
(ver `vercel.json` na raiz): um backend FastAPI e um frontend React (CRA), com domínio e roteamento
compartilhados.

## Serviços

| Serviço  | Pasta (`root`) | Framework          | Entrypoint     | Público em  |
|----------|----------------|--------------------|----------------|-------------|
| backend  | `backend`      | `fastapi`          | `server:app`   | `/api/(.*)` |
| frontend | `frontend`     | `create-react-app` | —              | `/(.*)`     |

As reescritas (`rewrites`) vão da mais específica para a mais genérica: `/api/*` cai no backend;
todo o resto cai no frontend.

## Sem bindings

Não há comunicação servidor-a-servidor. O único tráfego é **navegador → backend**
(o formulário de contato faz `POST /api/booking-requests`), que passa pelo rewrite público `/api`
no mesmo domínio. Por isso não há `bindings` no `vercel.json`.

## Passo a passo

1. Suba o repositório para o GitHub (botão **Save to Github** no chat).
2. Na Vercel: **Add New → Project** e importe o repositório.
3. Em **Framework**, selecione **Services** (a Vercel também detecta pelo `vercel.json`).
4. Configure as **Environment Variables** (abaixo) e clique em **Deploy**.

## Variáveis de ambiente (painel da Vercel)

### Serviço `backend`
- `MONGO_URL` — string de conexão do MongoDB (recomendado **MongoDB Atlas**, ex.:
  `mongodb+srv://usuario:senha@cluster.xxxx.mongodb.net/?retryWrites=true&w=majority`).
- `DB_NAME` — nome do banco (ex.: `elegance`).
- `CORS_ORIGINS` *(opcional)* — origens permitidas separadas por vírgula. Como o site chama o backend
  no **mesmo domínio** via `/api`, o CORS não é necessário; pode deixar `*` ou definir o domínio final.

### Serviço `frontend`
- `REACT_APP_BACKEND_URL` — **deixe VAZIO** (ou não defina). Assim o site chama `/api/...` no mesmo
  domínio e o rewrite roteia para o backend. (Só preencha com uma URL completa se, por algum motivo,
  quiser apontar para um backend em outro domínio.)

## MongoDB Atlas (resumo)

1. Crie um cluster gratuito (M0) em https://www.mongodb.com/atlas.
2. **Database Access**: crie um usuário/senha.
3. **Network Access**: libere `0.0.0.0/0` (ou os IPs da Vercel).
4. **Connect → Drivers**: copie a connection string e use em `MONGO_URL`.

## Teste local (opcional)

Com a Vercel CLI: `vercel dev` sobe todos os serviços juntos e injeta as variáveis de vinculação.
