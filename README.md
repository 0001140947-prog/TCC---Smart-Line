# TSEA Smart Line - Versão Produção

Backend e frontend **separados**, com painel de login/registro, perfis (admin → gestor → operador → usuário), ordem de serviço auto-incrementada (OS-00001) e limite de tolerância configurável por operador.

## Estrutura

- **backend/** – API Flask (porta 5000): autenticação JWT, usuários, registros, export, portal externo.
- **frontend/** – Interface estática (HTML/JS): login, registro, dashboard, admin. Servir em outra porta (ex.: 8080).

## Como rodar

### 1. Backend (API)

```bash
cd Smart-Line
pip install -r backend/requirements.txt
python -m backend.app
```

- API: `http://0.0.0.0:5000`
- Na primeira execução é criado o usuário **admin**: `admin@tsea.local` / `Admin@123`
- Banco SQLite: `backend/database.db`

### 2. Frontend

```bash
cd Smart-Line
python -m http.server 8080 --directory frontend
```

- Acesse: `http://localhost:8080`
- Em `frontend/js/config.js` a URL da API é `http://localhost:5000`. Em produção, altere para a URL do backend.

### 3. Portal externo (hardware/sensor)

- **URL:** `POST http://<servidor>:5000/api/registro/portal`
- **Header:** `X-API-Key: portal-tsea-2024` (ou defina `SMARTLINE_PORTAL_API_KEY` no ambiente)
- **Body (JSON):** `dimensao_capturada`, opcional: `ordem_producao`, `limite_tolerancia`
- A ordem de serviço (OS-00001, OS-00002, …) é gerada automaticamente.

## Segurança

- Senhas com hash (werkzeug).
- JWT com expiração para acesso às rotas protegidas.
- Roles: **admin** (tudo + usuários), **gestor** (export, limpar), **operador** (registrar, ver), **usuario** (só ver).
- Portal externo usa API key no header (sem JWT).
- Em produção: defina `SMARTLINE_SECRET_KEY` e `SMARTLINE_CORS_ORIGINS` (origens do frontend).

## Limite por operador

- Cada usuário pode definir um **limite padrão** em “Meu limite” no dashboard.
- Ao abrir “Registrar Inspeção”, o campo limite vem preenchido com esse padrão e pode ser alterado a cada registro.

## Deploy

- Firewall: `sudo ufw allow 5000/tcp` (e 8080 se servir o frontend no mesmo host).
- Produção: use HTTPS (reverse proxy) e variáveis de ambiente para `SECRET_KEY`, `CORS_ORIGINS` e `PORTAL_API_KEY`.
