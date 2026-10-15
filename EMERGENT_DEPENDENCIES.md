# EMERGENT_DEPENDENCIES — Auditoria de Independência

Documento gerado na migração do projeto **ELEGANCE Barber Shop** para repositório
independente. Lista tudo que dependia da plataforma Emergent e o status após a limpeza.

> **Conclusão:** o projeto NÃO possui lock-in funcional. Era um site estático + 1 endpoint.
> Toda dependência da Emergent era de **telemetria/infra** e foi removida ou substituída.

---

## Dependências identificadas e tratadas

### 1. `emergent-main.js` (script de plataforma)
- **O que faz:** injeta badge/telemetria da Emergent no site.
- **Onde:** `frontend/public/index.html` (removido).
- **Substituível:** Sim.
- **Ação tomada:** ✅ **REMOVIDO** do `index.html`.
- **Impacto:** nenhum na funcionalidade do site.

### 2. PostHog apontando para `ap.emergent.sh`
- **O que faz:** analytics/session recording via projeto PostHog da Emergent.
- **Onde:** bloco `<script>` no final do `frontend/public/index.html` (removido).
- **Substituível:** Sim.
- **Ação tomada:** ✅ **REMOVIDO**. Se quiser analytics próprio, crie uma conta PostHog/GA
  e reinsira o snippet com a SUA key/host.
- **Impacto:** nenhum na funcionalidade; você perde apenas a telemetria da Emergent.

### 3. `emergentintegrations==0.2.2` (lib Python)
- **O que faz:** SDK interno da Emergent para LLMs. **Não é usado** pelo código.
- **Onde:** `backend/requirements.txt` (removido).
- **Substituível:** Sim (desnecessário).
- **Ação tomada:** ✅ **REMOVIDO**. Era um bloqueador real: não existe no PyPI, então
  `pip install -r requirements.txt` fora da Emergent falharia.
- **Impacto:** nenhum — nenhuma importação no projeto.

### 4. `REACT_APP_BACKEND_URL` apontando para o preview da Emergent
- **O que faz:** define a URL do backend que o frontend consome.
- **Onde:** `frontend/.env`.
- **Substituível:** Sim.
- **Ação tomada:** documentado em `frontend/.env.example`. No destino, aponte para a URL
  pública do SEU backend.
- **Impacto:** nenhum, desde que configurado corretamente.

### 5. Supervisor + Ingress Kubernetes (`/api` routing)
- **O que faz:** orquestra serviços e roteia `/api` → backend:8001 na infra Emergent.
- **Onde:** infraestrutura da plataforma (não versionado no repo).
- **Substituível:** Sim.
- **Ação tomada:** substituído por **`docker-compose.yml`** (mongo + backend + frontend/Nginx)
  e **`frontend/nginx.conf`** (SPA fallback).
- **Impacto:** nenhum — comportamento equivalente.

### 6. `@emergentbase/overlay` e `@emergentbase/visual-edits` (pacotes npm)
- **O que faziam:** overlay de erros e edição visual da Emergent, **apenas em modo dev**.
- **Onde:** `frontend/package.json` (devDependencies, URLs `assets.emergent.sh`) + `frontend/craco.config.js`.
- **Substituível:** Sim.
- **Ação tomada:** ✅ **REMOVIDOS COMPLETAMENTE** via `yarn remove` (package.json + yarn.lock) e todo o
  código exclusivo foi retirado do `craco.config.js`. Eram um bloqueador: as URLs apontavam para
  `assets.emergent.sh`, quebrando `yarn install` fora da Emergent.
- **Impacto:** nenhum em produção nem no dev server (compila normalmente).

### 7. Plugin `health-check` + `ENABLE_HEALTH_CHECK`
- **O que faz:** health check do preview da Emergent.
- **Onde:** `frontend/plugins/health-check/` + variável `ENABLE_HEALTH_CHECK`.
- **Substituível:** Sim.
- **Ação tomada:** mantido `ENABLE_HEALTH_CHECK=false` nos `.env.example` (desativado). Plugin local,
  sem dependência externa da Emergent.
- **Impacto:** nenhum.

---

## Dependências RESTANTES da Emergent

**Nenhuma.** Os pacotes `@emergentbase/*` foram removidos do `package.json`/`yarn.lock` e do
`craco.config.js`. O projeto roda 100% em infraestrutura própria (Cloudflare Pages + Railway/Render + MongoDB Atlas) sem qualquer chamada obrigatória à Emergent.
