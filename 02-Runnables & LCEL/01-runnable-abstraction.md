# Runnable Abstraction

Runnables are one of the most important concepts in modern LangChain. They provide a common interface for working with components such as prompts, chat models, output parsers, retrievers, and custom functions.

---

## 1. What is a Runnable?

A **Runnable** is a LangChain component that accepts an input and produces an output.

```text
Input
  ↓
Runnable
  ↓
Output
```

For example:

```text
Input
  ↓
Chat Model
  ↓
AIMessage
```

A chat model can be treated as a Runnable because it receives input and produces output.

Similarly:

```text
Dictionary Input
  ↓
Prompt Template
  ↓
Formatted Prompt
```

---

## 2. Why Does LangChain Use Runnables?

LangChain applications contain many different components:

- Prompt templates
- Chat models
- Output parsers
- Retrievers
- Tools
- Custom Python functions

Runnables provide a common interface for interacting with these components.

```text
Prompt
   ↓
Model
   ↓
Output Parser
```

This consistency makes components easier to connect and reuse.

---

## 3. The Runnable Interface

The most important Runnable methods are:

| Method | Purpose |
|---|---|
| `invoke()` | Run with one input |
| `ainvoke()` | Run one input asynchronously |
| `batch()` | Run multiple inputs |
| `abatch()` | Run multiple inputs asynchronously |
| `stream()` | Stream output synchronously |
| `astream()` | Stream output asynchronously |

```text
                Runnable
                   │
       ┌───────────┼───────────┐
       ↓           ↓           ↓
     invoke      batch       stream
       ↓           ↓           ↓
    One Input  Many Inputs  Output Chunks
```

---

## 4. Examples of Runnables

### Chat Model

```python
response = model.invoke("What is LangChain?")
```

```text
Input
  ↓
Chat Model Runnable
  ↓
AIMessage
```

### Prompt Template

```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_template(
    "Explain {topic} simply."
)

result = prompt.invoke({
    "topic": "LangChain"
})
```

```text
Dictionary Input
       ↓
Prompt Runnable
       ↓
Formatted Prompt
```

### Output Parser

```python
from langchain_core.output_parsers import StrOutputParser

parser = StrOutputParser()

result = parser.invoke(response)
```

```text
AIMessage
    ↓
Output Parser Runnable
    ↓
String
```

---

## 5. `invoke()`

`invoke()` is the most basic Runnable method.

It runs a Runnable with a single input and returns the result.

```python
output = runnable.invoke(input)
```

Example:

```python
response = model.invoke(
    "Explain LangChain in simple words."
)

print(response.content)
```

```text
One Input
    ↓
invoke()
    ↓
Runnable
    ↓
One Output
```

---

## 6. `ainvoke()`

`ainvoke()` is the asynchronous version of `invoke()`.

```python
result = await runnable.ainvoke(input)
```

Example:

```python
response = await model.ainvoke(
    "Explain LangChain."
)

print(response.content)
```

Useful for:

- Async applications
- Web servers
- Concurrent workflows
- I/O-heavy applications

---

## 7. `invoke()` vs `ainvoke()`

### Synchronous

```python
result = runnable.invoke(input)
```

```text
Start
  ↓
Run Runnable
  ↓
Wait
  ↓
Get Result
```

### Asynchronous

```python
result = await runnable.ainvoke(input)
```

```text
Start
  ↓
Run Runnable
  ↓
Other Async Work Can Continue
  ↓
Get Result
```

Simple rule:

```text
Normal synchronous code → invoke()

Async application → ainvoke()
```

---

## 8. `batch()`

`batch()` processes multiple inputs.

Instead of:

```python
result1 = runnable.invoke(input1)
result2 = runnable.invoke(input2)
result3 = runnable.invoke(input3)
```

you can use:

```python
results = runnable.batch([
    input1,
    input2,
    input3
])
```

Example:

```python
responses = model.batch([
    "What is LangChain?",
    "What is Python?",
    "What is an API?"
])
```

```text
Input 1 ──┐
Input 2 ──┼──→ batch() ──→ Output 1
Input 3 ──┘                 Output 2
                            Output 3
```

Useful for:

- Processing many documents
- Generating multiple summaries
- Classifying multiple texts
- Running the same prompt for many inputs

---

## 9. `abatch()`

`abatch()` is the asynchronous version of `batch()`.

```python
results = await runnable.abatch([
    input1,
    input2,
    input3
])
```

```text
Multiple Inputs
      ↓
   abatch()
      ↓
Async Processing
      ↓
Multiple Outputs
```

---

## 10. `batch()` vs `abatch()`

| Method | Type | Input |
|---|---|---|
| `batch()` | Synchronous | Multiple inputs |
| `abatch()` | Asynchronous | Multiple inputs |

```text
batch()                  abatch()
   ↓                        ↓
Multiple Inputs         Multiple Inputs
   ↓                        ↓
Multiple Outputs        Async Processing
                            ↓
                        Multiple Outputs
```

---

## 11. `stream()`

`stream()` allows output to be received gradually instead of waiting for the complete result.

```python
for chunk in model.stream(
    "Tell me a short story."
):
    print(chunk.content, end="")
```

Instead of:

```text
Wait...
Wait...
Wait...
Complete Response
```

you receive output incrementally:

```text
Chunk 1
Chunk 2
Chunk 3
Chunk 4
```

```text
Input
  ↓
stream()
  ↓
Runnable
  ↓
Output Chunk
  ↓
Output Chunk
  ↓
Output Chunk
```

Streaming is useful for:

- Chat applications
- AI assistants
- Interactive user interfaces
- Long responses

---

## 12. `astream()`

`astream()` is the asynchronous version of `stream()`.

```python
async for chunk in model.astream(
    "Tell me a short story."
):
    print(chunk.content, end="")
```

```text
Input
  ↓
astream()
  ↓
Async Runnable
  ↓
Chunk
  ↓
Chunk
  ↓
Chunk
```

---

## 13. `stream()` vs `astream()`

### Synchronous Streaming

```python
for chunk in runnable.stream(input):
    print(chunk)
```

### Asynchronous Streaming

```python
async for chunk in runnable.astream(input):
    print(chunk)
```

Simple rule:

```text
Synchronous application → stream()

Asynchronous application → astream()
```

---

## 14. Runnable Input and Output

Every Runnable has an input and an output.

### Prompt Runnable

```text
Input:
{
    "topic": "LangChain"
}

        ↓

Output:
Formatted Prompt
```

### Model Runnable

```text
Input:
Prompt / Messages

        ↓

Output:
AIMessage
```

### Parser Runnable

```text
Input:
AIMessage

        ↓

Output:
String
```

---

## 15. Runnable Pipeline

Consider:

```python
chain = prompt | model | parser
```

Each component is a Runnable.

```text
Input
  ↓
Prompt Runnable
  ↓
Formatted Prompt / Messages
  ↓
Model Runnable
  ↓
AIMessage
  ↓
Parser Runnable
  ↓
Final Output
```

This is the foundation of **LCEL**.

---

## 16. A Simple Runnable Example

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_template(
    "Explain {topic} in simple words."
)

parser = StrOutputParser()

chain = prompt | model | parser

result = chain.invoke({
    "topic": "Runnables"
})

print(result)
```

Flow:

```text
Input Dictionary
       ↓
Prompt
       ↓
Messages
       ↓
Model
       ↓
AIMessage
       ↓
Output Parser
       ↓
String
```

Every major component in this pipeline is a Runnable.