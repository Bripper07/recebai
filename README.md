# 📦 RecebAI — Plataforma de Recebimento Inteligente

> Projeto Integrador — Introdução à IA | PUC-Rio  
> **Equipe:** Bruno, Pedro Henrique, Marcelo, Isabel, João Henrique

---

## O que é o RecebAI?

O RecebAI é uma plataforma de gestão de encomendas para condomínios. Quando uma encomenda chega na portaria, o sistema registra os dados, notifica o morador automaticamente via WhatsApp com uma mensagem personalizada por IA, e permite que o morador consulte suas encomendas conversando diretamente com um agente de IA pelo WhatsApp.

**Site:** https://bripper07.github.io/recebai

---

## Funcionalidades

- **Registro de encomendas** pelo porteiro via painel web
- **Notificação automática** via WhatsApp com mensagem personalizada por IA ao receber
- **Agente conversacional** no WhatsApp — o morador pergunta sobre suas encomendas e o agente responde com memória de conversa
- **Reenvio automático** de lembrete após 3 dias de encomenda parada
- **Relatório do gestor** com métricas, gráficos e alertas de encomendas paradas
- **Envio de relatório por email** com conteúdo gerado por IA
- **Site responsivo** — funciona em celular e computador
- **Cadastro de moradores** com nome, CPF, celular e unidade

---

## Stack Tecnológico

| Componente | Tecnologia |
|---|---|
| Frontend | GitHub Pages (HTML/CSS/JS) |
| Backend | n8n Cloud (automação via workflows) |
| Banco de dados | Google Sheets |
| WhatsApp | Evolution API 2.3.7 (Railway) |
| IA | OpenAI GPT-4o mini |
| Memória do agente | n8n Simple Memory |
| Email | Gmail OAuth2 via n8n |

---

## Arquitetura

```
Porteiro (web) → n8n Workflow 3 → Google Sheets
                              ↓
                      OpenAI (gera mensagem)
                              ↓
                      Evolution API → WhatsApp (morador)

Morador (WhatsApp) → Evolution API → n8n Workflow 8 (Agente)
                                           ↓
                                    Google Sheets (busca encomendas)
                                           ↓
                                    OpenAI + Simple Memory
                                           ↓
                                    Evolution API → WhatsApp (resposta)

Schedule (todo dia 9h) → n8n Workflow 9 → Google Sheets
                                               ↓
                                     Filtra encomendas > 3 dias
                                               ↓
                                     Evolution API → WhatsApp (lembrete)
```

---

## Workflows n8n (10 no total)

| # | Nome | Função |
|---|---|---|
| 1 | Serve Frontend | Servia o HTML (desativado) |
| 2 | Listar Encomendas | GET `/recebai/encomendas` |
| 3 | Registrar Encomenda | POST `/recebai/encomendas` + notificação WhatsApp |
| 4 | Confirmar Retirada | POST `/recebai/encomendas/retirar` |
| 5 | Stats | GET `/recebai/stats` |
| 6 | Cadastro Moradores | POST `/recebai/moradores` |
| 7 | Listar Moradores | GET `/recebai/moradores` |
| 8 | Agente Conversacional | Webhook WhatsApp + IA com memória |
| 9 | Reenvio de Notificação | Schedule diário às 9h |
| 10 | Enviar Relatório Email | POST `/recebai/relatorio/email` |

---

## Google Sheets

**ID:** `1S0PaEIVJ7fbjzipFDXFoFrtmrxxuem1IA3VMk7BL73c`

**Aba `RecebAI - Encomendas`:**
```
id | tracking_code | destinatario_nome | destinatario_unidade | transportadora | status | created_at | retirada_at | retirada_por | destinatario_telefone
```

**Aba `RecebAI - Moradores`:**
```
id | nome | cpf | celular | unidade
```

---

## Como rodar localmente

O frontend é um único arquivo `index.html` — basta abrir no browser. Não há dependências ou build.

Para apontar para sua própria instância de n8n, troque a constante `API_BASE` no topo do script:

```javascript
const API_BASE = 'https://SEU_N8N.app.n8n.cloud/webhook';
```

---

## Infraestrutura

| Serviço | URL |
|---|---|
| Frontend | https://bripper07.github.io/recebai |
| n8n Cloud | https://brunin00.app.n8n.cloud |
| Evolution API | https://evolution-api-production-2cb9.up.railway.app |
| Instância WhatsApp | RecebAI (+55 67 98476 0343) |

---

## Observações importantes para reprodução

- Números de telefone devem ser salvos **com o prefixo 55** (ex: `5521999999999`)
- O filtro de busca no Google Sheets deve estar em **modo Expression**, não Fixed
- O nó Append to Google Sheets deve referenciar `$('Prepare Data').first().json.campo` e não `$json.campo`
- A Evolution API 2.3.7 usa `{ "number": "...", "text": "..." }` para envio de mensagens
- O webhook da Evolution API é configurado via POST em `/webhook/set/RecebAI`

---
