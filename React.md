#Aula 01
Aqui vai um “guia de professor”, direto ao ponto, para você dominar **React** no desenvolvimento de aplicações web modernas.

# O que é o React

Biblioteca JavaScript para criar **interfaces declarativas** com componentes reutilizáveis (UI = função do estado). Desde o React 18/19, o ecossistema evoluiu com **concorrência**, **Suspense**, **Actions** e **Server Components**, melhorando performance e DX (developer experience). ([legacy.reactjs.org][1], [react.dev][2])

# Fundamentos (base que nunca muda)

* **Componentes**: funções que retornam JSX. Evite classes em projetos novos (legado).
* **JSX**: açucar sintático para `React.createElement`.
* **Props**: dados de entrada (imutáveis).
* **State**: dados internos do componente (mutação via setters).
* **Renderização declarativa**: você descreve o estado final; o React reconcilia o DOM.

Exemplo simples:

```jsx
function Contador() {
  const [n, setN] = React.useState(0);
  return (
    <button onClick={() => setN(n + 1)}>
      Cliquei {n} vez(es)
    </button>
  );
}
```

# Hooks essenciais (90% dos casos)

* `useState` – estado local.
* `useEffect` – efeitos colaterais (fetch, timers).
* `useRef` – referência mutável sem re-render.
* `useMemo` / `useCallback` – memorizar cálculos e funções custosas.
* `useContext` – consumo de contexto (estado global leve).
* `useReducer` – lógica de estado complexa.
* **React 18**: `useTransition`, `useDeferredValue` para UX fluida. ([legacy.reactjs.org][1])

# Performance moderna (o que realmente importa)

* **Automatic batching** (React 18): múltiplas atualizações de estado no mesmo tick são agrupadas automaticamente → menos renders. ([legacy.reactjs.org][1])
* **Transitions**: diferenciam atualizações urgentes (input) de não urgentes (render pesado).
* **Suspense**: mostra fallback enquanto dados/recursos carregam (client e server). ([legacy.reactjs.org][1])
* **React Compiler (beta)**: otimizações automáticas de re-render e memos na build; acompanha o projeto com cuidado em apps grandes. ([react.dev][3])

# Formulários no React 19 (mais simples)

* **Actions**: funções assíncronas que lidam com submit, pendência, erro e *optimistic UI* sem boilerplate.
* **`useActionState` / `useFormStatus`**: estado do submit/loading/erro por formulário/ação.
* `<form action={minhaAction}>` passa a ser um padrão estável. ([react.dev][2], [freecodecamp.org][4], [Medium][5])

Exemplo enxuto:

```jsx
"use client";
import { useActionState } from "react";
async function salvar(formData) {
  // envia para API...
  return { ok: true };
}
export default function Perfil() {
  const [state, action, isPending] = useActionState(salvar, null);
  return (
    <form action={action}>
      <input name="nome" />
      <button disabled={isPending}>Salvar</button>
      {isPending && <p>Salvando...</p>}
      {state?.error && <p>Erro: {state.error}</p>}
    </form>
  );
}
```

# Server Components (RSC) e Server Actions

* **Server Components**: componentes que **rodam no servidor**, rendem antes do bundle do cliente e podem acessar dados/segredos sem enviar código extra ao navegador. Reduz JS no cliente e melhora TTFB. ([react.dev][6])
* **`'use server'`**: marca **Server Functions** invocáveis do cliente ou em `<form action={...}>`; atenção a serialização e segurança. ([react.dev][7])
* **Next.js** usa **Server Components por padrão** (App Router); você opta por Client Components quando precisa de interatividade imediata (event handlers, `useState`, etc.). ([nextjs.org][8])

# Estado global (quando o Context não basta)

* **Context** para temas, usuário, configurações.
* **Redux Toolkit**, **Zustand**, **Jotai** para estado complexo/compartilhado.
* Evite “overstate”: às vezes **colocar o estado mais perto do uso** (lifting mínimo) resolve.

# Data fetching (padrões práticos)

* **RSC + fetch no servidor** → menos JS no cliente e melhor cache.
* **Client**: bibliotecas como **TanStack Query** para cache/revalidação, fallback de Suspense e mutations.

# Roteamento

* **Next.js (App Router)**: filesystem routing, layouts aninhados, streaming + Suspense, RSC e Server Actions.
* **React Router**: ótimo para SPAs puras; desde v7 tem integração com RSC/bundlers compatíveis. ([reactrouter.com][9])

# Acessibilidade e UX

* Use **semântica** (landmarks, rótulos), foco visível, ARIA apenas quando necessário.
* Prefira **componentes controlados** com mensagens de erro claras e **teclabilidade**.

# Testes

* **React Testing Library** (testa comportamento do usuário) + **Vitest/Jest**.
* Teste hooks isolados (bibliotecas utilitárias) e fluxos principais.

# Build, tipos e tooling

* **Vite** para SPAs; **Next.js** para apps completas (SSR/SSG/RSC).
* **TypeScript**: tipar props/estado/retornos melhora refatoração e DX.
* **ESLint + Prettier** e *CI*; **Bundle Analyzer** para vigiar o tamanho.

# Segurança essencial

* **Nunca** exponha chaves no cliente; use RSC/Server Actions ou rotas API.
* Valide dados no servidor; sanitize HTML; proteja contra XSS/CSRF.
* Headers e *Content Security Policy* quando publicar.

# Boas práticas (checklist rápido)

* Componentes pequenos, focados e nomeados por intenção.
* Estado mínimo e **imutável**; derive o que puder de `props/state`.
* `key` estável em listas; evite index como `key` quando houver reordenação.
* Meça (Profiler, Lighthouse) antes de otimizar.

---

## Mapa de estudo sugerido

1. JSX, componentes, props/state → 2) hooks básicos → 3) rotas e dados → 4) Suspense/transitions → 5) formulários com Actions → 6) RSC/Server Actions em Next.js → 7) testes e performance → 8) deploy (Vercel) e observabilidade.

Se quiser, eu monto um **roteiro prático** com mini-projetos (SPA com Router, dashboard com TanStack Query, app Next.js com RSC/Actions) e entrego códigos-base para cada etapa.

[1]: https://legacy.reactjs.org/blog/2022/03/29/react-v18.html?utm_source=chatgpt.com "React v18.0 – React Blog"
[2]: https://react.dev/blog/2024/12/05/react-19?utm_source=chatgpt.com "React v19"
[3]: https://react.dev/blog/2024/10/21/react-compiler-beta-release?utm_source=chatgpt.com "React Compiler Beta Release"
[4]: https://www.freecodecamp.org/news/react-19-actions-simpliy-form-submission-and-loading-states/?utm_source=chatgpt.com "React 19 Actions – How to Simplify Form Submission and ..."
[5]: https://medium.com/%40ignatovich.dm/enhancing-form-handling-in-react-19-a-look-at-action-useformstate-and-useformstatus-a5ee68d6bf93?utm_source=chatgpt.com "Enhancing Form Handling in React 19"
[6]: https://react.dev/reference/rsc/server-components?utm_source=chatgpt.com "Server Components"
[7]: https://react.dev/reference/rsc/use-server?utm_source=chatgpt.com "'use server' directive"
[8]: https://nextjs.org/learn/react-foundations/server-and-client-components?utm_source=chatgpt.com "Server and Client Components - React Foundations"
[9]: https://reactrouter.com/how-to/react-server-components?utm_source=chatgpt.com "React Server Components"
---

# Aula 02
Ótimo 👨‍🏫
Vou montar um **roteiro prático de aprendizado de React** com **mini-projetos progressivos**, cada um cobrindo conceitos-chave e com exemplos que você pode implementar.

---

# 🚀 Roteiro prático de estudos em React

## 1. Fundamentos do React (JSX, componentes, props e state)

📌 **Mini-projeto:** Contador simples

* Criar um componente `<Contador />` com botão que incrementa um número.
* Usar `useState` para armazenar o valor.
* Aprender a passar `props` (ex: valor inicial).

---

## 2. Hooks essenciais e ciclo de vida

📌 **Mini-projeto:** Lista de tarefas (To-Do List)

* Adicionar e remover tarefas.
* Usar `useState` para a lista.
* Usar `useEffect` para salvar no `localStorage`.
* Introduzir `useRef` para capturar o input.

---

## 3. Roteamento

📌 **Mini-projeto:** SPA com múltiplas páginas

* Instalar **React Router**.
* Criar páginas: Home, Sobre e Contato.
* Navegar entre elas sem recarregar o navegador.

---

## 4. Consumo de API e Suspense

📌 **Mini-projeto:** App de busca de usuários

* Criar busca na API do GitHub.
* Mostrar estado de carregamento (`loading`).
* Introduzir `Suspense` com fallback para simular carregamento.

---

## 5. Formulários modernos (React 19 – Actions e `useActionState`)

📌 **Mini-projeto:** Formulário de login

* Criar `<form>` com `action={minhaAction}`.
* Usar `useActionState` para tratar envio, erro e estado pendente.
* Mostrar feedback para o usuário (“Salvando…”, erros, etc.).

---

## 6. Estado global

📌 **Mini-projeto:** Carrinho de compras

* Criar contexto (`CartContext`).
* Adicionar e remover itens.
* Usar `useReducer` para gerenciar o estado do carrinho.
* Mostrar quantidade total em um ícone fixo.

---

## 7. React Server Components (com Next.js)

📌 **Mini-projeto:** Dashboard de produtos

* Criar projeto Next.js (App Router).
* Buscar produtos do banco (ou API mock) diretamente em um Server Component.
* Renderizar tabela no servidor → cliente só recebe HTML leve.

---

## 8. Server Actions e formulários com Next.js

📌 **Mini-projeto:** CRUD de produtos

* Criar Server Actions para `createProduct`, `updateProduct`, `deleteProduct`.
* Conectar formulários a essas Actions.
* Usar `useFormStatus` para mostrar estado de loading.

---

## 9. Testes e qualidade

📌 **Mini-projeto:** Testando a To-Do List (do item 2)

* Usar **React Testing Library** e **Vitest**.
* Testar: adicionar tarefa, marcar como concluída, remover.
* Garantir acessibilidade (botões com labels corretos).

---

## 10. Deploy e boas práticas

📌 **Mini-projeto:** Publicando o Dashboard no Vercel

* Criar repositório no GitHub.
* Deploy automático no Vercel.
* Configurar variáveis de ambiente.
* Usar **Lighthouse** para checar performance e acessibilidade.

---

👉 Esse roteiro cobre a jornada de **iniciante → intermediário → avançado** no React moderno, passando de **JSX** até **Server Actions** em Next.js.

---

Quer que eu prepare os **códigos-base de cada mini-projeto** (já prontos para rodar no VSCode) ou prefere que eu monte **passo a passo guiado** para você ir codando e entendendo?

---

Aula 03
# Roteiro prático — **Contador simples** (passo a passo para codar e entender)

Vou ser objetivo e direto. Siga estes passos no seu ambiente (Node.js instalado).

---

## 1) Criar o projeto (Vite + React)

No terminal:

```bash
# cria o projeto
npm init vite@latest contador-leotech -- --template react

cd contador-leotech
npm install

# iniciar servidor de desenvolvimento
npm run dev
```

Abra `http://localhost:5173` (porta mostrada no terminal).

---

## 2) Estrutura mínima (arquivos que vamos usar)

```
contador-leotech/
├─ index.html
├─ src/
│  ├─ main.jsx
│  ├─ App.jsx
│  ├─ styles.css
│  └─ components/
│     └─ Contador.jsx
```

---

## 3) Componente `Contador.jsx`

Crie `src/components/Contador.jsx` com este código:

```jsx
import React, { useState } from "react";

export default function Contador({ inicial = 0, passo = 1 }) {
  // state local: valor do contador
  const [valor, setValor] = useState(inicial);

  // atualiza usando função (prevState) para evitar problemas de concorrência
  const incrementar = () => setValor(v => v + passo);
  const decrementar = () => setValor(v => v - passo);
  const reset = () => setValor(inicial);

  return (
    <div className="contador">
      <h2>Contador</h2>
      <p>Valor: <strong>{valor}</strong></p>

      <div className="botoes">
        <button aria-label="decrementar" onClick={decrementar}>−</button>
        <button aria-label="incrementar" onClick={incrementar}>+</button>
        <button aria-label="reset" onClick={reset}>Reset</button>
      </div>
    </div>
  );
}
```

**Explicação rápida**:

* `useState(inicial)` cria o estado local `valor` com valor inicial vindo da `prop` `inicial`.
* `incrementar`/`decrementar` usam o *functional update* `setValor(v => v + passo)` — recomendado quando o novo valor depende do anterior.
* `inicial` e `passo` são **props** com valores padrão (`0` e `1`).

---

## 4) Usar o componente em `App.jsx`

Edite `src/App.jsx`:

```jsx
import React from "react";
import Contador from "./components/Contador";
import "./styles.css";

export default function App() {
  return (
    <main>
      <h1>Exemplo: Contador</h1>

      {/* contador com props */}
      <Contador inicial={5} passo={2} />

      {/* contador com valores padrão */}
      <Contador />
    </main>
  );
}
```

Rode a página e verifique dois contadores funcionando (um inicia em 5 e soma 2).

---

## 5) Estilos rápidos (`styles.css`)

Crie `src/styles.css`:

```css
body { font-family: system-ui, Arial; padding: 24px; }
.contador { border: 1px solid #ddd; padding: 12px; margin: 8px 0; width: 220px; }
.botoes button { margin-right: 6px; padding: 6px 10px; }
```

---

## 6) Teste e verificação

* Clique em `+` / `−` / `Reset` e observe o valor.
* Abra DevTools → Components (React DevTools) para inspecionar estado.
* Se quiser ver logs, adicione `console.log(valor)` dentro do componente (apenas para debugging).

---

## 7) Pontos importantes e boas práticas (resumido)

* **Imutabilidade**: não mutacione diretamente `valor`, use `setValor`.
* **Functional update** evita problemas quando várias atualizações ocorrem juntas.
* **Props** são imutáveis dentro do componente; se a prop `inicial` mudar e você quiser que o state acompanhe, use `useEffect` para sincronizar (explico se quiser).
* **Acessibilidade**: botões com `aria-label` e elementos semânticos.

---

## 8) Exercícios rápidos (próximo passo)

1. Adicionar input para alterar `passo` dinamicamente.
2. Persistir o valor no `localStorage` com `useEffect`.
3. Converter para TypeScript (`.tsx`) e tipar props.

Se quiser, eu já gero os trechos para `localStorage` e o input para `passo` — qual desses você prefere agora?
