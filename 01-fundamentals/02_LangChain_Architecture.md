# LangChain Architecture

## Overview

A simplified view:

```text
                    Your Application
                           |
                       LangChain
                           |
        +------------------+------------------+
        |                  |                  |
      Models             Prompts           Tools
        |                  |                  |
        +------------------+------------------+
                           |
                       Runnables
                           |
                +----------+----------+
                |                     |
             Retrievers           Agents
                |                     |
              RAG                Tool Calling
```


---

# 1. Main Parts of the LangChain Ecosystem

The major concepts you will learn are:

1. `langchain-core`
2. Model integrations
3. Messages
4. Prompts
5. Output parsers
6. Runnables
7. Document loaders
8. Text splitters
9. Embeddings
10. Vector stores
11. Retrievers
12. Tools
13. Agents
14. LangGraph
15. LangSmith

We will learn them gradually.

---

# 2. `langchain-core`

`langchain-core` contains many of the fundamental abstractions used throughout the LangChain ecosystem.

Important concepts include:

- Messages
- Prompts
- Runnables
- Output parsers
- Tools
- Runnable configuration
- Callbacks

A simplified view:

```text
                 langchain-core
                       |
       +---------------+---------------+
       |               |               |
    Messages        Prompts        Runnables
       |               |               |
       +---------------+---------------+
                       |
                 Output Parsers
                       |
                    Tools
```

Think of `langchain-core` as the **foundation layer**.

---

# 3. Model Integrations

LangChain itself is not the model.

Models are provided by different companies and projects.

Examples:

- OpenAI
- Anthropic
- Google
- Groq
- OpenRouter
- Hugging Face
- Local models

Conceptually:

```text
                    LangChain
                       |
                 Model Interface
                       |
        +--------------+--------------+
        |              |              |
      OpenAI        Anthropic       Google
        |              |              |
       GPT            Claude         Gemini
```

This separation allows your application to work with different model providers.

---

# 4. Models

The model is responsible for generating or transforming information.

For chat applications, you will commonly work with chat models.

Basic flow:

```text
Input
  ↓
Chat Model
  ↓
AI Response
```

A simplified Python example:

```python
response = model.invoke("What is LangChain?")
```

The important method concepts are:

```text
invoke()   → execute one input
stream()   → receive output progressively
batch()    → process multiple inputs
ainvoke()  → asynchronous invocation
astream()  → asynchronous streaming
abatch()   → asynchronous batch processing
```


---

# 5. Messages

Chat models work with messages.

The most important message types are:

```text
SystemMessage
HumanMessage
AIMessage
ToolMessage
```

A simple conversation:

```text
SystemMessage
      ↓
"You are a helpful teacher."

HumanMessage
      ↓
"What is LangChain?"

AIMessage
      ↓
"LangChain is..."
```

Messages provide structure to a conversation.

---

# 6. Prompts

A prompt tells the model what you want it to do.

For example:

```text
You are a Python teacher.

Explain the following topic:
{topic}
```

Instead of hardcoding the value, we can use a prompt template.

Conceptually:

```text
Prompt Template
       +
    Variables
       ↓
     Prompt
```

Example:

```python
prompt = ChatPromptTemplate.from_template(
    "Explain {topic} in simple language."
)
```

Later:

```python
prompt_value = prompt.invoke({"topic": "LangChain"})
```

---

# 7. Output Parsers

LLMs usually return text or structured responses.

An output parser helps transform the model output into the format your application needs.

Example:

```text
LLM
 ↓
Raw Response
 ↓
Output Parser
 ↓
Structured Result
```

Possible outputs:

```text
String
JSON
Pydantic Object
List
Dictionary
```

For example:

```text
LLM
 ↓
"Name: Saurabh, Age: 25"
 ↓
Parser
 ↓
Structured Object
```
---
# 8. Runnables

**Runnables are one of the most important concepts in modern LangChain.**

A Runnable is a component that follows a common interface for execution.

Common operations include:

```text
invoke()
stream()
batch()
ainvoke()
astream()
abatch()
```

Many LangChain components can work together because they follow this common Runnable interface.

---

# 9. LCEL

LCEL stands for:

**LangChain Expression Language**

It provides a concise way to compose Runnable components.

For example:

```python
chain = prompt | model | parser
```

The pipeline means:

```text
Input
  ↓
Prompt
  ↓
Model
  ↓
Parser
  ↓
Output
```

The `|` operator is called the **pipe operator**.

---
# 10. Document Loaders

Real applications often need external data.

Examples:

```text
PDF
TXT
CSV
JSON
HTML
Markdown
Web pages
Word documents
```

Document loaders help bring this data into your application.

Architecture:

```text
External Data
     ↓
Document Loader
     ↓
Document Objects
```

A LangChain `Document` commonly contains:

```text
page_content
metadata
```

For example:

```text
Document
 ├── page_content
 └── metadata
```

---

# 11. Text Splitters

Large documents cannot always be passed directly to an LLM or vector database.

We split them into smaller pieces called **chunks**.

```text
Large Document
      ↓
Text Splitter
      ↓
Chunk 1
Chunk 2
Chunk 3
Chunk 4
```

Important concepts:

- Chunk size
- Chunk overlap
- Recursive splitting
- Token-based splitting
- Markdown-aware splitting
- Semantic chunking

This becomes very important when building RAG applications.

---

# 12. Embeddings

Embeddings convert text into numerical vectors.

For example:

```text
"LangChain is a framework"
             ↓
      Embedding Model
             ↓
[0.12, -0.43, 0.87, ...]
```

The vector represents semantic information about the text.

Architecture:

```text
Text
 ↓
Embedding Model
 ↓
Vector
```

Embeddings are commonly used for semantic search and RAG.

---

# 13. Vector Stores

Vectors can be stored in a vector database or vector store.

Examples:

- FAISS
- Chroma
- Pinecone
- Qdrant
- Weaviate
- Milvus

Architecture:

```text
Documents
    ↓
Chunks
    ↓
Embeddings
    ↓
Vector Store
```

When a user asks a question:

```text
Question
   ↓
Question Embedding
   ↓
Vector Search
   ↓
Relevant Chunks
```

---

# 14. Retrievers

A retriever is responsible for finding relevant information.

Example:

```text
User Question
      ↓
   Retriever
      ↓
Relevant Documents
```

For example:

```text
Question:
"What is the total amount of invoice 101?"

             ↓

         Retriever

             ↓

Invoice 101 chunks
```

Retrievers are a key part of RAG.

---

# 15. RAG Architecture

Putting the previous concepts together:

```text
             Documents
                 ↓
          Document Loader
                 ↓
           Text Splitter
                 ↓
             Embeddings
                 ↓
           Vector Store
                 ↓
              Retriever
                 ↓
          Relevant Context
                 ↓
              Prompt
                 ↓
                LLM
                 ↓
              Answer
```

This is the basic architecture of a RAG system.

---

# 16. Tools

Tools allow an LLM or agent to interact with external functionality.

Examples:

```text
Calculator
Database
Search API
Weather API
Python function
REST API
File system
```

Architecture:

```text
LLM / Agent
     ↓
   Tool
     ↓
External System
     ↓
   Result
     ↓
LLM / Agent
```

A tool can be as simple as a Python function.

Conceptually:

```python
def calculate_total(a, b):
    return a + b
```

This function can be exposed to an agent as a tool.

---

# 17. Agents

An agent is different from a simple chain.

### Chain

A chain generally follows a predefined path:

```text
Input
 ↓
Prompt
 ↓
LLM
 ↓
Parser
 ↓
Output
```

### Agent

An agent can decide what action to take:

```text
User
 ↓
Agent
 ↓
Decide
 ↓
Tool
 ↓
Observe Result
 ↓
Decide Again
 ↓
Final Answer
```

For example:

```text
User:
"What is 25 * 40 and save the result?"

          ↓

       Agent

          ↓
   Calculator Tool

          ↓
        1000

          ↓

     Save Tool

          ↓

       Success

          ↓

       Answer
```

Agents become much more powerful when combined with LangGraph.

---

# 18. LangGraph

LangGraph is designed for building more complex and stateful workflows and agents.

Important concepts include:

- Graphs
- Nodes
- Edges
- State
- Reducers
- Conditional routing
- Loops
- Persistence
- Checkpointing
- Human-in-the-loop

A basic graph:

```text
START
  ↓
Node A
  ↓
Node B
  ↓
Node C
  ↓
 END
```

A more advanced graph:

```text
             START
               ↓
            Agent
           /     \     
       Tool A   Tool B
          \       /
           ↓     ↓
            Agent
               ↓
              END
```

---

# 19. LangSmith

LangSmith is used for observing, debugging, evaluating, and testing LLM applications.

Conceptually:

```text
Your Application
       ↓
LangChain / LangGraph
       ↓
LangSmith
       ↓
Tracing
Debugging
Evaluation
Monitoring
```

For example, when a RAG answer is wrong, tracing can help you inspect:

```text
User Question
      ↓
Retriever
      ↓
Retrieved Documents
      ↓
Prompt
      ↓
LLM
      ↓
Final Answer
```

This becomes extremely useful for production systems.

---


# 20. How a Simple LangChain Application Works

Suppose the user asks:

> "Explain LangChain in simple words."

The flow can be:

```text
User Question
      ↓
ChatPromptTemplate
      ↓
Chat Model
      ↓
Output Parser
      ↓
Final Answer
```

In code, the architecture may look like:

```python
chain = prompt | model | parser

result = chain.invoke(
    {"topic": "LangChain"}
)
```