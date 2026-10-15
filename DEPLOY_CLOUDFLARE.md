# Guia de Deploy — Elegance Barber Shop

Stack: **React (frontend)** + **FastAPI (backend)** + **MongoDB**.

> Resumo rápido: o **frontend** vai para o **Cloudflare Pages**. O **backend FastAPI**
> precisa de um host que rode Python (Railway / Render / Google Cloud Run) — o
> Cloudflare Workers **não** roda FastAPI nativamente. O **MongoDB** usa o MongoDB Atlas.

---

## 1. Exportar o código (GitHub)

1. Na barra do chat do Emergent, clique em **Save → Save to GitHub**.
2. Conecte sua conta GitHub (OAuth) e crie/escolha um repositório.
3. No seu computador: `git clone <url-do-repo>`.

> `.env` e dados do MongoDB **não** são exportados. Você reconfigura no destino.

---

## 2. Banco de dados — MongoDB Atlas (grátis)

1. Crie uma conta em https://www.mongodb.com/atlas e um cluster gratuito (M0).
2. Em **Database Access**, crie um usuário/senha.
3. Em **Network Access**, libere o IP do seu host de backend (ou `0.0.0.0/0` para testes).
4. Copie a **connection string** (ex.: `mongodb+srv://user:pass@cluster.xxx.mongodb.net`).

---

## 3. Backend FastAPI — Railway / Render / Cloud Run

Variáveis de ambiente necessárias:

```
MONGO_URL=<sua connection string do Atlas>
DB_NAME=elegance
CORS_ORIGINS=https://<seu-dominio-no-cloudflare-pages>
```

Comando de start (porta via variável `$PORT` do provedor):

```
uvicorn server:app --host 0.0.0.0 --port $PORT
```

Diretório raiz do serviço: `backend/`. Dependências: `backend/requirements.txt`.

Após o deploy, anote a URL pública do backend (ex.: `https://elegance-api.up.railway.app`).

---

## 4. Frontend React — Cloudflare Pages

1. Em https://dash.cloudflare.com → **Workers & Pages → Create → Pages → Connect to Git**.
2. Selecione o repositório.
3. Configure o build:
   - **Framework preset:** Create React App
   - **Build command:** `yarn build` (ou `npm run build`)
   - **Build output directory:** `build`
   - **Root directory:** `frontend`
4. Em **Environment variables**, adicione:
   ```
   REACT_APP_BACKEND_URL=https://<url-publica-do-seu-backend>
   ```
5. Faça o deploy. Cloudflare dá uma URL `*.pages.dev` (domínio custom opcional).

> O frontend chama o backend sempre via `REACT_APP_BACKEND_URL` + prefixo `/api`.

---

## 5. Checklist final

- [ ] `CORS_ORIGINS` no backend aponta para o domínio do Cloudflare Pages.
- [ ] `REACT_APP_BACKEND_URL` no Pages aponta para a URL pública do backend.
- [ ] MongoDB Atlas liberou o IP do backend.
- [ ] Testar o envio do formulário de agendamento (POST `/api/booking-requests`).
- [ ] Links de WhatsApp (`wa.me`) funcionam (não precisam de backend).

---

## Alternativa mais simples

O **deploy gerenciado do Emergent** já usa Cloudflare para o frontend e cuida de
backend + MongoDB automaticamente, com URL pública, rollback e variáveis gerenciadas.
Custo: 50 ECUs/mês. Basta usar o botão de Deploy na plataforma.
