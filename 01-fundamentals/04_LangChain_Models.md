# LangChain Models

## Overview

A **model** is the component that processes input and produces an AI-generated response.

In LangChain, models are one of the most important building blocks.

A simple flow is:

```text
Input
  ↓
Model
  ↓
AI Response
```

In a real application:

```text
User
  ↓
Prompt
  ↓
Chat Model
  ↓
AIMessage
  ↓
Application
```

The model is responsible for the actual language understanding and generation.

---

# 1. What is a Model?

An LLM or chat model is a machine learning model trained to understand and generate language.

Examples of model providers include:

- OpenAI
- Anthropic
- Google
- Groq
- OpenRouter
- Hugging Face
- Local model runtimes

LangChain provides interfaces that allow your application to interact with these models.

Important distinction:

```text
LangChain
    ↓
Framework / orchestration

Model Provider
    ↓
Provides the actual model

LLM / Chat Model
    ↓
Generates the response
```

---

# 2. Model vs LangChain

Remember:

```text
LangChain ≠ Model
```

For example:

```text
Your Python Application
        ↓
     LangChain
        ↓
  Model Integration
        ↓
     OpenRouter
        ↓
   Selected Model
        ↓
     Response
```

LangChain helps your application communicate with and compose the model with other components.

---

# 3. What is an LLM?

An LLM can be thought of as a model that receives text and generates text.

Conceptually:

```text
String Input
     ↓
    LLM
     ↓
String Output
```

Example:

```text
Input:
"Explain Python."

        ↓

      LLM

        ↓

Output:
"Python is a programming language..."
```

---

# 4. What is a Chat Model?

A chat model works with structured messages.

Instead of only receiving one string:

```text
"Explain Python"
```

it can receive:

```text
SystemMessage
HumanMessage
AIMessage
```

Conceptually:

```text
Messages
   ↓
Chat Model
   ↓
AIMessage
```

Example:

```text
System:
You are a Python teacher.

Human:
Explain decorators.

AI:
A decorator is...
```

---

# 5. Provider Integrations

LangChain separates the core framework from many model-provider integrations.

Conceptually:

```text
                    LangChain
                       |
                Model Interface
                       |
       +---------------+---------------+
       |               |               |
    OpenAI          Anthropic        Google
       |               |               |
      GPT             Claude         Gemini
```

You install the integration package you need.

Examples:

```bash
pip install -U langchain-openai
```

```bash
pip install -U langchain-anthropic
```

```bash
pip install -U langchain-google-genai
```

```bash
pip install -U langchain-groq
```

---

# 6. `init_chat_model`

LangChain provides a convenient way to initialize chat models using `init_chat_model`.

Conceptually:

```python
from langchain.chat_models import init_chat_model

model = init_chat_model(
    "provider:model-name"
)
```

The exact provider configuration depends on the provider and environment.

The idea is:

```text
Provider + Model
       ↓
init_chat_model()
       ↓
Chat Model
```

This gives you a standardized model interface.

---

# 7. Why Use `init_chat_model`?

Suppose you want to experiment with different providers.

Conceptually:

```text
Application
    ↓
init_chat_model()
    ↓
Model A
```

Then:

```text
Application
    ↓
init_chat_model()
    ↓
Model B
```

Your application logic can remain largely similar while the underlying model changes.

This is useful for experimentation and model comparison.

---

# 8. OpenRouter with LangChain

OpenRouter provides access to many models through a common API.

A common LangChain approach is to use the OpenAI-compatible integration.

Install:

```bash
pip install -U langchain-openai
```

Environment variable:

```env
OPENROUTER_API_KEY=your_api_key_here
```

Then:

```python
import os

from dotenv import load_dotenv
from langchain_openai import ChatOpenAI

load_dotenv()

model = ChatOpenAI(
    model="your-openrouter-model",
    api_key=os.getenv("OPENROUTER_API_KEY"),
    base_url="https://openrouter.ai/api/v1",
)
```

The important architecture is:

```text
LangChain
    ↓
ChatOpenAI Integration
    ↓
OpenRouter API
    ↓
Selected Model
```

The `model` value must be a model identifier available through your OpenRouter account/provider setup.

---

# 9. Calling a Model with `invoke()`

The simplest way to execute a model is:

```python
response = model.invoke(
    "Explain LangChain in simple words."
)
```

Then:

```python
print(response.content)
```

Flow:

```text
String
  ↓
Chat Model
  ↓
AIMessage
  ↓
response.content
  ↓
String
```

---

# 10. Understanding the Response

A model response is usually more than just a string.

For example:

```python
response = model.invoke("What is LangChain?")
```

You can inspect:

```python
print(type(response))
```

You may see a message type such as:

```text
AIMessage
```

Then:

```python
print(response.content)
```

gets the generated text.

You can also inspect additional information:

```python
print(response.response_metadata)
```

Depending on the provider, the response may contain metadata such as:

- Token usage
- Model information
- Finish reason
- Provider-specific information

---

# 11. Model Parameters

Models can usually be configured with different parameters.

Important parameters include:

- Temperature
- Max tokens / output token limits
- Model name
- Timeout
- Retry configuration
- Streaming configuration
- Provider-specific parameters

---

# 12. Temperature

Temperature controls how deterministic or varied the model's output can be.

A simplified mental model:

```text
Low temperature
      ↓
More predictable
More consistent
```

```text
High temperature
      ↓
More varied
More creative
```

For example:

```python
model = ChatOpenAI(
    model="your-model",
    temperature=0
)
```

For deterministic tasks such as structured extraction, a lower temperature is often preferred.

For creative writing, a higher temperature may be useful.

Important:

> Temperature does not mean "intelligence". It mainly controls the randomness/variability of generation.

---

# 13. Temperature Example

Suppose the prompt is:

```text
Give me a name for a coffee shop.
```

With lower temperature, the model may produce more predictable results.

With higher temperature, the model may generate more varied results.

Conceptually:

```text
Temperature = 0
      ↓
Consistent style

Temperature = higher
      ↓
More variation
```

Exact behavior depends on the model and provider.

---

# 14. Max Output Tokens

Models have limits on how much output they can generate.

A token is a unit used by the model to represent text.

For example:

```text
Input tokens
     +
Output tokens
     =
Context / usage
```

A model may allow configuration of the maximum output length.

The exact parameter name can vary by provider/integration.

The important concept is:

```text
Maximum output tokens
        ↓
Controls maximum generated output
```

---

# 15. Timeout

A timeout prevents your application from waiting indefinitely for a model response.

Conceptually:

```text
Application
     ↓
   Model
     ↓
Response within timeout → Continue
```

If the model takes too long:

```text
Timeout
   ↓
Error / Retry / Fallback
```

Timeouts become important in production systems.

---

# 16. Retries

Temporary failures can happen because of:

- Network problems
- Rate limits
- Provider errors
- Temporary service failures

Retry configuration can help handle transient failures.

Conceptually:

```text
Request
  ↓
Failure
  ↓
Retry
  ↓
Success
```

Do not blindly retry every error. Permanent errors such as invalid API keys usually need configuration changes rather than repeated retries.

---

# 17. Model Configuration

A model can be configured according to the application.

Example:

```python
model = ChatOpenAI(
    model="your-model",
    temperature=0,
    timeout=30,
    max_retries=2,
)
```

The exact supported parameters depend on the integration.

---

# 18. Model Metadata

Model responses can contain metadata.

Example:

```python
response = model.invoke("Explain LangChain")

print(response.response_metadata)
```

Depending on the model/provider, metadata may contain:

```text
Model information
Token usage
Finish reason
Provider information
Other provider-specific fields
```

Metadata becomes useful for:

- Debugging
- Cost analysis
- Monitoring
- Evaluation
- Production observability

---

# 19. Token Usage

Models process tokens rather than raw characters.

A response may provide usage information such as:

```text
Input tokens
Output tokens
Total tokens
```

Conceptually:

```text
Input tokens
     +
Output tokens
     =
Total token usage
```

Token usage can matter for:

- Cost
- Context limits
- Performance
- Application optimization

The exact metadata structure varies by provider.

---

# 20. Context Window

A model also has a context window.

The context contains information such as:

```text
System instructions
+
User messages
+
Previous messages
+
Retrieved documents
+
Tool results
+
Current prompt
```

Conceptually:

```text
              Context Window
        +-----------------------+
        | System instructions   |
        | Conversation history  |
        | User question         |
        | Retrieved context     |
        | Tool results          |
        +-----------------------+
                    ↓
                  Model
```

The context window is different from the maximum output tokens.

---

# 21. `invoke()`

`invoke()` is used when you want one complete result.

```python
response = model.invoke(
    "Explain RAG."
)
```

Mental model:

```text
One Input
   ↓
Model
   ↓
One Result
```

---

# 22. `stream()`

`stream()` allows the response to be consumed progressively.

Streaming Architecture

```text
User
 ↓
Application
 ↓
Model
 ↓
Chunk 1 ──→ UI
Chunk 2 ──→ UI
Chunk 3 ──→ UI
Chunk 4 ──→ UI
...
```

Example:

```python
for chunk in model.stream(
    "Explain LangChain in detail."
):
    print(chunk.content, end="")
```

Instead of:

```text
Wait
 ↓
Complete response
```

you get:

```text
Chunk 1
Chunk 2
Chunk 3
Chunk 4
...
```

This is especially useful for chat interfaces.

---

# 23. `batch()`

`batch()` allows you to process multiple inputs.

Example:

```python
responses = model.batch([
    "What is LangChain?",
    "What is LangGraph?",
    "What is RAG?"
])
```

Conceptually:

```text
Input 1 ─┐
Input 2 ─┼──→ Model ──→ Results
Input 3 ─┘
```

This is useful when you have many independent inputs.

---

# 24. `ainvoke()`

For asynchronous applications:

```python
response = await model.ainvoke(
    "Explain LangChain."
)
```

Architecture:

```text
Async Application
       ↓
   ainvoke()
       ↓
     Model
       ↓
   AIMessage
```

This is useful in frameworks such as FastAPI and other asynchronous applications.

---

# 25. `astream()`

For asynchronous streaming:

```python
async for chunk in model.astream(
    "Explain LangChain."
):
    print(chunk.content, end="")
```

Conceptually:

```text
Async Application
       ↓
    astream()
       ↓
Chunk 1
Chunk 2
Chunk 3
Chunk 4
```

---

# 26. `abatch()`

For asynchronous batch processing:

```python
responses = await model.abatch([
    "What is LangChain?",
    "What is RAG?",
    "What is LangGraph?"
])
```

Conceptually:

```text
Multiple Inputs
      ↓
   abatch()
      ↓
Multiple Results
```

---

# 27. Model Abstraction

One major advantage of LangChain is that application code can often interact with different chat models through a common interface.

Conceptually:

```text
                 Application
                      ↓
               Chat Model Interface
                      ↓
        +-------------+-------------+
        |             |             |
      Model A       Model B       Model C
```

This is useful for:

- Experimentation
- A/B testing
- Model comparison
- Provider migration
- Fallback strategies