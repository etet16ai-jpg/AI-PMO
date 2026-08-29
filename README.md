# AI-ERP with n8n — Graduation Project

A small, no-code ERP for a fictional Israeli electronics business, built on
n8n + Airtable + an AI layer (agents + RAG), with a web admin UI in
Lovable or Base44. Full spec: [`final-project-spec.docx`](./final-project-spec.docx).

## Three layers

| Layer   | Where            | Role                                                            |
|---------|------------------|----------------------------------------------------------------|
| Data    | Airtable (cloud) | 4 tables in use (of a 14-table model) — source of truth        |
| Logic   | n8n (this repo)  | 10 workflows + 3 AI agents + a vector store (RAG)              |
| UI      | Lovable / Base44 | admin app: tables, forms, dashboard, chat with the agent      |

## This repo runs the Logic layer

A **separate, isolated** n8n instance — it shares nothing with the production
n8n on this host except the machine and the Caddy proxy:

- own container `n8n-final`, own DB + encryption key (volume `n8n_final_data`)
- own compose project (`final-project`) and network
- hard memory / CPU caps (`docker-compose.yml`) so it can't starve production

## Run it

```bash
cp .env.example .env
# set N8N_ENCRYPTION_KEY (openssl rand -hex 32) and N8N_HOST in .env

docker compose up -d
docker compose logs -f n8n-final
```

Direct access for setup: <http://127.0.0.1:5679> (localhost only).

Public access: add the block in [`Caddyfile.snippet`](./Caddyfile.snippet) to
the host Caddyfile + a DNS record, then
`docker exec caddy caddy reload --config /etc/caddy/Caddyfile`.

## Workflows

Exported JSON lives in [`workflows/`](./workflows/). Export / import:

```bash
docker exec n8n-final n8n export:workflow --all --output=/tmp/wf.json
docker cp n8n-final:/tmp/wf.json ./workflows/
```

n8n workflow exports reference credentials by name/ID only — no secrets — so
they are safe to commit. Real secrets always stay in n8n's credential store
and in the gitignored `.env`, never in this repo.

## Layout

```
docker-compose.yml       the isolated n8n-final instance + resource caps
.env.example             template; copy to .env (gitignored) and fill in
Caddyfile.snippet        reverse-proxy block for the host Caddy
workflows/               exported workflow JSON
docs/                    architecture notes, Airtable schema, VAT rules
final-project-spec.docx  the course brief
```
