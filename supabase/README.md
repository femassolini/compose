# Supabase no Easypanel

Baseado no self-hosted oficial do Supabase (`supabase/supabase/docker`), adaptado para o Easypanel:

- `container_name` removido (conflita com o Easypanel)
- `ports` removidas (roteamento é feito pelos Domains do Easypanel)
- alias de rede `realtime-dev.supabase-realtime` adicionado ao serviço `realtime` (necessário para o Kong/Realtime, já que o `container_name` foi removido)

## Deploy no Easypanel (passo a passo)

### 1. Criar o serviço

1. No Easypanel: **New Service** → **Compose** → nome (ex.: `supabase`) → **Git**
2. Preencha:
   - **Repository URL**: `https://github.com/femassolini/compose`
   - **Branch**: `main`
   - **Build Path**: `/supabase/code`
   - **Docker Compose File**: `docker-compose.yml`
3. Na aba de **Environment**, o Easypanel importa automaticamente o `.env.example`.
   Substitua **todo o conteúdo** pelo seu `.env` gerado localmente
   (arquivo `supabase/code/.env` na sua máquina — ele contém senhas fortes já geradas)
   e altere as 3 ocorrências de `CHANGE_ME_YOUR_DOMAIN` para o domínio que você
   vai apontar para o Supabase (ex.: `https://supabase.seudominio.com`).
4. Mantenha **Create .env file** habilitado.
5. Clique em **Deploy** e aguarde todos os containers subir (o primeiro pull das
   imagens demora — são ~13 serviços).

> Requisitos: VPS com pelo menos 2 GB de RAM livres (ideal 4 GB) antes de subir o Easypanel + Supabase.

### 2. Domain (roteamento)

1. No serviço Compose: **Domains** → **Create Domain**
2. Domínio: `supabase.seudominio.com` (DNS A/AAAA apontando para a VPS, com proxy/HTTPS)
3. **Internal Service**: `kong` · **Port**: `8000` · **HTTPS**

Tudo passa pelo Kong na porta 8000:

- API: `https://seu-dominio/rest/v1`, `/auth/v1`, `/storage/v1`, `/realtime/v1`, `/functions/v1`
- Studio (dashboard): `https://seu-dominio/` ( pede Basic Auth )

### 3. Login do Studio

Abra `https://seu-dominio/` e autentique com o Basic Auth:

- **DASHBOARD_USERNAME** e **DASHBOARD_PASSWORD` do seu `.env`.

### 4. Credenciais da API

No `.env` (guarde bem, não exponha em frontend o `SERVICE_ROLE_KEY`):

- `ANON_KEY` — chave pública (frontend)
- `SERVICE_ROLE_KEY` — acesso total (somente backend)

### 5. E-mail (opcional no início)

Por padrão o `.env` gerado traz `ENABLE_EMAIL_AUTOCONFIRM=true`, então cadastros
funcionam sem SMTP. Para produção, configure um SMTP real e volte
`ENABLE_EMAIL_AUTOCONFIRM=false`.

## Acesso direto ao Postgres (limitação do Easypanel)

O Easypanel só publica HTTP(S) via Domains. Para conectar de fora na porta
5432/6543 (pooler `supavisor`) é preciso expor TCP via Traefik — fora do escopo
básico. Alternativas:

- Usar a API REST pública (`/rest/v1`) com `ANON_KEY`
- Rodar apps que precisam do banco na mesma VPS e conectar via rede interna do
  Docker (nome do serviço: `db:5432` ou `supavisor:6543`)

## Atualizar versões

Este fork é sincronizado com o upstream `easypanel-io/compose` (que por sua vez
segue o `supabase/supabase/docker`). As imagens estão pinadas em versões
testadas — não misture versões novas sem revisar `volumes/` e `kong.yml`.
