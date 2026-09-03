# Prompt Templates

## Overview

A prompt is the instruction or input we send to an LLM.

A **prompt template** is a reusable structure for creating prompts dynamically.

Instead of writing:

```python
prompt = "Explain LangChain in simple language."
```

we can create:

```python
prompt = "Explain {topic} in simple language."
```

Then provide the value:

```text
topic = "RAG"
```

The template produces:

```text
Explain RAG in simple language.
```

The main idea is:

```text
Prompt Template
      +
    Variables
      ↓
Formatted Prompt
      ↓
      Model
```

---

# 1. What is a Prompt?

A prompt is the input or instruction given to a model.

Example:

```text
Explain LangChain in simple language.
```

A prompt can contain:

- Instructions
- Context
- User questions
- Examples
- Constraints
- Output requirements

---

# 2. What is a Prompt Template?

A prompt template is a reusable prompt containing variables.

Example:

```text
Explain {topic} in simple language.
```

Here:

```text
{topic}
```

is a variable.

If:

```text
topic = "LangChain"
```

the final prompt becomes:

```text
Explain LangChain in simple language.
```

Therefore:

```text
One Template
      +
Different Variables
      ↓
Different Prompts
```

---

# 3. Why Use Prompt Templates?

Without a template:

```python
prompt1 = "Explain LangChain."
prompt2 = "Explain RAG."
prompt3 = "Explain LangGraph."
```

With a template:

```python
prompt = "Explain {topic}."
```

You can reuse it with:

```text
LangChain
RAG
LangGraph
```

Advantages:

- Reusable
- Easier to maintain
- Dynamic
- Consistent
- Easier to compose into chains

---

# 4. `PromptTemplate`

`PromptTemplate` is designed for reusable text prompts.

```python
from langchain_core.prompts import PromptTemplate

prompt = PromptTemplate.from_template(
    "Explain {topic} in simple language."
)
```

The template contains:

```text
Explain {topic} in simple language.
```

---

# 5. Invoking a `PromptTemplate`

Provide values for the variables:

```python
prompt_value = prompt.invoke({
    "topic": "LangChain"
})
```

Conceptually:

```text
Input
{
    "topic": "LangChain"
}

        ↓

PromptTemplate

        ↓

"Explain LangChain in simple language."
```

The prompt creates the formatted input; it does not itself generate an AI answer.

---

# 6. Multiple Variables

A prompt can contain multiple variables.

```python
prompt = PromptTemplate.from_template(
    "Explain {topic} for a {audience} audience."
)
```

Invoke:

```python
prompt_value = prompt.invoke({
    "topic": "RAG",
    "audience": "beginner"
})
```

Result:

```text
Explain RAG for a beginner audience.
```

---

# 7. Required Variables

Suppose:

```python
prompt = PromptTemplate.from_template(
    "Explain {topic}."
)
```

The expected input contains:

```text
topic
```

Correct:

```python
prompt.invoke({
    "topic": "RAG"
})
```

If a required variable is missing, the template cannot be formatted correctly.

---

# 8. Inspecting Variables

You can inspect the variables expected by a prompt:

```python
print(prompt.input_variables)
```

For:

```python
PromptTemplate.from_template(
    "Explain {topic} for a {audience}."
)
```

the variables will correspond to:

```text
topic
audience
```

---

# 9. `ChatPromptTemplate`

`ChatPromptTemplate` is designed for chat models.

Instead of creating one large string, you define different message roles.

```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages([
    (
        "system",
        "You are a helpful LangChain teacher."
    ),
    (
        "human",
        "Explain {topic} in simple language."
    ),
])
```

This creates a structured chat prompt.

---

# 10. Why `ChatPromptTemplate`?

A chat model works naturally with messages.

Instead of:

```text
One Large String
```

we can create:

```text
System Message
       +
Human Message
       ↓
Chat Model
```

Example:

```text
System:
You are a helpful LangChain teacher.

Human:
Explain RAG in simple language.
```

---

# 11. `from_messages()`

`from_messages()` constructs a chat prompt from message definitions.

```python
prompt = ChatPromptTemplate.from_messages([
    (
        "system",
        "You are an expert Python teacher."
    ),
    (
        "human",
        "Explain {topic}."
    ),
])
```

The first tuple defines the system message.

The second defines the human message.

---

# 12. Invoking `ChatPromptTemplate`

```python
messages = prompt.invoke({
    "topic": "LangChain"
})
```

Conceptually:

```text
Input
  ↓
ChatPromptTemplate
  ↓
SystemMessage
  +
HumanMessage
```

The result is a prompt representation containing structured messages.

---

# 13. Prompt Template + Model

Connect the prompt to a model:

```python
chain = prompt | model
```

Then:

```python
response = chain.invoke({
    "topic": "LangChain"
})
```

Flow:

```text
Dictionary
    ↓
ChatPromptTemplate
    ↓
Messages
    ↓
Chat Model
    ↓
AIMessage
```

---

# 14. Prompt Templates Are Runnables

A prompt template can be invoked:

```python
prompt.invoke(...)
```

Therefore, it participates in the Runnable-style interface.

---

# 15. Dynamic Prompts

Prompt templates allow prompts to change based on input.

```python
prompt = ChatPromptTemplate.from_messages([
    (
        "system",
        "You are a {role}."
    ),
    (
        "human",
        "Explain {topic} at a {level} level."
    ),
])
```

Input:

```python
{
    "role": "teacher",
    "topic": "RAG",
    "level": "beginner"
}
```

Result:

```text
System:
You are a teacher.

Human:
Explain RAG at a beginner level.
```

---

# 16. `MessagesPlaceholder`

Sometimes we already have a list of messages and want to insert them into a prompt.

This is where `MessagesPlaceholder` is useful.

```python
from langchain_core.prompts import (
    ChatPromptTemplate,
    MessagesPlaceholder,
)

prompt = ChatPromptTemplate.from_messages([
    (
        "system",
        "You are a helpful assistant."
    ),
    MessagesPlaceholder(
        variable_name="history"
    ),
    (
        "human",
        "{question}"
    ),
])
```

Supply:

```python
{
    "history": messages,
    "question": "What is RAG?"
}
```

---

# 17. Why `MessagesPlaceholder` Matters

It is useful for:

- Conversation history
- Agents
- Chat applications
- Memory/state
- Tool calls

Architecture:

```text
System Prompt
      ↓
MessagesPlaceholder
      ↓
Conversation History
      ↓
Current Question
      ↓
Chat Model
```

---

# 18. Prompt Templates in RAG

A typical RAG prompt might be:

```text
System:
Answer the question using the provided context.

Context:
{context}

Human:
{question}
```

In LangChain:

```python
prompt = ChatPromptTemplate.from_messages([
    (
        "system",
        "Answer the question using the provided context.

"
        "Context:
{context}"
    ),
    (
        "human",
        "{question}"
    ),
])
```

Then:

```python
prompt.invoke({
    "context": "LangChain is a framework...",
    "question": "What is LangChain?"
})
```