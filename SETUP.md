# Guia de Setup Detalhado — Explora+

Este documento detalha o ambiente de desenvolvimento, as decisões de arquitetura e o fluxo de trabalho para os integrantes do grupo.

---

## Por que Docker?

Docker garante que o ambiente de todos os integrantes seja idêntico, independente do sistema operacional (Windows, Mac, Linux). Isso elimina o clássico problema de "na minha máquina funciona".

Cada serviço roda em seu próprio container isolado:

| Container  | O que roda                              | Porta local |
|------------|-----------------------------------------|-------------|
| `db`       | PostgreSQL 16 + PostGIS 3.4            | 5432        |
| `backend`  | Django 5.x + DRF                       | 8000        |
| `frontend` | Expo / React Native                    | 8081, 19000–19002 |

---

## Variáveis de ambiente

Todas as configurações sensíveis ficam no arquivo `.env` (nunca commitado no Git). O `.env.example` serve de template.

### Variáveis disponíveis

| Variável               | Descrição                                 | Padrão sugerido         |
|------------------------|-------------------------------------------|-------------------------|
| `DB_NAME`              | Nome do banco PostgreSQL                  | `explora_plus`          |
| `DB_USER`              | Usuário do banco                          | `explora_user`          |
| `DB_PASSWORD`          | Senha do banco                            | `explora_pass`          |
| `DB_HOST`              | Host do banco (dentro do Docker: `db`)    | `db`                    |
| `DB_PORT`              | Porta do banco                            | `5432`                  |
| `DJANGO_SECRET_KEY`    | Chave secreta do Django (mude em prod!)   | _(gere uma aleatória)_  |
| `DJANGO_DEBUG`         | Ativar modo debug                         | `True`                  |
| `DJANGO_ALLOWED_HOSTS` | Hosts permitidos                          | `localhost,127.0.0.1`   |
| `EXPO_PUBLIC_API_URL`  | URL do backend acessível pelo app         | `http://localhost:8000` |

### Como gerar uma SECRET_KEY segura

```bash
python3 -c "import secrets; print(secrets.token_urlsafe(50))"
```

---

## Fluxo de desenvolvimento

### Hot reload

Os volumes do Docker Compose montam o código do host dentro do container:

- **Backend:** qualquer alteração em `./backend/` é refletida instantaneamente (o Django detecta e reinicia automaticamente com `runserver`).
- **Frontend:** qualquer alteração em `./frontend/` é refletida pelo Fast Refresh do Expo.

Você **não** precisa reconstruir a imagem a cada alteração de código. O `--build` só é necessário quando você muda o `Dockerfile`, `requirements.txt` ou `package.json`.

### Adicionando dependências Python

```bash
# 1. Adicione ao backend/requirements.txt com versão fixada
echo "nova-lib==1.2.3" >> backend/requirements.txt

# 2. Reconstrua a imagem do backend
docker compose up --build backend
```

### Adicionando dependências Node

```bash
# 1. Instale via npm (será salvo no package.json)
docker compose exec frontend npm install nome-do-pacote

# 2. Ou reconstrua se preferir
docker compose up --build frontend
```

### Criando um novo app Django

```bash
docker compose exec backend python manage.py startapp nome_do_app
```

Lembre de adicionar `"nome_do_app"` em `INSTALLED_APPS` no `settings.py`.

### Rodando migrations após criar models

```bash
# Criar arquivo de migration
docker compose exec backend python manage.py makemigrations

# Aplicar ao banco
docker compose exec backend python manage.py migrate
```

---

## Acessando serviços

### Admin do Django

1. Crie um superusuário: `docker compose exec backend python manage.py createsuperuser`
2. Acesse: `http://localhost:8000/admin/`

### Health Check da API

```bash
curl -s http://localhost:8000/api/health/ | python3 -m json.tool
```

### Banco de dados diretamente

```bash
# Via psql
docker compose exec db psql -U explora_user -d explora_plus

# Comandos úteis dentro do psql:
# \dt         — listar tabelas
# \l          — listar bancos
# \q          — sair
```

---

## Expo Go no celular

### Opção 1: Rede local (recomendado para desenvolvimento)

O Expo detecta automaticamente o IP da máquina. Para que o celular acesse o backend:

1. Descubra o IP da sua máquina: `ip a` (Linux) / `ifconfig` (Mac) / `ipconfig` (Windows)
2. Edite `frontend/.env`:
   ```
   EXPO_PUBLIC_API_URL=http://SEU_IP:8000
   ```
3. Reinicie: `docker compose restart frontend`
4. Leia o QR code com o Expo Go

### Opção 2: Túnel (quando a rede local bloqueia)

Se estiver em uma rede que bloqueia conexões diretas (ex: rede universitária), use o modo túnel do Expo:

1. Edite o `CMD` no `frontend/Dockerfile`:
   ```dockerfile
   CMD ["npx", "expo", "start", "--tunnel"]
   ```
2. Reconstrua: `docker compose up --build frontend`

O modo túnel usa os servidores do Expo como relay, então funciona em qualquer rede, mas é um pouco mais lento.

---

## Estrutura do backend Django

```
backend/
├── explora_plus/         # Pacote do projeto Django
│   ├── settings.py       # Configurações (lê do .env via python-decouple)
│   ├── urls.py           # Roteamento raiz
│   ├── wsgi.py           # WSGI para deploy
│   └── asgi.py           # ASGI (futuro, para WebSockets)
├── core/                 # App inicial
│   ├── views.py          # Health check endpoint
│   └── urls.py           # Rotas do app core
├── manage.py
├── requirements.txt
├── Dockerfile
└── entrypoint.sh         # Script de inicialização do container
```

### Endpoints disponíveis

| Método | Rota          | Descrição                        | Auth |
|--------|---------------|----------------------------------|------|
| GET    | /api/health/  | Verifica DB e PostGIS            | Não  |
| GET    | /admin/       | Interface administrativa Django  | Sim  |

---

## Convenções do projeto

- **Python:** PEP 8, 4 espaços de indentação
- **TypeScript/JS:** 2 espaços de indentação
- **Commits:** mensagens em inglês no imperativo: `add`, `fix`, `update`, `remove`
- **Branches:** `feature/<descricao>`, `fix/<descricao>`
- **Nunca commite** arquivos `.env` — eles estão no `.gitignore`
