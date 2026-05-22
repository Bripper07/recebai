# RecebAI — Gestão de Encomendas para Condomínios

Plataforma para gerenciar encomendas recebidas na portaria de condomínios, com notificação automática via WhatsApp para os moradores.

Projeto Integrador — PUC Rio, Introdução à IA.

**Demo:** https://bripper07.github.io/recebai

---

## O que o sistema faz

- Portaria registra encomenda recebida
- Morador recebe notificação automática no WhatsApp
- Morador vai à portaria e confirma retirada
- Dashboard mostra estatísticas em tempo real
- Cadastro de moradores com preenchimento automático de telefone na hora do registro

---

## Tecnologias

- **Frontend:** HTML/CSS/JS puro, hospedado no GitHub Pages
- **Backend:** n8n Cloud (sem código próprio, só workflows)
- **Banco de dados:** Google Sheets
- **Notificações:** Evolution API + WhatsApp (via Baileys)
- **Hospedagem da Evolution API:** Railway

---

## Estrutura do repositório

```
recebai/
├── index.html              # Frontend completo (single page app)
└── workflows/
    ├── RecebAI - 1. Serve Frontend.json
    ├── RecebAI - 2. Listar Encomendas.json
    ├── RecebAI - 3. Registrar Encomenda.json
    ├── RecebAI - 4. Confirmar Retirada.json
    ├── RecebAI - 5. Stats.json
    ├── RecebAI - 6. Cadastro Moradores.json
    └── RecebAI - 7. Listar Moradores.json
```

---

## Como configurar do zero

### 1. Google Sheets

Crie uma planilha e adicione duas abas:

**Aba 1 — `RecebAI - Encomendas`**
Colunas na linha 1:
```
id | tracking_code | destinatario_nome | destinatario_unidade | transportadora | status | created_at | retirada_at | retirada_por | destinatario_telefone
```

**Aba 2 — `RecebAI - Moradores`**
Colunas na linha 1:
```
id | nome | cpf | celular | unidade
```

Anote o ID da planilha (parte da URL entre `/d/` e `/edit`).

---

### 2. n8n Cloud

1. Crie uma conta em [n8n.io](https://n8n.io)
2. Conecte sua conta do Google Sheets via OAuth (Settings > Credentials)
3. Importe os 7 workflows da pasta `/workflows` um por um (menu > Import from file)
4. Em cada workflow que usa o Google Sheets, atualize o campo **Document** para apontar pra sua planilha
5. Ative todos os workflows (botão Inactive > Active)

Anote a URL base dos seus webhooks (ex: `https://seuusuario.app.n8n.cloud/webhook`).

---

### 3. Evolution API (WhatsApp)

1. Crie uma conta no [Railway](https://railway.app)
2. Faça deploy do template **"Evolution API Whatsapp using n8n"**
3. Acesse o manager da Evolution API (`sua-url.railway.app/manager`)
4. Crie uma instância chamada `RecebAI`
5. Conecte um número de WhatsApp escaneando o QR Code
6. Anote a URL base e a API Key global

> O número conectado será o remetente das notificações. Pode ser qualquer número WhatsApp, inclusive pessoal.

---

### 4. Frontend

No arquivo `index.html`, na linha do `API_BASE`, substitua pela URL dos seus webhooks:

```javascript
const API_BASE = 'https://seuusuario.app.n8n.cloud/webhook';
```

Hospede o `index.html` no GitHub Pages:
- Repositório > Settings > Pages > Source: main branch, pasta raiz

---

### 5. Workflow 3 — Notificação WhatsApp

No nó **HTTP Request** do workflow de Registrar Encomenda, configure:

```
Method: POST
URL: https://sua-evolution-api.railway.app/message/sendText/RecebAI
Header: apikey: SUA_API_KEY
Body: {
  "number": "{{ $('Prepare Data').first().json.destinatario_telefone }}",
  "text": "Olá {{ $('Prepare Data').first().json.destinatario_nome }}! Sua encomenda ({{ $('Prepare Data').first().json.transportadora }} - {{ $('Prepare Data').first().json.tracking_code }}) chegou na portaria. Por favor, retire no horário disponível. 📦"
}
```

> O número precisa ter o código do país. O sistema já adiciona `55` automaticamente para números brasileiros.

---

## Fluxo completo

```
Portaria preenche formulário
        ↓
Webhook POST /recebai/encomendas
        ↓
Salva no Google Sheets
        ↓
Envia WhatsApp pro morador
        ↓
Morador vai à portaria
        ↓
Portaria clica em "Confirmar Retirada"
        ↓
Webhook POST /recebai/encomendas/retirar
        ↓
Atualiza status no Google Sheets
```

---

## Credenciais do projeto (demo)

> Não reutilize em produção.

| Serviço | Valor |
|---|---|
| n8n Cloud | https://bripper.app.n8n.cloud |
| Evolution API | https://evolution-api-production-2cb9.up.railway.app |
| Instância WhatsApp | RecebAI |
| Google Sheets ID | 1S0PaEIVJ7fbjzipFDXFoFrtmrxxuem1IA3VMk7BL73c |
