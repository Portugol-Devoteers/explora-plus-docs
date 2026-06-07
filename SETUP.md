# Guia de Setup Detalhado — Explora+

Este documento detalha o ambiente de desenvolvimento, as decisoes de arquitetura e o fluxo de trabalho para os integrantes do grupo.

Codigo e documentacao estao publicos na organizacao GitHub **EXPLORA-PLUS**: https://github.com/EXPLORA-PLUS

---

## Repositorios

O projeto esta dividido em tres repositorios independentes dentro da org:

| Repositorio | Papel |
|---|---|
| [`explora-plus-backend`](https://github.com/EXPLORA-PLUS/explora-plus-backend) | API Django + PostGIS — planner de rotas, catalogo de POIs, autenticacao |
| [`explora-plus-frontend`](https://github.com/EXPLORA-PLUS/explora-plus-frontend) | App Expo / React Native — interface do usuario, mapa, biblioteca de lugares |
| [`explora-plus-docs`](https://github.com/EXPLORA-PLUS/explora-plus-docs) | Documentacao, modelagem UML, diagramas Mermaid e paper academico |

Cada repositorio tem seu proprio `README.md` com instrucoes de setup especificas.

---

## Por que Docker?

Docker garante que o ambiente de todos os integrantes seja identico, independente do sistema operacional (Windows, Mac, Linux). Isso elimina o classico problema de "na minha maquina funciona".

Cada servico roda em seu proprio container isolado:

| Container  | O que roda                              | Porta host | Porta interna |
|------------|-----------------------------------------|------------|---------------|
| `db`       | PostgreSQL 16 + PostGIS 3.4            | 5433       | 5432          |
| `backend`  | Django 5.1 + DRF + Gunicorn            | 8080       | 8000          |
| `frontend` | Expo / React Native Web                | 8082       | 8081          |

---

## Variaveis de ambiente

Todas as configuracoes sensiveis ficam no arquivo `.env` (nunca commitado no Git). O `.env.example` serve de template.

### Backend (`explora-plus-backend/.env`)

| Variavel               | Descricao                                 | Padrao sugerido         |
|------------------------|-------------------------------------------|-------------------------|
| `DB_NAME`              | Nome do banco PostgreSQL                  | `explora_plus`          |
| `DB_USER`              | Usuario do banco                          | `explora_user`          |
| `DB_PASSWORD`          | Senha do banco                            | `explora_pass`          |
| `DB_HOST`              | Host do banco (dentro do Docker: `db`)    | `db`                    |
| `DB_PORT`              | Porta do banco (interna ao Docker)        | `5432`                  |
| `DJANGO_SECRET_KEY`    | Chave secreta do Django (mude em prod!)   | _(gere uma aleatoria)_  |
| `DJANGO_DEBUG`         | Ativar modo debug                         | `True`                  |
| `DJANGO_ALLOWED_HOSTS` | Hosts permitidos                          | `localhost,127.0.0.1,backend` |

### Frontend (`explora-plus-frontend/.env`)

| Variavel               | Descricao                                 | Valor recomendado       |
|------------------------|-------------------------------------------|-------------------------|
| `EXPO_PUBLIC_API_URL`  | URL do backend acessivel pelo app         | `http://localhost:8080` |

> **Atencao:** o `.env.example` historico pode estar com porta `8000`. Use `8080` no setup atual.

### Preferencias de busca por usuario

As configuracoes do planner ficam no backend e sao persistidas por usuario autenticado.

Defaults atuais:

- `include_culture = true`
- `include_park = true`
- `include_food = true`
- `poi_spacing_m = 100`
- `max_search_radius_m = 250`

Presets aceitos:

- distancia entre POIs: `75`, `100`, `150`
- raio maximo de busca: `150`, `250`, `400`

Regra importante:

- salvar essas preferencias **nao** recalcula a rota atual
- elas passam a valer na proxima vez que o usuario tocar em `Gerar rota`

### Como gerar uma SECRET_KEY segura

```bash
python3 -c "import secrets; print(secrets.token_urlsafe(50))"
```

---

## Subindo o projeto

### Backend

```bash
cd explora-plus-backend
docker compose up --build -d
docker compose exec backend python manage.py migrate
docker compose exec backend python manage.py seed_demo --reset
```

API disponivel em: `http://localhost:8080/api/health/`

### Frontend

```bash
cd explora-plus-frontend
npm install
npm run web
# ou via Docker:
docker compose up --build -d
# Frontend em http://localhost:8082
```

---

## Fluxo de desenvolvimento

### Hot reload

Os volumes do Docker Compose montam o codigo do host dentro do container:

- **Backend:** qualquer alteracao em `./` e refletida instantaneamente (Django detecta e reinicia com runserver).
- **Frontend:** qualquer alteracao e refletida pelo Fast Refresh do Expo.

O `--build` so e necessario quando voce muda o `Dockerfile`, `requirements.txt` ou `package.json`.

### Adicionando dependencias Python

```bash
# 1. Adicione ao requirements.txt com versao fixada
echo "nova-lib==1.2.3" >> requirements.txt

# 2. Reconstrua a imagem do backend
docker compose up --build backend
```

### Adicionando dependencias Node

```bash
docker compose exec frontend npm install nome-do-pacote
# ou reconstrua:
docker compose up --build frontend
```

### Criando um novo app Django

```bash
docker compose exec backend python manage.py startapp nome_do_app
```

Lembre de adicionar `"nome_do_app"` em `INSTALLED_APPS` no `settings/base.py`.

### Rodando migrations

```bash
docker compose exec backend python manage.py makemigrations
docker compose exec backend python manage.py migrate
# Verificar que nao ha migrations pendentes:
docker compose exec backend python manage.py makemigrations --check --dry-run
```

---

## Acessando servicos

### Admin do Django

```
http://localhost:8080/admin/
```

Criar superusuario:
```bash
docker compose exec backend python manage.py createsuperuser
```

### Health Check da API

```bash
curl -s http://localhost:8080/api/health/ | python3 -m json.tool
```

Resposta esperada:
```json
{"status": "ok", "db": "ok", "postgis": "..."}
```

### Banco de dados diretamente

```bash
docker compose exec db psql -U explora_user -d explora_plus
# \dt  - listar tabelas
# \l   - listar bancos
# \q   - sair
```

Porta externa do PostgreSQL: `localhost:5433`

---

## Seeds e dados de demonstracao

```bash
# Popula categorias canonicas (culture, park, food) e lugares curados de Santos
docker compose exec backend python manage.py seed_demo --reset
```

O `--reset` apaga `PlaceImage` e `Place` antes de repopular.

---

## Testes

### Suite principal (tour_routes)

```bash
docker compose exec backend python manage.py test tour_routes.tests \
  --settings=explora_plus.settings.test \
  --verbosity 2 --noinput
```

### Verificacao de tipos (frontend)

```bash
npm exec tsc -- --noEmit
```

---

## Estrutura do backend Django

```
explora-plus-backend/
|-- accounts/              # registro, login, refresh, /api/me/
|-- core/                  # health check e constantes de dominio
|-- places/                # dominio canonico de lugares e estados do usuario
|   |-- management/commands/
|   |   `-- seed_demo.py   # dados curados de demonstracao
|   `-- catalog.py         # upsert de lugares a partir de payloads
|-- tickets/               # endpoint isolado de ingressos mockados
|-- tour_routes/           # planner, cache, rota atual, biblioteca e preferencias
|   |-- services/          # geocoding, routing, Overpass, Wikidata, Wikipedia, map builder
|   `-- tests/             # testes do fluxo de rotas
|-- routes/                # legado, fora do caminho ativo
|-- explora_plus/          # configuracao Django e roteamento raiz
|   `-- settings/
|       |-- base.py        # configuracoes compartilhadas
|       |-- docker.py      # configuracoes especificas do Docker (default do manage.py)
|       `-- test.py        # configuracoes para suite de testes
|-- docker-compose.yml
|-- Dockerfile
|-- entrypoint.sh
|-- manage.py
`-- requirements.txt
```

### Endpoints disponiveis

| Metodo | Rota | Auth | Papel |
|--------|------|------|-------|
| GET    | `/api/health/` | Nao | Verifica DB e PostGIS |
| POST   | `/api/auth/register/` | Nao | Cria usuario |
| POST   | `/api/auth/login/` | Nao | Login JWT |
| POST   | `/api/auth/refresh/` | Nao | Renova access token |
| GET    | `/api/me/` | Sim | Usuario atual |
| GET    | `/api/places/` | Nao | Lista lugares ativos |
| GET    | `/api/places/<slug>/` | Nao | Detalhe de lugar |
| POST   | `/api/tour-routes/` | Opcional | Calcula rota; salva se autenticado |
| GET    | `/api/tour-routes/current/` | Sim | Rota mais recente do usuario |
| GET    | `/api/tour-routes/preferences/` | Sim | Le preferencias salvas ou defaults do planner |
| PATCH  | `/api/tour-routes/preferences/` | Sim | Salva preferencias para a proxima busca |
| GET    | `/api/tour-routes/places/` | Sim | Biblioteca pessoal de lugares |
| GET    | `/api/tour-routes/pois/<stop_id>/` | Nao | Detalhe enriquecido de POI |
| PATCH  | `/api/tour-routes/places/<stop_id>/visited/` | Sim | Marca/desmarca visitado |
| DELETE | `/api/tour-routes/saved/<route_id>/stops/<stop_id>/` | Sim | Exclui stop da rota |
| PATCH  | `/api/tour-routes/saved/<route_id>/stops/<stop_id>/state/` | Sim | Muda estado publico do stop |
| GET    | `/admin/` | Sim | Interface administrativa Django |

### Fluxo das preferencias

1. `ProfileScreen` abre `SearchSettingsScreen`.
2. O frontend chama `GET /api/tour-routes/preferences/`.
3. O usuario altera categorias, distancia entre POIs e raio maximo.
4. O frontend chama `PATCH /api/tour-routes/preferences/`.
5. O backend salva `UserRouteSearchPreference`.
6. A rota atual continua igual.
7. A proxima chamada de `POST /api/tour-routes/` passa a usar as novas preferencias e gera um `cache_key` diferente quando necessario.

---

## Expo Go no celular

### Opcao 1: Rede local (recomendado)

1. Descubra o IP da sua maquina: `ipconfig` (Windows) / `ifconfig` (Mac/Linux)
2. Edite `explora-plus-frontend/.env`:
   ```
   EXPO_PUBLIC_API_URL=http://SEU_IP:8080
   ```
3. Reinicie: `docker compose restart frontend`
4. Leia o QR code com o Expo Go

### Opcao 2: Tunel (quando a rede local bloqueia)

Se estiver em uma rede que bloqueia conexoes diretas (ex: rede universitaria):

1. Edite o `CMD` no `frontend/Dockerfile`:
   ```dockerfile
   CMD ["npx", "expo", "start", "--tunnel"]
   ```
2. Reconstrua: `docker compose up --build frontend`

---

## Convencoes do projeto

- **Python:** PEP 8, 4 espacos de indentacao
- **TypeScript/JS:** 2 espacos de indentacao
- **Commits:** mensagens em ingles no imperativo: `add`, `fix`, `update`, `remove`
- **Branches:** `feature/<descricao>`, `fix/<descricao>`
- **Nunca commite** arquivos `.env` -- eles estao no `.gitignore`
