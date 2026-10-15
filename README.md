# ELEGANCE Barber Shop — Site Oficial

Site institucional premium da **ELEGANCE Barber Shop** (Foz do Iguaçu/PR).
Vitrine visual + funil de conversão para WhatsApp + formulário de pré-agendamento.

Stack: **React (CRACO)** + **FastAPI** + **MongoDB**. Projeto 100% independente da Emergent.

---

## 1. Requisitos

**Opção Docker (recomendada):**
- Docker + Docker Compose

**Opção local (sem Docker):**
- Node.js 20+ e Yarn
- Python 3.11+
- MongoDB 7+ (local ou MongoDB Atlas)

---

## 2. Instalação

```bash
git clone <url-do-seu-repositorio>
cd <pasta-do-projeto>
```

### Configurar variáveis de ambiente

```bash
cp backend/.env.example  backend/.env
cp frontend/.env.example frontend/.env
```

Edite os valores conforme a seção **5. Variáveis de ambiente**.

---

## 3. Como executar

### A) Com Docker Compose (tudo de uma vez)

```bash
docker compose up --build
```

- Frontend: http://localhost:3000
- Backend:  http://localhost:8001/api/
- MongoDB:  interno (rede do compose)

Para produção, defina as variáveis antes do build:

```bash
REACT_APP_BACKEND_URL=https://api.seu-dominio.com \
CORS_ORIGINS=https://seu-dominio.com \
docker compose up --build -d
```

### B) Local (sem Docker)

**Backend:**
```bash
cd backend
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
uvicorn server:app --host 0.0.0.0 --port 8001 --reload
```

**Frontend:**
```bash
cd frontend
yarn install
yarn start           # http://localhost:3000
```

---

## 4. Banco de dados

- Banco: **MongoDB**, nome definido por `DB_NAME` (padrão: `elegance`).
- Coleção usada: **`booking_requests`** (envios do formulário de agendamento).
- **Não há ORM nem migrations** — o MongoDB cria a coleção automaticamente no primeiro insert.

Documento `booking_requests`:
```json
{
  "_id": "ObjectId",
  "name": "string",
  "whatsapp": "string",
  "service": "string",
  "date": "string",
  "time": "string",
  "message": "string",
  "created_at": "ISO-8601 string (UTC)"
}
```

> O site **não possui** autenticação, usuários, sessões, uploads ou pagamentos.
> As 117 imagens são arquivos **estáticos** em `frontend/public/images/elegance/`.

---

## 5. Variáveis de ambiente

### `backend/.env`
| Variável | Descrição | Exemplo |
|---|---|---|
| `MONGO_URL` | Conexão do MongoDB | `mongodb://localhost:27017` ou `mongodb+srv://...` |
| `DB_NAME` | Nome do banco | `elegance` |
| `CORS_ORIGINS` | Origens permitidas (vírgula, sem barra final) | `https://seu-dominio.com` |

### `frontend/.env`
| Variável | Descrição | Exemplo |
|---|---|---|
| `REACT_APP_BACKEND_URL` | URL pública do backend (sem `/api`) | `https://api.seu-dominio.com` |
| `WDS_SOCKET_PORT` | Porta do websocket do dev server (opcional) | `3000` |
| `ENABLE_HEALTH_CHECK` | Health check do preview (manter `false`) | `false` |

> **NUNCA** faça commit de `.env` com valores reais. Use apenas `.env.example`.
> Não há secrets/API keys reais neste projeto.

---

## 6. Build de produção

**Frontend:**
```bash
cd frontend
REACT_APP_BACKEND_URL=https://api.seu-dominio.com yarn build
# saída em: frontend/build/  (servir com Nginx/Cloudflare Pages/qualquer CDN)
```

**Backend:** rode com Uvicorn/Gunicorn atrás de um proxy reverso:
```bash
uvicorn server:app --host 0.0.0.0 --port 8001
```

---

## 7. Deploy

Veja **`DEPLOY_CLOUDFLARE.md`** para o passo a passo (Cloudflare Pages + backend em
Railway/Render/Cloud Run + MongoDB Atlas). Resumo das opções:

- **Frontend:** Cloudflare Pages, Vercel, Netlify ou Nginx (pasta `build/`).
- **Backend:** qualquer host que rode Python (Docker, Railway, Render, Cloud Run, VPS).
- **Banco:** MongoDB Atlas (grátis) ou MongoDB self-hosted.

Lembre-se de alinhar `CORS_ORIGINS` (backend) com o domínio do frontend, e
`REACT_APP_BACKEND_URL` (frontend) com a URL pública do backend.

---

## 8. Backup e restauração do banco

```bash
# Backup → ./backups/<timestamp>/elegance
MONGO_URL="mongodb://localhost:27017" DB_NAME="elegance" ./scripts/backup-mongo.sh

# Restaurar
MONGO_URL="mongodb://localhost:27017" DB_NAME="elegance" \
  ./scripts/restore-mongo.sh backups/<timestamp>/elegance
```

> **Dados de produção na Emergent:** o backup acima funciona contra qualquer `MONGO_URL`.
> Se você tem registros gravados no banco de produção da Emergent, exporte-os de lá
> (via `mongodump` apontando para a connection string de produção) e restaure no novo banco.

---

## 9. Como atualizar o sistema

- **Conteúdo do site** (textos, preços, serviços, equipe, links, imagens):
  edite **`frontend/src/config/site.js`** — é o arquivo central de conteúdo.
- **Imagens:** adicione em `frontend/public/images/elegance/` e referencie no `site.js`.
- Após alterar código, rode novamente o build do frontend / reinicie o backend
  (ou `docker compose up --build`).

---

## 10. Serviços externos necessários

| Serviço | Obrigatório? | Observação |
|---|---|---|
| MongoDB | Sim | Self-hosted ou Atlas (grátis) |
| WhatsApp (`wa.me`) | Não requer conta/API | Apenas deep links no frontend |
| Google Fonts | Opcional | CDN de fontes (pode ser self-hosted) |
| Google Maps | Opcional | Embed estático (sem API key) |

Nenhum serviço pago é obrigatório.

---

## 11. Estrutura do projeto

```
.
├── backend/
│   ├── server.py              # FastAPI: GET /api/ e POST /api/booking-requests
│   ├── requirements.txt
│   ├── Dockerfile
│   ├── .env.example
│   └── tests/                 # pytest
├── frontend/
│   ├── public/
│   │   ├── index.html         # SEO/OG + JSON-LD (telemetria da Emergent removida)
│   │   └── images/elegance/   # 117 imagens estáticas (AVIF/WebP)
│   ├── src/
│   │   ├── components/elegance/   # Seções do site (Hero, Gallery, Contact, ...)
│   │   ├── components/ui/         # shadcn/ui
│   │   ├── config/site.js         # CONTEÚDO CENTRAL do site
│   │   ├── App.js
│   │   └── index.js
│   ├── Dockerfile
│   ├── nginx.conf
│   └── .env.example
├── scripts/                   # backup/restore do MongoDB
├── docker-compose.yml
├── DEPLOY_CLOUDFLARE.md
├── EMERGENT_DEPENDENCIES.md   # auditoria de dependências da Emergent (todas removíveis)
└── README.md
```

---

Feito com atenção à preservação total das funcionalidades originais.
Consulte `EMERGENT_DEPENDENCIES.md` para o diagnóstico de independência da Emergent.
