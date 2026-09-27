# Valu·AI·tion — deploy

Dois pedaços pra subir: **o site** (GitHub Pages, estático, grátis) e **o backend de IA** (Cloudflare Workers, também tem plano grátis generoso).

## 1. Subir o site no GitHub Pages

```bash
# dentro da pasta valuaition-site/
git init
git add index.html README.md
git commit -m "primeira versão do Valu·AI·tion"
```

1. Crie um repositório novo no GitHub (pode ser público), ex: `valuaition`
2. `git remote add origin https://github.com/SEU-USUARIO/valuaition.git`
3. `git branch -M main && git push -u origin main`
4. No GitHub: **Settings → Pages → Source → branch `main`, pasta `/ (root)`** → Save
5. Em 1-2 minutos o site fica em `https://SEU-USUARIO.github.io/valuaition/`

Nesse ponto o site já funciona **sem IA nenhuma** — o parser por palavra-chave sozinho já lê e calcula os índices. O botão de fallback de IA vai aparecer, mas mostrar erro de "backend não configurado" até você fazer o passo 2.

## 2. Subir o backend de IA (Cloudflare Workers)

```bash
npm install -g wrangler
cd worker
wrangler login          # abre o navegador pra autenticar com sua conta Cloudflare (grátis)
wrangler secret put ANTHROPIC_API_KEY
# cole sua API key da Anthropic quando pedir (console.anthropic.com → API Keys)
wrangler deploy
```

Isso te devolve uma URL tipo `https://valuaition-worker.SEU-SUBDOMINIO.workers.dev` — é a `AI_ENDPOINT`.

## 3. Conectar o site ao backend

No `index.html`, ache a linha:

```js
const AI_ENDPOINT = 'https://SEU-WORKER.SEU-SUBDOMINIO.workers.dev';
```

Troque pela URL real que o `wrangler deploy` te deu, depois:

```bash
git add index.html
git commit -m "conecta backend de IA"
git push
```

## 4. Travar o CORS (importante antes de divulgar o site)

No `worker/worker.js`, troque:

```js
const ALLOWED_ORIGIN = '*';
```

por:

```js
const ALLOWED_ORIGIN = 'https://SEU-USUARIO.github.io';
```

e rode `wrangler deploy` de novo. Sem isso, qualquer site pode chamar seu Worker e gastar sua cota de API.

## 5. Limite de uso (recomendado antes de deixar público de verdade)

Sem isso, **você paga o token de todo mundo que usar o botão de IA** — não tem limite embutido ainda. Formas simples de adicionar depois:
- Cloudflare Rate Limiting (poucos cliques, direto no dashboard, sem código)
- Um contador simples em Cloudflare KV (ex: 5 chamadas por IP por dia)

Não é bloqueante pra testar sozinho, mas é obrigatório antes de mandar o link pra alguém.

## Onde pegar a API key da Anthropic
console.anthropic.com → **API Keys** → Create Key. Guarde um limite de gasto mensal lá também (Settings → Limits), como segunda camada de proteção.
