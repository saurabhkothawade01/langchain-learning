# LangChain Messages

## Overview

Messages are one of the most important concepts when working with LangChain chat models.

A message represents a piece of information in a conversation and includes a **role** and **content**.

A simplified conversation looks like:

```text
System Message
      ↓
Human Message
      ↓
AI Message
```

Instead of treating a conversation as one large string, LangChain represents it as structured messages.

---

# 1. What is a Message?

A message is a structured representation of a piece of communication.

A simplified message can be thought of as:

```text
Message
 ├── Role
 └── Content
```

For example:

```text
Role:
human

Content:
"What is LangChain?"
```

Another:

```text
Role:
system

Content:
"You are a helpful teacher."
```

This structure allows the model to understand who is saying what.

---

# 2. `HumanMessage`

`HumanMessage` represents input from the user.

```python
from langchain_core.messages import HumanMessage

message = HumanMessage(
    content="What is LangChain?"
)
```

Conceptually:

```text
HumanMessage
      ↓
User input
```

---

# 3. `AIMessage`

`AIMessage` represents a response from the AI model.

```python
from langchain_core.messages import AIMessage

message = AIMessage(
    content="LangChain is a framework for building LLM applications."
)
```

When you call:

```python
response = model.invoke(
    "What is LangChain?"
)
```

the result is commonly an `AIMessage`.

Therefore:

```python
print(response.content)
```

prints the generated text.

---

# 4. `SystemMessage`

`SystemMessage` provides instructions that define how the model should behave.

```python
from langchain_core.messages import SystemMessage

message = SystemMessage(
    content="You are an expert Python teacher."
)
```

Conceptually:

```text
SystemMessage
      ↓
Model instructions / behavior
```

---

# 5. `ToolMessage`

`ToolMessage` represents information returned by a tool.

Flow:

```text
Tool Call
    ↓
Calculator Tool
    ↓
Tool Result
    ↓
ToolMessage
```

For example:

```text
Calculate 25 × 40.

        ↓

Calculator:
1000

        ↓

ToolMessage:
1000
```

---

# 6. Message Roles

| Message | Role | Meaning |
|---|---|---|
| `SystemMessage` | system | Instructions/behavior |
| `HumanMessage` | human | User input |
| `AIMessage` | assistant/AI | Model response |
| `ToolMessage` | tool | Tool result |

---

# 7. Creating Messages

```python
from langchain_core.messages import (
    SystemMessage,
    HumanMessage,
)

messages = [
    SystemMessage(
        content="You are a helpful teacher."
    ),
    HumanMessage(
        content="What is LangChain?"
    ),
]
```

Now:

```text
messages
   |
   +── SystemMessage
   |
   +── HumanMessage
```

---

# 8. Sending Messages to a Chat Model

A chat model can receive a list of messages.

```python
response = model.invoke(messages)

print(response.content)
```

Architecture:

```text
SystemMessage
      +
HumanMessage
      ↓
   Chat Model
      ↓
   AIMessage
      ↓
response.content
```

---

# 9. Multiple Messages

A conversation can contain many messages.

```python
messages = [
    SystemMessage(
        content="You are a helpful teacher."
    ),
    HumanMessage(
        content="What is LangChain?"
    ),
    AIMessage(
        content="LangChain is a framework for building LLM applications."
    ),
    HumanMessage(
        content="Give me a simple example."
    ),
]
```

The model receives the entire sequence.

Conceptually:

```text
System
  ↓
Human
  ↓
AI
  ↓
Human
  ↓
Model
  ↓
AI
```

---

# 10. Message History

A conversation can be represented as a list of messages:

```text
[
    SystemMessage,
    HumanMessage,
    AIMessage,
    HumanMessage,
    AIMessage
]
```

This history provides context for the model.

Without history:

```text
User:
What is RAG?
```

With history:

```text
Human:
What is LangChain?

AI:
LangChain is...

Human:
What is RAG?
```

The model can use the previous conversation as context.

---

# 11. Message Content

The main content of a message can be accessed using:

```python
message.content
```

Example:

```python
message = HumanMessage(
    content="What is LangChain?"
)

print(message.content)
```

For an AI response:

```python
response = model.invoke(
    "Explain LangChain."
)

print(response.content)
```

---

# 12. Message Metadata

Messages can contain additional information beyond the main content.

Depending on the message type and provider, you may encounter:

```text
additional_kwargs
response_metadata
usage_metadata
tool_calls
```

For example:

```text
AIMessage
 ├── content
 ├── additional_kwargs
 ├── response_metadata
 ├── usage_metadata
 └── tool_calls
```

Not every field is populated for every model or provider.

---

# 13. `AIMessage` and Tool Calls

When a model decides to call a tool, the `AIMessage` can contain structured tool-call information.

Conceptually:

```text
User
 ↓
AIMessage
 ↓
Tool Call
 ↓
Tool
 ↓
ToolMessage
```

For example:

```text
AIMessage:

tool_calls:
    name = "calculator"
    arguments = {
        "a": 25,
        "b": 40
    }
```

The application executes the tool and produces:

```text
ToolMessage:
1000
```

This is the foundation of modern tool-calling agents.

---

# 14. Messages and Prompts

Messages and prompts are related, but they are not exactly the same.

### Message

Represents a single conversation unit.

```text
HumanMessage
AIMessage
SystemMessage
```

### Prompt

Defines how input is transformed into the instructions/messages sent to the model.

Conceptually:

```text
Prompt Template
      ↓
Formatted Prompt
      ↓
Messages
      ↓
Chat Model
```

---

# 15. Plain String vs Messages

## Plain string

```python
response = model.invoke(
    "Explain LangChain."
)
```

This is convenient for simple calls.

## Structured messages

```python
messages = [
    SystemMessage(
        content="You are a LangChain teacher."
    ),
    HumanMessage(
        content="Explain LangChain."
    ),
]

response = model.invoke(messages)
```

This gives you more control over roles and conversation structure.

---

# 16. Practical Example: Simple Conversation

```python
from langchain_core.messages import (
    SystemMessage,
    HumanMessage,
)

messages = [
    SystemMessage(
        content="You are a Python teacher."
    ),
    HumanMessage(
        content="What is a list in Python?"
    ),
]

response = model.invoke(messages)

print(response.content)
```

Add another turn:

```python
messages.append(
    HumanMessage(
        content="Give me an example."
    )
)

response = model.invoke(messages)

print(response.content)
```

The second model call receives the previous messages too.