Uma **API de pagamento** é uma interface que permite que sistemas (como um site ou app de e-commerce) se comuniquem com provedores de serviços financeiros para processar transações — cartão, Pix, boleto etc. — de forma segura e automatizada.

---

## **1. O que é uma API de pagamento**

* É um conjunto de **endpoints** e **protocolos** que padronizam como seu sistema envia dados de pagamento, recebe confirmações e lida com erros.
* Garante **segurança** (criptografia, tokenização, PCI Compliance), **confiabilidade** (confirmação de transações) e **integração simplificada** (não precisa lidar diretamente com adquirentes).
* Exemplo prático: quando o cliente clica em *Finalizar Compra*, seu sistema envia os dados para a API, que processa o pagamento e retorna a resposta (aprovado, recusado, pendente).

---

## **2. Modelos de APIs de pagamento no mercado**

| Modelo                             | Como funciona                                                                                          | Exemplos                                      |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------ | --------------------------------------------- |
| **Gateway de pagamento**           | Intermedia entre o e-commerce e os adquirentes, enviando transações para múltiplos meios de pagamento. | Vindi, PagSeguro, Stripe, PayPal, Adyen       |
| **API de adquirente**              | Conexão direta com a operadora que processa cartões.                                                   | Cielo API, Rede API, Getnet API               |
| **API de carteira digital**        | Pagamento via saldo ou cartão vinculado a uma conta digital.                                           | PayPal, Mercado Pago, PicPay                  |
| **API de pagamentos instantâneos** | Foco em transferências rápidas como Pix.                                                               | API Pix do Banco Central, Gerencianet Pix API |
| **API de agregador/marketplace**   | Permite múltiplos vendedores receberem pagamentos em um mesmo checkout.                                | Pagar.me, Stripe Connect, Mercado Pago Split  |

---

## **3. Integração em um projeto Fullstack de E-commerce**

### **Fluxo básico**

1. **Frontend (React, Vue, Angular)**

   * Captura os dados do pagamento via formulário seguro ou checkout da API.
   * Se possível, use **SDK oficial** ou **Checkout Hosted** para evitar manipular dados sensíveis diretamente.

2. **Backend (Node.js, Python, Java, etc.)**

   * Recebe os dados (ou token) do frontend.
   * Faz a requisição HTTPS para a API de pagamento usando **chave privada**.
   * Garante segurança com autenticação e criptografia.
   * Armazena status da transação no banco de dados.

3. **Banco de Dados (MySQL, PostgreSQL, MongoDB)**

   * Guarda registros da compra, status de pagamento e histórico para controle e auditoria.

4. **Webhooks (notificações automáticas)**

   * A API de pagamento envia para seu backend notificações sobre mudança no status da transação (ex.: boleto pago, Pix recebido).

---

### **Exemplo prático: fluxo usando uma API de pagamento (Stripe)**

```javascript
// Backend Node.js (Express)
import express from 'express';
import Stripe from 'stripe';

const app = express();
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

app.post('/checkout', async (req, res) => {
  const { amount, currency, payment_method } = req.body;

  try {
    const paymentIntent = await stripe.paymentIntents.create({
      amount,
      currency,
      payment_method,
      confirm: true
    });

    res.json(paymentIntent);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

app.listen(3000);
```

Nesse caso:

* **Frontend** obtém o token de pagamento via SDK do Stripe.
* **Backend** confirma o pagamento via API.
* **Webhook** atualiza status automaticamente.

---

## **4. Boas práticas**

* **Nunca** trafegar dados de cartão puro pelo seu servidor se não for PCI DSS.
* Usar **HTTPS obrigatório** em todas as requisições.
* Sempre utilizar **ambiente de sandbox** para testes.
* Implementar **tratamento de erros e retentativas**.
* Usar **tokens** para representar métodos de pagamento, nunca armazenar dados sensíveis.

---

Abaixo está um **guia direto e prático** para integrar **cartão, Pix e boleto** em um e-commerce **React + Node.js (Express) + Webhooks**. O modelo é **agnóstico de provedor** (funciona para Vindi, Stripe, Pagar.me, Mercado Pago etc.) — troque apenas a implementação dos *adapters*.

---

# 1) Arquitetura mínima

* **Frontend (React)**: coleta dados e recebe *tokens/URLs* de pagamento do backend.
* **Backend (Node/Express)**: cria/autoriza transações via **SDK/HTTP** do provedor.
* **Webhooks**: recebe notificações assíncronas (pago, recusado, estornado).
* **DB** (PostgreSQL/MySQL): pedidos, pagamentos, logs de webhook.

```
Cliente → Frontend → Backend → Provedor de Pagamento
                 ↑      ↓
             Webhooks ←──
```

---

# 2) Banco de dados (SQL simples)

```sql
CREATE TABLE orders (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL,
  amount_cents INT NOT NULL,
  currency VARCHAR(3) NOT NULL DEFAULT 'BRL',
  status VARCHAR(20) NOT NULL, -- created, pending, paid, failed, refunded
  created_at TIMESTAMP DEFAULT now()
);

CREATE TABLE payments (
  id UUID PRIMARY KEY,
  order_id UUID REFERENCES orders(id),
  provider VARCHAR(20),              -- vindi|stripe|pagarme|mp
  method VARCHAR(20),                -- card|pix|boleto
  provider_payment_id TEXT,          -- id/intent/charge
  status VARCHAR(20),                -- pending|paid|failed
  qr_code TEXT,                      -- Pix (opcional)
  boleto_url TEXT,                   -- Boleto (opcional)
  raw_response JSONB,
  created_at TIMESTAMP DEFAULT now()
);
```

---

# 3) Backend (Node + Express)

## 3.1 Setup e variáveis

```bash
npm i express cors body-parser dotenv
# e o SDK do seu provedor (ex.: stripe, pagarme, mercadopago, vindi via HTTP)
```

`.env`

```
APP_URL=http://localhost:5173
WEBHOOK_SECRET=troque-aqui
PROVIDER=stripe            # ou vindi/pagarme/mercadopago
STRIPE_SECRET_KEY=sk_live_...
# Para outros provedores, adicione suas chaves aqui
```

`server.js`

```js
import express from 'express';
import cors from 'cors';
import bodyParser from 'body-parser';
import 'dotenv/config';
import { createCardPayment, createPixPayment, createBoletoPayment } from './src/payments.js';
import { handleWebhook } from './src/webhooks.js';

const app = express();
app.use(cors({ origin: process.env.APP_URL }));
app.use(bodyParser.json());

// Checkout (cartão)
app.post('/api/checkout/card', async (req, res) => {
  try {
    const { orderId, amountCents, currency, paymentToken } = req.body;
    const out = await createCardPayment({ orderId, amountCents, currency, paymentToken });
    res.json(out); // { paymentId, status, ... }
  } catch (e) { res.status(400).json({ error: e.message }); }
});

// Checkout (Pix)
app.post('/api/checkout/pix', async (req, res) => {
  try {
    const { orderId, amountCents, currency } = req.body;
    const out = await createPixPayment({ orderId, amountCents, currency });
    res.json(out); // { paymentId, status, qrCode, qrCodeUrl }
  } catch (e) { res.status(400).json({ error: e.message }); }
});

// Checkout (Boleto)
app.post('/api/checkout/boleto', async (req, res) => {
  try {
    const { orderId, amountCents, currency, customer } = req.body; // nome, cpf, endereço
    const out = await createBoletoPayment({ orderId, amountCents, currency, customer });
    res.json(out); // { paymentId, status, boletoUrl, barcode }
  } catch (e) { res.status(400).json({ error: e.message }); }
});

// Webhook (um endpoint para todos os eventos)
app.post('/api/webhooks/provider', express.raw({ type: '*/*' }), (req, res) => {
  try {
    handleWebhook(req); // valida assinatura e atualiza DB
    res.sendStatus(200);
  } catch (e) { res.sendStatus(400); }
});

app.listen(3001, () => console.log('API on :3001'));
```

> **Dica segurança:** mantenha `/api/webhooks/provider` com **validação de assinatura** do provedor e IP allowlist quando possível.

## 3.2 Adapters por método (exemplos concisos)

`src/payments.js` (esqueleto com troca por provedor)

```js
import 'dotenv/config';
// Exemplos: Stripe para cartão, provedor Pix, provedor boleto.
// Substitua a implementação conforme seu PROVEDOR.

export async function createCardPayment({ orderId, amountCents, currency, paymentToken }) {
  // Exemplo c/ Stripe (Payment Intents + token do frontend)
  const Stripe = (await import('stripe')).default;
  const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);
  const intent = await stripe.paymentIntents.create({
    amount: amountCents,
    currency: currency.toLowerCase(),
    payment_method: paymentToken,    // recebido do frontend
    confirm: true,
    metadata: { orderId }
  });
  return {
    paymentId: intent.id,
    status: intent.status,           // 'succeeded' | 'requires_action' ...
  };
}

export async function createPixPayment({ orderId, amountCents, currency }) {
  // Fluxo genérico: crie cobrança Pix e retorne brcode/url
  // Ex.: Mercado Pago/Pagar.me/Vindi: crie "charge" PIX e pegue QR
  // Abaixo: pseudo-resposta (trocar por SDK/HTTP real do seu provedor)
  const response = {
    id: 'pix_123',
    status: 'pending',
    qr_code: '00020126430014BR.GOV.BCB.PIX...6304ABCD',
    qr_code_url: 'https://example.com/qrcode.png'
  };
  return {
    paymentId: response.id,
    status: response.status,
    qrCode: response.qr_code,
    qrCodeUrl: response.qr_code_url
  };
}

export async function createBoletoPayment({ orderId, amountCents, currency, customer }) {
  // Fluxo genérico: crie boleto (due_date, customer) e retorne URL/código
  const response = {
    id: 'bol_456',
    status: 'pending',
    boleto_url: 'https://example.com/boleto.pdf',
    barcode: '23793.38127 60000.000000 12345.678901 1 91230000010000'
  };
  return {
    paymentId: response.id,
    status: response.status,
    boletoUrl: response.boleto_url,
    barcode: response.barcode
  };
}
```

`src/webhooks.js` (validação + atualização)

```js
import crypto from 'crypto';
// Troque a validação conforme o provedor (ex.: assinatura do Stripe/MP/Vindi)

export function handleWebhook(req) {
  const signature = req.headers['x-signature'] || '';
  const body = req.body; // raw buffer (configure express.raw)

  // Exemplo: verificação HMAC simples (padrão fictício)
  const expected = crypto
    .createHmac('sha256', process.env.WEBHOOK_SECRET)
    .update(body)
    .digest('hex');

  if (signature !== expected) throw new Error('Invalid signature');

  const event = JSON.parse(body.toString());
  // Atualize pagamento/pedido conforme event.type/status
  // ex: payment.paid → set orders.status=paid
}
```

---

# 4) Frontend (React)

## 4.1 Cartão (ex.: Stripe Elements — tokenização no browser)

```bash
npm i @stripe/stripe-js @stripe/react-stripe-js
```

```jsx
import {loadStripe} from '@stripe/stripe-js';
import {Elements, CardElement, useStripe, useElements} from '@stripe/react-stripe-js';

const stripePromise = loadStripe(import.meta.env.VITE_STRIPE_PK);

function CardCheckout() {
  const stripe = useStripe();
  const elements = useElements();

  const pay = async () => {
    const {paymentMethod, error} = await stripe.createPaymentMethod({
      type: 'card',
      card: elements.getElement(CardElement)
    });
    if (error) return alert(error.message);

    const r = await fetch('/api/checkout/card', {
      method:'POST',
      headers:{'Content-Type':'application/json'},
      body: JSON.stringify({
        orderId: crypto.randomUUID(),
        amountCents: 10000,
        currency: 'BRL',
        paymentToken: paymentMethod.id
      })
    });
    const data = await r.json();
    alert(`Status: ${data.status}`);
  };

  return (
    <div>
      <CardElement />
      <button onClick={pay}>Pagar</button>
    </div>
  );
}

export default function Page(){
  return <Elements stripe={stripePromise}><CardCheckout/></Elements>;
}
```

> **Por quê assim?** O cartão é tokenizado no **frontend** (PCI reduzido). O **backend** confirma o pagamento com a **chave secreta**.

## 4.2 Pix (mostrar QR Code / copia e cola)

```jsx
import { useState } from 'react';

export function PixCheckout(){
  const [qr, setQr] = useState(null);

  const pay = async () => {
    const r = await fetch('/api/checkout/pix', {
      method:'POST',
      headers:{'Content-Type':'application/json'},
      body: JSON.stringify({
        orderId: crypto.randomUUID(),
        amountCents: 10000,
        currency: 'BRL'
      })
    });
    const data = await r.json();
    setQr(data);
  };

  return (
    <div>
      <button onClick={pay}>Gerar Pix</button>
      {qr && (
        <div>
          <img src={qr.qrCodeUrl} alt="QR Pix" />
          <textarea readOnly value={qr.qrCode} />
          <p>Status: {qr.status}</p>
        </div>
      )}
    </div>
  );
}
```

## 4.3 Boleto (link/linha digitável)

```jsx
import { useState } from 'react';

export function BoletoCheckout(){
  const [boleto, setBoleto] = useState(null);

  const pay = async () => {
    const r = await fetch('/api/checkout/boleto', {
      method:'POST',
      headers:{'Content-Type':'application/json'},
      body: JSON.stringify({
        orderId: crypto.randomUUID(),
        amountCents: 10000,
        currency: 'BRL',
        customer: { name:'Cliente Teste', cpf:'00000000000' }
      })
    });
    setBoleto(await r.json());
  };

  return (
    <div>
      <button onClick={pay}>Gerar Boleto</button>
      {boleto && (
        <div>
          <a href={boleto.boletoUrl} target="_blank">Ver Boleto</a>
          <p>Linha Digitável: {boleto.barcode}</p>
          <p>Status: {boleto.status}</p>
        </div>
      )}
    </div>
  );
}
```

---

# 5) Webhooks: estados típicos

* **Cartão**: `requires_action` → 3DS no frontend; `succeeded` → marcar pedido como **paid**.
* **Pix**: `pending` ao criar QR; `paid` quando o PSP confirma → atualize pedido.
* **Boleto**: `pending` ao emitir; `paid` na compensação (D+1/D+2); `expired` se não pago.

> Webhook deve ser **idempotente** (verifique se já processou aquele `event.id`).

---

# 6) Boas práticas (resumo)

* **HTTPS sempre** (frontend, backend e chamadas ao provedor).
* **Nunca armazene** dados de cartão; use **tokenização/checkout hospedado**.
* **Valide webhooks** (assinatura HMAC/secret do provedor).
* **Retry com backoff** para chamadas ao provedor e processamento de webhooks.
* **Logs/auditoria** de requisições e mudanças de status.
* **Feature flags** para alternar provedores sem mudar o fluxo.

---

# 7) Como plugar provedores específicos

* **Vindi**: use endpoints de *charges*/*transactions* para cartão, Pix e boleto; configure **webhooks** no painel e guarde `payment_profile` (token).
* **Stripe**: *Payment Intents* (cartão), *Pix* (no Brasil), *Checkout Session* opcional.
* **Pagar.me/Adiq/Mercado Pago**: possuem **SDKs** para cartão/Pix/boleto + *split* para marketplace.
  Basta substituir a lógica dos *adapters* no `payments.js` pelas chamadas do SDK/HTTP do seu provedor.

---


