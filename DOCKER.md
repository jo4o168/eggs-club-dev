# Egg's Club — Docker (guia completo)

Comandos de **primeira vez** e **dia a dia**: veja o [README.md](./README.md).

Com Docker instalado, qualquer pessoa da equipe sobe **MySQL + API Laravel + fila + frontend Next.js** sem instalar PHP, Composer, Node ou MySQL na máquina.

## Modo desenvolvimento (hot-reload)

Monta o código local nos containers e usa `npm run dev` / `artisan serve`:

```bash
docker compose -f docker-compose.yml -f docker-compose.dev.yml up --build
```

## E-mail (recuperação de senha)

Por padrão o driver é `log` (mensagens aparecem nos logs do backend). Para SMTP (Gmail etc.), configure no `.env` da raiz as variáveis `MAIL_*` (veja `.env.example`).

## Deploy (backend e frontend em repositórios separados)

O `docker-compose.yml` na raiz é padrão de mercado para **desenvolvimento local** (time inteiro sobe o mesmo stack). Em **produção** quase ninguém sobe esse compose igual — cada serviço vai para um host/PaaS com a imagem Docker dele.

Fluxo típico com 2 repos:

| Repo | O que sobe | Variáveis |
|------|------------|-----------|
| **backend-store** | Imagem do `Dockerfile` → Railway, Render, ECS, VPS | Painel do provedor (= antigo `.env`, sem arquivo no disco) |
| **frontend-store** | Imagem Next.js | `NEXT_PUBLIC_API_URL=https://api.seudominio.com/api` no build |
| **Banco** | MySQL gerenciado (RDS, PlanetScale, etc.) | `DB_HOST`, `DB_PASSWORD` no backend |

O `.env` da raiz **não vai para produção** — só orienta o compose local. No servidor você configura as mesmas chaves no painel do provedor.

Opções comuns:

1. **Dois deploys** — API em um serviço, front em outro (Vercel no front + VPS na API é comum).
2. **Repo `eggs-club-infra`** (opcional) — só `docker-compose.yml` + `.env.example` para a faculdade clonar os 3 juntos.
3. **Kubernetes / ECS** — um manifest por serviço, secrets no cluster.

Checklist produção:

- `APP_ENV=production`, `APP_DEBUG=false`
- `APP_KEY` fixo (secret)
- `FRONTEND_URL` e `NEXT_PUBLIC_API_URL` com domínio real
- SMTP real em `MAIL_*`
- Migrations no CI ou no start do container (`php artisan migrate --force`)

## Estrutura

```
eggs-club/
├── docker-compose.yml          # stack padrão (build de produção)
├── docker-compose.dev.yml      # override com hot-reload
├── .env.example                # variáveis do compose
├── backend-store/Dockerfile
└── frontend-store/Dockerfile
```
