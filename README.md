# ClearIT Copilot — Backend (Deploy)

> **Aviso:** A instância em `clearit-copilot.onrender.com` é temporária, usada apenas para testes e avaliação da banca do programa Pulse Mais 2026. Será desativada em breve.

Backend do ClearIT Copilot em produção. Recebe requisições do widget FreshService e da POC, processa com busca semântica + Gemini e retorna diagnósticos.

**Squad Sherlock (B5) — Pulse Mais 2026**

---

## Como funciona

Este é o servidor que roda no Render (ou qualquer hosting Node.js). Ele expõe a API REST que os frontends consomem:

- `/api/analyze` — Diagnóstico RAG (embedding + busca + geração)
- `/api/chat` — Chat follow-up contextualizado
- `/api/feedback` — Registra avaliação do analista
- `/api/web-search` — Busca web com resumo + análise de compatibilidade
- `/api/feedbacks` — Lista feedbacks com estatísticas
- `/api/tickets` — Lista chamados ativos

---

## Pré-requisitos

- Node.js 18+
- Chave de API do Gemini ([Google AI Studio](https://aistudio.google.com/apikey))

---

## Instalação local

```bash
git clone <url-do-repo>
cd clearit-copilot-deploy
npm install
```

Crie o arquivo `.env` baseado no `.env.example`:

```bash
cp .env.example .env
```

Edite o `.env`:

```env
GEMINI_API_KEY=AIzaSy...sua_chave_aqui
FRESHSERVICE_DOMAIN=seu_subdominio
FRESHSERVICE_API_KEY=sua_api_key_freshservice
PORT=3000
```

| Variável | Obrigatória | Descrição |
|----------|-------------|-----------|
| `GEMINI_API_KEY` | Sim | Chave do Google AI Studio. Sem ela, cai pro fallback por tokens |
| `FRESHSERVICE_DOMAIN` | Não | Subdomínio do FreshService (ex: `suaempresa` de `suaempresa.freshservice.com`) |
| `FRESHSERVICE_API_KEY` | Não | API key do FreshService (Profile Settings). Permite buscar tickets reais |
| `PORT` | Não | Porta do servidor (default: 3000) |

---

## Executar

```bash
node server.js
```

---

## Deploy no Render

1. Crie uma conta no [Render](https://render.com)
2. New → Web Service → conecte o repositório GitHub
3. Configurações:
   - **Build Command:** `npm install`
   - **Start Command:** `node server.js`
4. **Variáveis de ambiente** (configurar no painel do Render em Environment):

   | Variável | Obrigatória | Valor |
   |----------|-------------|-------|
   | `GEMINI_API_KEY` | Sim | Sua chave do Google AI Studio |
   | `FRESHSERVICE_DOMAIN` | Não | Subdomínio (ex: `suaempresa`) |
   | `FRESHSERVICE_API_KEY` | Não | API key do FreshService |
   | `PORT` | Não | O Render define automaticamente |

   > As variáveis são configuradas **no painel do Render**, não em arquivo. Nunca commite credenciais.

5. Deploy automático a cada push na branch main

### Manter ativo (free tier)

O free tier dorme após 15 min de inatividade. Para manter acordado:
- Use [cron-job.org](https://cron-job.org) (grátis)
- Configure um GET para `https://sua-url.onrender.com/api/feedbacks` a cada 14 minutos

---

## Estrutura

```
clearit-copilot-deploy/
├── server.js              # API Express (endpoints + lógica RAG)
├── services/
│   ├── masking.js         # Pipeline DLP/LGPD (14 regras)
│   └── rag.js             # Embeddings, busca semântica, fallback por tokens
├── public/
│   ├── index.html         # Frontend (serve como página de teste)
│   ├── app.js
│   ├── styles.css
│   ├── manual.html        # Manual do sistema (acessado pelo widget via URL)
│   ├── fluxo-diagnostico.svg    # Diagrama: fluxo do analista
│   ├── arquitetura-atual.svg    # Diagrama: arquitetura em produção
│   └── arquitetura-futura.svg   # Diagrama: arquitetura planejada
├── data/
│   ├── mock-tickets.json  # Base vetorial (40 tickets + 13 KBs)
│   └── active-tickets.json
├── .env.example           # Template de variáveis
├── .gitignore
└── package.json
```

---

## Sobre o manual.html

O manual do sistema (`public/manual.html`) está hospedado neste backend porque é a única URL pública HTTPS acessível pelo widget do FreshService. Quando o analista clica "📖 Manual" no widget, abre `backend_url/manual.html` em nova aba.

Os diagramas SVG (`fluxo-diagnostico.svg`, `arquitetura-atual.svg`, `arquitetura-futura.svg`) também estão aqui pelo mesmo motivo — são referenciados pelo manual e precisam ser servidos via HTTPS para renderizar corretamente no browser.

---

## Atualizar

Para aplicar mudanças da POC neste deploy:

1. Copie os arquivos atualizados:
   - `server.js`
   - `services/masking.js`
   - `services/rag.js`
   - `data/mock-tickets.json` (se a base mudar)

2. Commit e push — o Render faz deploy automático

Não copie: `.env`, `data/embeddings-cache.json`, `data/feedbacks.json` (gerados em runtime).

---

## Repositórios Relacionados

| Repo | Descrição |
|------|-----------|
| [sherlock-clearit-copilot](https://github.com/beandy-dev/sherlock-clearit-copilot) | POC completa (frontend + backend + docs) |
| [clearit-copilot-freshservice-api](https://github.com/beandy-dev/clearit-copilot-freshservice-api) | Widget FreshService (Custom App) |

📖 **Manual do Sistema:** [https://clearit-copilot.onrender.com/manual.html](https://clearit-copilot.onrender.com/manual.html)

---

## Equipe

| Membro | Frente |
|--------|--------|
| [Beatriz Andrade Lourenço](https://linkedin.com/in/anbeaty) | Tecnologia, Produto e Negócios |
| [Davi da Paz Mota](https://linkedin.com/in/davidapaz05) | Tecnologia e Produto |
| [Maria Eduarda Ferreira Santos](https://linkedin.com/in/dudazfd) | Tecnologia e Produto |
| [Maria Eloisa Gomes da Conceição](https://linkedin.com/in/mariaeloisa69) | Negócios e Estratégia |
| [Phelipe Alexandre de Almeida](https://linkedin.com/in/phelipe134) | Tecnologia e Produto |
