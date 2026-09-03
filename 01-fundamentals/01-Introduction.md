# LangChain

## Definition

**LangChain is an open-source framework for building applications powered by Large Language Models (LLMs).**

In simple words:

> LangChain provides tools and abstractions that help you connect an LLM with prompts, data, tools, APIs, databases, memory, and other components to build real-world AI applications.

---

## 1. Why do we need LangChain?

A simple LLM application can look like:

```text
Python Application
       ↓
LLM API
       ↓
Response
```

For a real-world application such as "Chat with my company's 1,000 PDF documents", you may need:

```text
PDFs
 ↓
Read documents
 ↓
Split documents
 ↓
Create embeddings
 ↓
Store vectors
 ↓
Search relevant information
 ↓
Create prompt
 ↓
Send context + question to LLM
 ↓
Generate answer
```

LangChain provides reusable abstractions for many of these components.

---

## 2. What does LangChain provide?

Think of LangChain as a **toolkit**.

| Component | Purpose |
|---|---|
| Models | Connect to LLMs |
| Messages | Represent conversations |
| Prompts | Create reusable prompts |
| Output parsers | Convert LLM output into useful formats |
| Runnables | Compose components together |
| Document loaders | Load external data |
| Text splitters | Break large documents into chunks |
| Embeddings | Convert text into vectors |
| Vector stores | Store/search vectors |
| Retrievers | Retrieve relevant information |
| Tools | Allow LLMs to interact with external systems |
| Agents | Allow LLMs to decide which tools to use |

---

## 3. LangChain is NOT an LLM

This is one of the most important concepts.

**LangChain != GPT**

**LangChain != Claude**

**LangChain != Gemini**

Instead:

```text
                 LangChain
                    |
       +------------+------------+
       |            |            |
     OpenAI       Anthropic    Google
       |            |            |
      GPT          Claude      Gemini
```

LangChain acts as an application framework/orchestration layer around models and other components.

---

## 4. Simple analogy

Imagine you're building a restaurant.

### LLM = Chef

The chef can prepare food.

### Prompt = Order

You tell the chef what you want.

### LangChain = Restaurant infrastructure

It helps coordinate:

- Customer orders
- Kitchen
- Ingredients
- Tools
- Database
- Workflows
- Output

Similarly:

```text
User
   ↓
LangChain Application
   ↓
Prompt / Context / Tools
   ↓
LLM
   ↓
Response
```

---

## 5. What problems does LangChain solve?

### Problem 1 — Connecting different models

You might start with OpenAI and later want to try:

- Gemini
- Claude
- Groq
- OpenRouter
- Local models

LangChain provides standardized interfaces that make switching models easier.

### Problem 2 — Managing prompts

Instead of writing prompts everywhere, you can create reusable prompt templates.

Conceptually:

```text
Prompt Template
       +
    Variables
       ↓
     Prompt
```

### Problem 3 — Connecting multiple components

Suppose you want:

```text
Question
   ↓
Prompt
   ↓
LLM
   ↓
Parser
   ↓
Final Result
```

LangChain allows you to compose these components into a chain.

For example:

```python
chain = prompt | model | parser
```

The `|` operator becomes very important when learning **LCEL (LangChain Expression Language)**.

### Problem 4 — Connecting LLMs to external data

An LLM does not automatically know your private documents.

For example:

```text
Company PDFs
Employee policies
Invoices
Database
Internal documentation
```

LangChain provides components that help build systems such as:

```text
Private Documents
       ↓
Retriever
       ↓
Relevant Context
       ↓
LLM
       ↓
Answer
```

This is the foundation of **RAG (Retrieval-Augmented Generation)**.

### Problem 5 — Giving LLMs tools

You can expose capabilities such as:

```text
Calculate something
Query your database
Call your API
Search your application
Send an email
```

as **tools**.

Then an agent can decide which tool to use:

```text
User
 ↓
Agent
 ↓
Choose Tool
 ↓
Database Tool
 ↓
Result
 ↓
Agent
 ↓
Answer
```

---

## 6. LangChain's core idea

The fundamental idea is:

> **Build AI applications by composing reusable components.**

Instead of creating one giant program, you create small components:

```text
Prompt
   ↓
Model
   ↓
Parser
   ↓
Retriever
   ↓
Tool
   ↓
Agent
```

and connect them.

This concept of **composition** is extremely important throughout LangChain.

---

## 7. LangChain vs calling an LLM directly

### Direct API

```python
response = client.chat.completions.create(...)
```

Architecture:

```text
Python
  ↓
Provider API
  ↓
LLM
```

### LangChain

```text
Python
  ↓
LangChain
  ↓
Model
  ↓
LLM
```

LangChain becomes especially useful when your application grows:

```text
                   LangChain
                       |
        +--------------+--------------+
        |              |              |
      Prompt         Retriever       Tools
        |              |              |
        +--------------+--------------+
                       ↓
                      LLM
                       ↓
                    Parser
                       ↓
                    Output
```

---

## 8. LangChain ecosystem

### LangChain

Used for building LLM application components and workflows.

```text
Models
Prompts
Retrievers
Tools
Agents
Runnables
```

### LangGraph

Used for building more complex, stateful, controllable agent workflows.

```text
Nodes
 ↓
Edges
 ↓
State
 ↓
Loops
 ↓
Human approval
 ↓
Persistence
```

### LangSmith

Used for:

```text
Tracing
Debugging
Evaluation
Monitoring
Testing
```
```text
        Your AI Application
               |
       +-------+--------+
       |                |
   LangChain         LangGraph
       |                |
       +-------+--------+
               ↓
          LangSmith
```

---

## 9. Where LangChain fits in a real application

For an **Invoice RAG application**, a possible architecture is:

```text
                    User
                     ↓
                 FastAPI
                     ↓
                LangChain
                     ↓
       +-------------+-------------+
       |             |             |
    Prompt        Retriever      Tools
                     ↓
                Vector Store
                     ↓
                Relevant Docs
                     ↓
                    LLM
                     ↓
                 Response
```