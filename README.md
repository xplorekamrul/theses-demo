
# Thesys React SDK – README

## 🔍 Overview

Thesys is a **Generative UI platform** that integrates LLM responses with front-end React rendering in real time. Instead of returning plain text, its **C1 API** outputs a structured UI specification (JSON), which is rendered by the **GenUI React SDK** (C1Component / C1Chat). This enables prompts to drive forms, charts, buttons, etc. without manual UI wiring.

---

## ✅ Prerequisites

- Node.js ≥ 14, npm or yarn  
- React.js (or Next.js) **or** React Native  
- A **Thesys C1 API key** (obtainable from Thesys Console)

---

## 📦 Installation

### Web (React / Next.js)

```bash
npx create-next-app thesys-demo
cd thesys-demo
npm install @thesys/genui-sdk
```

### React Native

```bash
npx react-native init MyThesysApp
cd MyThesysApp
npm install @thesys/genui-sdk
```

---

## ⚙️ Backend Integration (C1 API)

Thesys C1 API is OpenAI-compatible with a custom base URL:

```js
import OpenAI from "openai";

const client = new OpenAI({
  apiKey: process.env.THESYS_API_KEY,
  baseURL: "https://api.thesys.dev/v1/embed",
});

const resp = await client.chat.completions.create({
  model: "c1-nightly",
  messages: [
    { role: "user", content: "Show me a form to order a scarf" }
  ],
});

console.log(resp);
```

---

## 🧩 Frontend Rendering

### Using `<C1Component>`

```tsx
import React, { useEffect, useState } from "react";
import { C1Component, ThemeProvider } from "@thesys/genui-sdk";

export default function App() {
  const [c1JSON, setC1JSON] = useState(null);

  useEffect(() => {
    async function fetchSpec() {
      const res = await fetch("/api/chat");
      const body = await res.json();
      setC1JSON(body);
    }
    fetchSpec();
  }, []);

  return (
    <ThemeProvider>
      {c1JSON ? <C1Component c1Response={c1JSON} /> : <div>Loading UI…</div>}
    </ThemeProvider>
  );
}
```

### Using `<C1Chat>` (Chat UI with streaming)
`src/app/page.tsx`
```tsx
'use client';
import "@crayonai/react-ui/styles/index.css";
import { C1Chat } from "@thesys/genui-sdk";

export default function Home() {
  return <C1Chat theme={{ mode: "dark" }} apiUrl="/api/chat" />;
}
```

---

## 🛠️ Backend Tools Example (Next.js API)

### 1. Chat Context Store

`src/app/api/chat/messageStore.ts`

```ts
import OpenAI from "openai";

export type DBMessage = OpenAI.Chat.ChatCompletionMessageParam & { id?: string };

const messages: Record<string, DBMessage[]> = {};

export function getMessageStore(threadId: string) {
  if (!messages[threadId]) messages[threadId] = [];
  return {
    addMessage: (msg: DBMessage) => messages[threadId].push(msg),
    getMessages: () => messages[threadId],
    getOpenAIList: () => messages[threadId].map(({ id, ...m }) => m)
  };
}
```

### 2. Business Logic Tool Example

`src/app/api/chat/orderManagement.ts`

```ts
import { z } from "zod";

const scarfSchema = z.object({
  kind: z.literal("scarves"),
  quantity: z.number(),
  colors: z.enum(["red", "blue", "green", "yellow", "purple", "orange"]),
  deliveryDate: z.string(),
  shipping: z.enum(["normal", "express"]),
});

export const orderSchema = z.object({
  order: scarfSchema,
});

const orders: unknown[] = [];

export async function createOrder(orderInput: unknown) {
  const parsed = orderSchema.safeParse(orderInput);
  if (!parsed.success) {
    return { success: false, error: parsed.error.message };
  }
  orders.push(parsed.data.order);
  return { success: true };
}

export function getOrders() {
  return orders;
}
```


## 🧠 Summary

- Use **Thesys C1 API** to get LLM responses as JSON UI spec.
- Use **GenUI React SDK** (`C1Component` / `C1Chat`) to render those UIs live.
- Integrate with your standard backend tools (e.g. order logic, message storage).
- Both chat and form-driven UIs supported.
- Easily customizable across React & React Native platforms.
