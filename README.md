# Egg's Club

Monorepo local com **backend** (`backend-store` — Laravel) e **frontend** (`frontend-store` — Next.js).

**Desenvolvimento = só Docker.** Um único arquivo de configuração: **`.env` na raiz** (`eggs-club/.env`). O container do Laravel **não lê** `backend-store/.env`.

> Detalhes extras (hot-reload, deploy, e-mail): [DOCKER.md](./DOCKER.md)

---

## Pré-requisitos

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) instalado e **em execução**
- Terminal no WSL/Linux (ou equivalente)

---

## Primeira vez

Todos os comandos abaixo são na **raiz** do projeto:

```bash
cd /caminho/para/eggs-club
```

### 1. Criar o `.env` do Docker

```bash
cp .env.example .env
```

### 2. Definir a `APP_KEY`

Abra o arquivo `.env` da **raiz** e preencha `APP_KEY`.

Gere uma chave:

```bash
docker compose run --rm backend php artisan key:generate --show
```

Cole o valor em `APP_KEY=` no `.env` da raiz.

> Sem `APP_KEY`, o container do backend pode ficar reiniciando em loop.

### 3. Subir os containers (com build)

```bash
docker compose up --build
```

Na primeira vez o build pode levar **vários minutos**.

### 4. Acessar

| Serviço  | URL |
|----------|-----|
| Frontend | http://localhost:3000 |
| API      | http://localhost:8000/api |
| Health   | http://localhost:8000/up |

### 5. (Opcional) Usuários de teste

No `.env` da raiz:

```env
SEED_DATABASE=true
```

Depois:

```bash
docker compose up --build
```

Cria usuários de exemplo (senha `123456` — ver `backend-store/database/seeders/DatabaseSeeder.php`).

---

## Dia a dia

Sempre na **raiz** (`eggs-club/`):

### 1. Abrir o Docker Desktop

Deixe o Docker rodando antes de subir o projeto.

### 2. Iniciar o projeto

```bash
cd /caminho/para/eggs-club
docker compose up -d
```

### 3. Parar (sem apagar o banco)

```bash
docker compose down
```

### Quando usar `--build` de novo?

Só quando mudar dependências ou Docker:

- `composer.json` / `composer.lock`
- `package.json` / `package-lock.json`
- `Dockerfile` ou `docker-compose.yml`

```bash
docker compose up --build -d
```

---

## Sobre o `.env` (só um que importa)

| Arquivo | Com `docker compose` |
|---------|----------------------|
| **`eggs-club/.env`** | **Sim** — Laravel, MySQL, e-mail, `APP_KEY`, URLs |
| `backend-store/.env` | **Não** — ignore no dia a dia (legado / repo do backend no Git) |
| `frontend-store/.env` | **Não** — use `NEXT_PUBLIC_API_URL` no `.env` da raiz |

O `.env.example` da raiz já traz **todas** as variáveis do Laravel que estavam no `backend-store/.env.example`. O `docker-compose` só sobrescreve banco (`DB_HOST=mysql`) e log (`LOG_CHANNEL=stderr`).

Conferir o que o container enxerga:

```bash
docker compose exec backend php artisan tinker --execute="echo config('mail.default');"
```

**Nunca commite** `.env` com senhas — só `.env.example`.

---

## Comandos úteis

```bash
# Ver containers rodando
docker compose ps

# Ver logs
docker compose logs -f

# Logs só do backend
docker compose logs -f backend

# Migrations manualmente
docker compose exec backend php artisan migrate

# Apagar banco e volumes (cuidado!)
docker compose down -v
```

---

## Git (dois repositórios)

Se backend e frontend estão em **repos separados** no GitHub:

- Commitar em cada repo: `Dockerfile`, `.env.example` do respectivo projeto
- Nesta pasta raiz: `docker-compose.yml`, `.env.example`, `README.md` — em um repo de “dev” da equipe ou só local (ver [DOCKER.md](./DOCKER.md))

Branches: `git checkout` dentro de `backend-store/` ou `frontend-store/` funciona normalmente. Depois de trocar branch ou puxar mudanças grandes, rode `docker compose up --build -d` na raiz.

---

## Estrutura

```
eggs-club/
├── .env.example          # modelo para docker compose
├── docker-compose.yml
├── docker-compose.dev.yml
├── README.md             # este arquivo
├── DOCKER.md             # guia completo
├── backend-store/        # API Laravel
└── frontend-store/       # app Next.js
```
