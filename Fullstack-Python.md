Boa! Criar uma aplicação **fullstack** com Python significa desenvolver **backend + frontend + banco de dados**, integrando tudo em um sistema único. Vou detalhar passo a passo como você pode estruturar um projeto desse tipo:

---

# 🔹 1. Conceitos básicos

* **Backend (server-side)** → escrito em **Python** (Flask, Django ou FastAPI). Responsável pela **lógica de negócio, APIs e banco de dados**.
* **Frontend (client-side)** → interface visual, geralmente em **HTML, CSS, JavaScript** (puro ou frameworks como React, Vue, Angular).
* **Banco de Dados** → pode ser **relacional** (PostgreSQL, MySQL, SQLite) ou **NoSQL** (MongoDB).

---

# 🔹 2. Estrutura mínima de um projeto Fullstack em Python

```
meu_projeto_fullstack/
│
├─ backend/              # Aplicação Python (API REST)
│  ├─ app.py
│  ├─ models.py
│  ├─ routes.py
│  └─ requirements.txt
│
├─ frontend/             # Interface (React, Vue ou HTML/CSS/JS)
│  ├─ index.html
│  ├─ src/
│  └─ package.json
│
└─ docker-compose.yml    # (opcional) para rodar tudo junto
```

---

# 🔹 3. Criando o **backend em Python** (Flask + API REST)

### Exemplo `backend/app.py`

```python
from flask import Flask, jsonify, request
from flask_sqlalchemy import SQLAlchemy
from flask_cors import CORS

app = Flask(__name__)
CORS(app)  # permite que o frontend acesse a API

app.config["SQLALCHEMY_DATABASE_URI"] = "sqlite:///dados.db"
app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False

db = SQLAlchemy(app)

# Modelo
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    nome = db.Column(db.String(80), nullable=False)

# Rotas API
@app.route("/users", methods=["GET"])
def listar_usuarios():
    usuarios = User.query.all()
    return jsonify([{"id": u.id, "nome": u.nome} for u in usuarios])

@app.route("/users", methods=["POST"])
def criar_usuario():
    data = request.json
    novo = User(nome=data["nome"])
    db.session.add(novo)
    db.session.commit()
    return jsonify({"id": novo.id, "nome": novo.nome}), 201

if __name__ == "__main__":
    with app.app_context():
        db.create_all()
    app.run(debug=True)
```

👉 Isso cria uma **API REST** que expõe endpoints para o frontend consumir.

---

# 🔹 4. Criando o **frontend**

Você pode usar:

* **HTML/CSS/JS simples** (para apps pequenos).
* **React.js** (mais moderno e escalável).

### Exemplo simples em `frontend/index.html`:

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>App Fullstack Python</title>
</head>
<body>
  <h1>Usuários</h1>
  <ul id="lista"></ul>
  <input id="nome" placeholder="Novo usuário">
  <button onclick="criar()">Adicionar</button>

  <script>
    async function carregar() {
      let res = await fetch("http://127.0.0.1:5000/users");
      let data = await res.json();
      let lista = document.getElementById("lista");
      lista.innerHTML = "";
      data.forEach(u => {
        let li = document.createElement("li");
        li.textContent = u.nome;
        lista.appendChild(li);
      });
    }

    async function criar() {
      let nome = document.getElementById("nome").value;
      await fetch("http://127.0.0.1:5000/users", {
        method: "POST",
        headers: {"Content-Type": "application/json"},
        body: JSON.stringify({nome})
      });
      carregar();
    }

    carregar();
  </script>
</body>
</html>
```

👉 O **frontend consome a API Flask** e exibe os dados dinamicamente.

---

# 🔹 5. Banco de dados

* **SQLite** (mais simples, usado acima).
* Para produção → **PostgreSQL** (muito comum com Python).

Se usar Docker, pode rodar tudo junto: Flask + PostgreSQL + React.

---

# 🔹 6. Deploy

* **Local**: Flask + frontend rodando em localhost.
* **Produção**:

  * Use **Gunicorn** ou **Uvicorn** (backend).
  * Sirva o **frontend buildado** no Nginx.
  * Gerencie tudo com **Docker Compose** ou Kubernetes.

---

# 🔹 7. Resumindo o fluxo

1. Usuário acessa o **frontend** (HTML/React).
2. O frontend faz requisições **HTTP (fetch/axios)** para a **API Flask**.
3. O **backend** processa os dados, acessa o **banco de dados** e devolve a resposta em JSON.
4. O **frontend atualiza a interface** dinamicamente.

---

👉 Esse é o esqueleto de um **fullstack em Python (Flask + Frontend + DB)**.
Se quiser, eu posso montar um **exemplo completo com React no frontend e Flask no backend**, usando **Docker** para integrar tudo.

Quer que eu faça um guia com **React + Flask + PostgreSQL (Docker)**, que é o padrão profissional em produção?


A seguir, um guia objetivo para criar um stack **React + Flask + PostgreSQL** com **Docker Compose** (ambiente de desenvolvimento) e um roteiro de build para produção.

---

# 1) Estrutura de pastas

```
meu_app_fullstack/
├─ docker-compose.yml
├─ .env                 # variáveis comuns (proxy, portas, etc.)
├─ backend/
│  ├─ Dockerfile
│  ├─ entrypoint.sh
│  ├─ requirements.txt
│  └─ app/
│     ├─ __init__.py
│     ├─ models.py
│     ├─ routes.py
│     └─ wsgi.py
└─ frontend/
   ├─ Dockerfile
   ├─ package.json
   ├─ vite.config.js
   └─ src/
      ├─ main.jsx
      └─ App.jsx
```

---

# 2) Variáveis de ambiente

Crie **.env** na raiz:

```
# App
PROJECT_NAME=meu_app

# Backend
BACKEND_PORT=5000
BACKEND_WORKERS=2

# Frontend
FRONTEND_PORT=5173

# DB
POSTGRES_USER=app_user
POSTGRES_PASSWORD=app_pass
POSTGRES_DB=app_db
POSTGRES_PORT=5432

# Alembic/Flask config
DATABASE_URL=postgresql+psycopg2://app_user:app_pass@db:5432/app_db
FLASK_ENV=development
FLASK_APP=app:wsgi_app
SECRET_KEY=dev-secret-change-me
CORS_ORIGINS=http://localhost:5173
```

> Observação: **DATABASE\_URL** usa o host `db` (nome do serviço Postgres no Compose).

---

# 3) Docker Compose (desenvolvimento)

Crie **docker-compose.yml** na raiz:

```yaml
version: "3.9"
services:
  db:
    image: postgres:16-alpine
    container_name: ${PROJECT_NAME}-db
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    ports:
      - "${POSTGRES_PORT}:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 5s
      timeout: 5s
      retries: 10

  backend:
    build: ./backend
    container_name: ${PROJECT_NAME}-backend
    depends_on:
      db:
        condition: service_healthy
    environment:
      DATABASE_URL: ${DATABASE_URL}
      SECRET_KEY: ${SECRET_KEY}
      CORS_ORIGINS: ${CORS_ORIGINS}
      FLASK_ENV: ${FLASK_ENV}
      BACKEND_WORKERS: ${BACKEND_WORKERS}
    ports:
      - "${BACKEND_PORT}:5000"
    volumes:
      - ./backend:/app
    command: ["/bin/sh", "/start"]

  frontend:
    build: ./frontend
    container_name: ${PROJECT_NAME}-frontend
    environment:
      VITE_API_URL: http://localhost:${BACKEND_PORT}
    ports:
      - "${FRONTEND_PORT}:5173"
    volumes:
      - ./frontend:/usr/src/app
      - /usr/src/app/node_modules
    command: ["npm", "run", "dev", "--", "--host"]

volumes:
  pgdata:
```

---

# 4) Backend Flask

## 4.1 requirements.txt

```
Flask==3.0.3
Flask-Cors==4.0.1
Flask-Migrate==4.0.7
Flask-SQLAlchemy==3.1.1
psycopg2-binary==2.9.9
SQLAlchemy==2.0.31
gunicorn==22.0.0
python-dotenv==1.0.1
```

## 4.2 Dockerfile (backend/Dockerfile)

```dockerfile
FROM python:3.12-slim
WORKDIR /app
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1
RUN apt-get update && apt-get install -y build-essential && rm -rf /var/lib/apt/lists/*
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt
COPY . /app
RUN chmod +x /app/entrypoint.sh && ln -s /app/entrypoint.sh /start
EXPOSE 5000
```

## 4.3 entrypoint.sh (backend/entrypoint.sh)

```bash
#!/usr/bin/env sh
set -e
# Aguarda DB
python - <<'PY'
import time, os
import psycopg2
from urllib.parse import urlparse
url = os.environ['DATABASE_URL']
parsed = urlparse(url.replace('+psycopg2',''))
for _ in range(50):
    try:
        conn = psycopg2.connect(dbname=parsed.path[1:], user=parsed.username, password=parsed.password, host=parsed.hostname, port=parsed.port)
        conn.close()
        break
    except Exception:
        time.sleep(1)
else:
    raise SystemExit('DB não respondeu a tempo')
PY
# Migrações
flask db upgrade || flask db init && flask db migrate -m "init" && flask db upgrade
# Servidor
exec gunicorn -w ${BACKEND_WORKERS:-2} -b 0.0.0.0:5000 app.wsgi:wsgi_app
```

## 4.4 app/**init**.py

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
from flask_cors import CORS
import os

db = SQLAlchemy()
migrate = Migrate()

def create_app():
    app = Flask(__name__)

    app.config["SQLALCHEMY_DATABASE_URI"] = os.getenv("DATABASE_URL", "sqlite:///app.db")
    app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False
    app.config["SECRET_KEY"] = os.getenv("SECRET_KEY", "dev")

    CORS(app, resources={r"/*": {"origins": os.getenv("CORS_ORIGINS", "*")}})

    db.init_app(app)
    migrate.init_app(app, db)

    from .models import User, Task  # noqa: F401
    from .routes import api_bp
    app.register_blueprint(api_bp, url_prefix="/api")

    @app.get("/health")
    def health():
        return {"status": "ok"}

    return app
```

## 4.5 app/models.py

```python
from datetime import datetime
from . import db

class User(db.Model):
    __tablename__ = "users"
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(120), nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow, nullable=False)

class Task(db.Model):
    __tablename__ = "tasks"
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(160), nullable=False)
    done = db.Column(db.Boolean, default=False, nullable=False)
    user_id = db.Column(db.Integer, db.ForeignKey("users.id"), nullable=True)
    created_at = db.Column(db.DateTime, default=datetime.utcnow, nullable=False)
```

## 4.6 app/routes.py

```python
from flask import Blueprint, request
from . import db
from .models import User, Task

api_bp = Blueprint("api", __name__)

# Users
@api_bp.get("/users")
def list_users():
    users = User.query.order_by(User.created_at.desc()).all()
    return {"users": [{"id": u.id, "name": u.name, "email": u.email} for u in users]}

@api_bp.post("/users")
def create_user():
    data = request.get_json() or {}
    if not data.get("name") or not data.get("email"):
        return {"error": "name e email são obrigatórios"}, 400
    user = User(name=data["name"], email=data["email"])
    db.session.add(user)
    db.session.commit()
    return {"id": user.id, "name": user.name, "email": user.email}, 201

# Tasks
@api_bp.get("/tasks")
def list_tasks():
    tasks = Task.query.order_by(Task.created_at.desc()).all()
    return {"tasks": [{"id": t.id, "title": t.title, "done": t.done, "user_id": t.user_id} for t in tasks]}

@api_bp.post("/tasks")
def create_task():
    data = request.get_json() or {}
    if not data.get("title"):
        return {"error": "title é obrigatório"}, 400
    task = Task(title=data["title"], user_id=data.get("user_id"))
    db.session.add(task)
    db.session.commit()
    return {"id": task.id, "title": task.title, "done": task.done, "user_id": task.user_id}, 201

@api_bp.post("/tasks/<int:task_id>/toggle")
def toggle_task(task_id: int):
    task = Task.query.get_or_404(task_id)
    task.done = not task.done
    db.session.commit()
    return {"id": task.id, "done": task.done}

@api_bp.delete("/tasks/<int:task_id>")
def delete_task(task_id: int):
    task = Task.query.get_or_404(task_id)
    db.session.delete(task)
    db.session.commit()
    return {"ok": True}
```

## 4.7 app/wsgi.py

```python
from . import create_app
wsgi_app = create_app()
```

---

# 5) Frontend (React + Vite)

## 5.1 Dockerfile (frontend/Dockerfile)

```dockerfile
FROM node:20-alpine
WORKDIR /usr/src/app
COPY package.json package-lock.json* ./
RUN npm ci || npm i
COPY . .
EXPOSE 5173
```

## 5.2 package.json (frontend/package.json)

```json
{
  "name": "frontend",
  "private": true,
  "version": "0.0.1",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview --host"
  },
  "dependencies": {
    "react": "^18.3.1",
    "react-dom": "^18.3.1"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.3.1",
    "vite": "^5.4.0"
  }
}
```

## 5.3 vite.config.js

```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    port: 5173,
    strictPort: true
  }
})
```

## 5.4 src/main.jsx

```jsx
import React from 'react'
import { createRoot } from 'react-dom/client'
import App from './App'

createRoot(document.getElementById('root')).render(<App />)
```

## 5.5 src/App.jsx

```jsx
import React, { useEffect, useState } from 'react'

const API = import.meta.env.VITE_API_URL || 'http://localhost:5000'

export default function App() {
  const [users, setUsers] = useState([])
  const [tasks, setTasks] = useState([])
  const [name, setName] = useState('')
  const [email, setEmail] = useState('')
  const [title, setTitle] = useState('')

  async function load() {
    const [u, t] = await Promise.all([
      fetch(`${API}/api/users`).then(r => r.json()),
      fetch(`${API}/api/tasks`).then(r => r.json())
    ])
    setUsers(u.users || [])
    setTasks(t.tasks || [])
  }

  async function addUser() {
    if (!name || !email) return
    await fetch(`${API}/api/users`, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ name, email }) })
    setName(''); setEmail(''); load()
  }

  async function addTask() {
    if (!title) return
    await fetch(`${API}/api/tasks`, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ title }) })
    setTitle(''); load()
  }

  async function toggleTask(id) {
    await fetch(`${API}/api/tasks/${id}/toggle`, { method: 'POST' })
    load()
  }

  async function deleteTask(id) {
    await fetch(`${API}/api/tasks/${id}`, { method: 'DELETE' })
    load()
  }

  useEffect(() => { load() }, [])

  return (
    <div style={{ fontFamily: 'system-ui', padding: 24, maxWidth: 900, margin: '0 auto' }}>
      <h1>Fullstack: React + Flask + Postgres</h1>

      <section>
        <h2>Usuários</h2>
        <input placeholder="Nome" value={name} onChange={e => setName(e.target.value)} />
        <input placeholder="Email" value={email} onChange={e => setEmail(e.target.value)} />
        <button onClick={addUser}>Adicionar</button>
        <ul>
          {users.map(u => <li key={u.id}>{u.name} — {u.email}</li>)}
        </ul>
      </section>

      <section>
        <h2>Tarefas</h2>
        <input placeholder="Título" value={title} onChange={e => setTitle(e.target.value)} />
        <button onClick={addTask}>Adicionar</button>
        <ul>
          {tasks.map(t => (
            <li key={t.id}>
              <button onClick={() => toggleTask(t.id)}>{t.done ? '✅' : '⬜'}</button>
              <span style={{ textDecoration: t.done ? 'line-through' : 'none', margin: '0 8px' }}>{t.title}</span>
              <button onClick={() => deleteTask(t.id)}>🗑️</button>
            </li>
          ))}
        </ul>
      </section>
    </div>
  )
}
```

## 5.6 index.html (Vite cria automaticamente). Certifique-se de ter `<div id="root"></div>`.

---

# 6) Inicialização

```bash
# 1. Build e subida dos serviços
docker compose up --build -d

# 2. Verifique saúde
curl http://localhost:5000/health  # => {"status":"ok"}

# 3. Acesse o frontend
http://localhost:5173
```

> Ao alterar código (backend/frontend), os containers em dev recarregam (volumes montados).

---

# 7) Migrações de banco (Flask-Migrate)

Dentro do container backend (apenas quando você mudar o schema):

```bash
docker compose exec backend flask db migrate -m "add new field"
docker compose exec backend flask db upgrade
```

---

# 8) Produção (resumo)

**Opção A (simples):**

* Backend: `gunicorn` (já configurado), atrás de **Nginx** como reverse proxy.
* Frontend: executar `npm run build` e servir `/dist` via **Nginx**.
* Banco: **Postgres** gerenciado (RDS, Cloud SQL) com backup/monitoramento.

**Opção B (containers dedicados):**

* Adicionar um serviço `nginx` no Compose com:

  * `location /api` → proxy para `backend:5000`
  * `location /` → servir os arquivos estáticos do build React.

Checklist de produção:

* Variáveis seguras (sem segredos em VCS).
* HTTPS (TLS) terminado no Nginx/Load Balancer.
* Logs estruturados + Healthchecks.
* Alembic/Flask-Migrate rodando em job/entrypoint.
* CORS restrito ao domínio do frontend.
* Backups e política de retenção do Postgres.

---

# 9) Teste rápido da API

```bash
curl -X POST http://localhost:5000/api/users \
  -H 'Content-Type: application/json' \
  -d '{"name":"Alice","email":"alice@acme.com"}'

curl http://localhost:5000/api/users
```

---

# 10) Próximos passos

* Autenticação JWT (Flask-JWT-Extended).
* Validação/serialização (pydantic ou marshmallow).
* Paginação, filtros e ordenação.
* Camada de serviços e testes (pytest + coverage).
* CI/CD (GitH

Continuação do guia, focando em **Nginx para produção**, build do frontend, pipeline CI/CD, backups e segurança.

---

# 11) Nginx para produção (proxy + estáticos)

## 11.1 docker-compose.prod.yml

Use um compose separado para produção, adicionando **nginx** e buildando o frontend.

```yaml
version: "3.9"
services:
  db:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - pgdata:/var/lib/postgresql/data

  backend:
    build: ./backend
    environment:
      DATABASE_URL: ${DATABASE_URL}
      SECRET_KEY: ${SECRET_KEY}
      CORS_ORIGINS: ${PUBLIC_URL}
      FLASK_ENV: production
      BACKEND_WORKERS: ${BACKEND_WORKERS:-4}
    depends_on:
      - db

  frontend:
    build:
      context: ./frontend
      target: build
    environment:
      VITE_API_URL: ${PUBLIC_API_URL}

  nginx:
    image: nginx:1.27-alpine
    depends_on:
      - backend
      - frontend
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./frontend/dist:/usr/share/nginx/html:ro
    ports:
      - "80:80"
      # - "443:443"  # habilite para TLS

volumes:
  pgdata:
```

> Dica: você pode manter um `docker-compose.yml` para desenvolvimento e `docker-compose.prod.yml` para produção, com `.env` distintos.

## 11.2 Nginx config (nginx/nginx.conf)

```nginx
worker_processes auto;
error_log /var/log/nginx/error.log warn;

events { worker_connections 1024; }

http {
  include       /etc/nginx/mime.types;
  default_type  application/octet-stream;
  sendfile      on;
  keepalive_timeout  65;
  gzip on;
  gzip_types text/plain text/css application/javascript application/json image/svg+xml;

  upstream backend_upstream {
    server backend:5000;
  }

  server {
    listen 80;
    server_name _;

    # Arquivos estáticos do React (build)
    root /usr/share/nginx/html;
    index index.html;

    location /api/ {
      proxy_pass http://backend_upstream/;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
    }

    # React Router fallback
    location / {
      try_files $uri /index.html;
      add_header Cache-Control "public, max-age=3600";
    }

    # Segurança básica
    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options DENY;
    add_header X-XSS-Protection "1; mode=block";
  }
}
```

Para TLS (HTTPS), crie um `server` em **443** com certificados (Let's Encrypt/Certbot) e redirecione `80 -> 443`.

---

# 12) Frontend: build (produção)

## 12.1 Dockerfile (frontend/Dockerfile) — multi-stage

```dockerfile
# Etapa 1: build
FROM node:20-alpine AS build
WORKDIR /usr/src/app
COPY package.json package-lock.json* ./
RUN npm ci || npm i
COPY . .
ARG VITE_API_URL
ENV VITE_API_URL=${VITE_API_URL}
RUN npm run build

# (Opcional) publique artefatos
FROM alpine AS export
WORKDIR /export
COPY --from=build /usr/src/app/dist ./dist
```

**Alternativa simples**: build local — execute `npm run build` no `frontend` e mapeie `./frontend/dist` para o Nginx como mostrado.

---

# 13) Backend: ajustes de produção

* `BACKEND_WORKERS`: 2–4 por CPU (ajuste com teste de carga).
* Restringir **CORS** ao domínio do app (`CORS_ORIGINS=https://meudominio.com`).
* `SECRET_KEY` forte e fora do repositório.
* Logs estruturados (STDOUT/ERR) para o orquestrador coletar.

**Comando gunicorn típico**:

```bash
exec gunicorn \
  --workers ${BACKEND_WORKERS:-4} \
  --bind 0.0.0.0:5000 \
  --access-logfile - --error-logfile - \
  app.wsgi:wsgi_app
```

---

# 14) .env.prod (exemplo)

```
PROJECT_NAME=meu_app
PUBLIC_URL=https://meudominio.com
PUBLIC_API_URL=https://meudominio.com
BACKEND_WORKERS=4
POSTGRES_USER=app_user
POSTGRES_PASSWORD=troque_me
POSTGRES_DB=app_db
DATABASE_URL=postgresql+psycopg2://app_user:troque_me@db:5432/app_db
SECRET_KEY=coloque_um_uuid_aleatorio
```

Rodar:

```bash
docker compose -f docker-compose.prod.yml --env-file .env.prod up --build -d
```

---

# 15) Pipeline CI/CD (GitHub Actions) — esqueleto

`.github/workflows/ci.yml`:

```yaml
name: CI
on: [push]
jobs:
  build-test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test_db
        ports: ["5432:5432"]
        options: >-
          --health-cmd pg_isready
          --health-interval 5s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: '3.12' }
      - name: Backend deps
        run: |
          pip install -r backend/requirements.txt
          pip install pytest
      - name: Testes backend
        env:
          DATABASE_URL: postgresql+psycopg2://postgres:postgres@localhost:5432/test_db
          SECRET_KEY: test
        run: |
          python - <<'PY'
from backend.app import create_app, db
app = create_app()
with app.app_context():
    db.create_all()
print('DB OK')
PY
          pytest -q || true
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - name: Frontend deps & build
        working-directory: frontend
        run: |
          npm ci || npm i
          npm run build
```

Para CD, adicione: login no registry, `docker buildx build --push` e etapa de deploy (SSH/Compose, ECS, etc.).

---

# 16) Backups & manutenção do Postgres

## 16.1 Backup simples

`scripts/backup.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail
: "${POSTGRES_USER?}" "${POSTGRES_PASSWORD?}" "${POSTGRES_DB?}" "${POSTGRES_HOST:=localhost}" "${POSTGRES_PORT:=5432}"
DATE=$(date +%Y%m%d-%H%M)
FILE=backup-$POSTGRES_DB-$DATE.sql.gz
PGPASSWORD=$POSTGRES_PASSWORD pg_dump -h "$POSTGRES_HOST" -p "$POSTGRES_PORT" -U "$POSTGRES_USER" "$POSTGRES_DB" | gzip > "$FILE"
echo "Gerado: $FILE"
```

Restauração:

```bash
gunzip -c backup-*.sql.gz | PGPASSWORD=$POSTGRES_PASSWORD psql -h "$POSTGRES_HOST" -p "$POSTGRES_PORT" -U "$POSTGRES_USER" -d "$POSTGRES_DB"
```

---

# 17) Observabilidade

* **Logs**: agregue com CloudWatch/ELK/Loki.
* **Healthchecks**: `/health` no backend; ping do Nginx.
* **Métricas**: `prometheus_flask_exporter` (expor `/metrics`).
* **Tracing**: OpenTelemetry + Jaeger/Tempo (opcional).

---

# 18) Segurança — checklist rápido

* TLS obrigatório (HSTS no Nginx).
* CORS restrito ao domínio do app.
* Rate limit (`flask-limiter`) em rotas sensíveis.
* Validação/sanitização de input (pydantic/marshmallow).
* Autenticação/JWT; rotação/expiração de tokens.
* Segredos em cofre (Secrets Manager/Vault); nunca em VCS.
* Imagens base atualizadas e scan de vulnerabilidades.

---

# 19) Makefile (qualidade de vida)

`Makefile` na raiz:

```makefile
.PHONY: up down logs be fe migrate
up:
	docker compose up -d

down:
	docker compose down -v

logs:
	docker compose logs -f --tail=100

be:
	docker compose exec backend bash || docker compose exec backend sh

fe:
	docker compose exec frontend sh

migrate:
	docker compose exec backend flask db migrate -m "auto" && docker compose exec backend flask db upgrade
```

---

# 20) Próximos incrementos recomendados

* **Auth**: JWT + refresh; ou OAuth2 (Keycloak/Auth0).
* **Camadas**: Services/Repositories.
* **Testes**: contratos (schemathesis), e2e do frontend (Playwright).
* **Cache**: Redis (sessões/fila/rate limit).
* **Fila**: Celery/RQ para jobs assíncronos.
* **CD**: Actions + registry + deploy automatizado.
