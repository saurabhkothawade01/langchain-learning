# LCEL (LangChain Expression Language)

LCEL is the composition system used in LangChain to connect Runnables together into pipelines and workflows.

---

# 1. What is LCEL?

**LCEL** stands for **LangChain Expression Language**.

It provides a declarative way to compose LangChain components.

The simplest example is:

```python
chain = prompt | model | parser
```

The `|` operator connects components together.

Conceptually:

```text
Input
  ↓
Prompt
  ↓
Model
  ↓
Output Parser
  ↓
Final Output
```

Each component in this pipeline is a **Runnable**.

---

# 2. Why LCEL Exists

A typical AI application contains multiple steps:

```text
User Input
    ↓
Prompt Formatting
    ↓
Model Call
    ↓
Output Processing
    ↓
Final Result
```

Without LCEL, these steps could be written manually:

```python
prompt_value = prompt.invoke(input_data)

response = model.invoke(prompt_value)

result = parser.invoke(response)
```

With LCEL:

```python
chain = prompt | model | parser

result = chain.invoke(input_data)
```

LCEL makes pipelines easier to:

- Read
- Compose
- Reuse
- Stream
- Run asynchronously
- Process in batches

---

# 3. The `|` Operator

The pipe operator is the most recognizable feature of LCEL.

```python
chain = prompt | model | parser
```

The output of one Runnable becomes the input of the next Runnable.

```text
Prompt
  │
  │ Output
  ↓
Model
  │
  │ Output
  ↓
Parser
```

This can also be understood as:

```text
A | B | C
```

meaning:

```text
Input
  ↓
A
  ↓
B
  ↓
C
  ↓
Output
```

---

# 4. Simple LCEL Example

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_template(
    "Explain {topic} in simple words."
)

parser = StrOutputParser()

chain = prompt | model | parser
```

Run the complete chain:

```python
result = chain.invoke({
    "topic": "LCEL"
})

print(result)
```

Flow:

```text
{
    "topic": "LCEL"
}
        ↓
Prompt
        ↓
Formatted Messages
        ↓
Chat Model
        ↓
AIMessage
        ↓
StrOutputParser
        ↓
String
```

---

# 5. LCEL and Runnables

LCEL is built around the Runnable abstraction.

Many LangChain components are Runnables:

- Chat models
- Prompt templates
- Output parsers
- Retrievers
- Custom functions

Because they share a common interface, they can be composed.

```text
Runnable A
    ↓
Runnable B
    ↓
Runnable C
```

Using LCEL:

```python
runnable_a | runnable_b | runnable_c
```

---

# 6. `RunnableSequence`

A `RunnableSequence` represents a sequence of Runnables.

For example:

```python
chain = prompt | model | parser
```

Conceptually:

```text
Runnable 1
    ↓
Runnable 2
    ↓
Runnable 3
```

The pipe operator creates a sequential pipeline.

You can also think of:

```python
prompt | model | parser
```

as:

```text
RunnableSequence(
    Prompt,
    Model,
    Parser
)
```

The output from each step is passed to the next step.

---

# 7. RunnableSequence Flow

Consider:

```python
chain = prompt | model | parser
```

Execution:

```text
Step 1
Input
  ↓
Prompt
  ↓

Step 2
Formatted Prompt
  ↓
Model
  ↓

Step 3
AIMessage
  ↓
Parser
  ↓

Final Output
```

The key idea is:

> The output type of one step should be suitable as the input for the next step.

---

# 8. Invoking an LCEL Chain

An LCEL chain is also a Runnable.

Therefore, it supports the same methods:

```python
chain.invoke(input)
```

This is a major benefit of LCEL.

```text
Individual Runnables
        ↓
LCEL Composition
        ↓
New Runnable
        ↓
Same Runnable Interface
```

---

# 9. RunnableParallel

`RunnableParallel` allows multiple Runnables to receive the same input.

Example concept:

```text
                 Input
                   │
          ┌────────┴────────┐
          ↓                 ↓
      Runnable A        Runnable B
          ↓                 ↓
       Output A          Output B
          └────────┬────────┘
                   ↓
             Combined Output
```

In LCEL, a dictionary can be used for parallel execution.

```python
chain = {
    "summary": summary_chain,
    "keywords": keyword_chain
}
```

Both chains receive the same input.

The result is structured by the dictionary keys.

```python
result = chain.invoke("LangChain is a framework...")
```

Conceptually:

```python
{
    "summary": "...",
    "keywords": ["...", "..."]
}
```

---

# 10. RunnableParallel Example

```python
from langchain_core.runnables import RunnableParallel

parallel_chain = RunnableParallel(
    summary=summary_chain,
    keywords=keyword_chain
)
```

Then:

```python
result = parallel_chain.invoke(
    "LangChain helps developers build AI applications."
)
```

Possible flow:

```text
Input Text
     │
     ├──────────────→ Summary Chain
     │                     ↓
     │                  Summary
     │
     └──────────────→ Keyword Chain
                           ↓
                        Keywords

              ↓
        Combined Result
```

---

# 11. RunnableSequence vs RunnableParallel

### RunnableSequence

Steps execute as a pipeline.

```text
Input
  ↓
A
  ↓
B
  ↓
C
  ↓
Output
```

### RunnableParallel

Multiple operations receive the same input.

```text
             Input
           /   |   \
          ↓    ↓    ↓
          A    B    C
          ↓    ↓    ↓
          └────┼────┘
               ↓
             Output
```

| Feature | RunnableSequence | RunnableParallel |
|---|---|---|
| Execution pattern | Sequential | Parallel |
| Input | Output from previous step | Same input to branches |
| Typical use | Pipelines | Multiple independent tasks |

---

# 12. `RunnablePassthrough`

`RunnablePassthrough` passes the input forward without changing it.

```python
from langchain_core.runnables import RunnablePassthrough

runnable = RunnablePassthrough()

result = runnable.invoke("Hello")

print(result)
```

Output:

```text
Hello
```

Conceptually:

```text
Input
  ↓
RunnablePassthrough
  ↓
Same Input
```

---

# 13. Why Use RunnablePassthrough?

It is useful when you need to preserve the original input while performing other operations.

For example:

```text
User Question
      │
      ├───────────────→ Retriever
      │                     ↓
      │                  Context
      │
      └────────────────→ Original Question
                              ↓
                    Combined Input
```

A common pattern:

```python
chain = {
    "context": retriever,
    "question": RunnablePassthrough()
}
```

If the input is:

```text
What is LangChain?
```

The output conceptually becomes:

```python
{
    "context": "...retrieved documents...",
    "question": "What is LangChain?"
}
```

---

# 14. `RunnablePassthrough.assign()`

`assign()` can add new values to an existing dictionary.

Example concept:

```python
chain = RunnablePassthrough.assign(
    extra_data=some_runnable
)
```

Input:

```python
{
    "question": "What is LangChain?"
}
```

Output:

```python
{
    "question": "What is LangChain?",
    "extra_data": "..."
}
```

This is useful when gradually building structured data through a chain.

---

# 15. `RunnableLambda`

`RunnableLambda` converts a Python function into a Runnable.

Example:

```python
from langchain_core.runnables import RunnableLambda

def to_uppercase(text):
    return text.upper()

uppercase = RunnableLambda(to_uppercase)

result = uppercase.invoke("hello")

print(result)
```

Output:

```text
HELLO
```

Conceptually:

```text
Input
  ↓
Python Function
  ↓
RunnableLambda
  ↓
Output
```

---

# 16. Why Use RunnableLambda?

Sometimes you need custom Python logic inside an LCEL pipeline.

For example:

```python
def extract_question(data):
    return data["question"]
```

Convert it into a Runnable:

```python
extract = RunnableLambda(extract_question)
```

Then compose it:

```python
chain = extract | prompt | model | parser
```

Flow:

```text
Dictionary Input
       ↓
Custom Python Function
       ↓
Question
       ↓
Prompt
       ↓
Model
       ↓
Parser
```

---

# 17. RunnableLambda Example

```python
from langchain_core.runnables import RunnableLambda

get_length = RunnableLambda(
    lambda text: len(text)
)

result = get_length.invoke(
    "LangChain"
)

print(result)
```

Output:

```text
9
```

A normal Python function becomes part of an LCEL pipeline.

---

# 18. `RunnableBranch`

`RunnableBranch` allows conditional routing.

Different inputs can follow different branches.

Conceptually:

```text
                    Input
                      ↓
                 Condition
                 /       \
              True       False
                ↓           ↓
             Chain A     Chain B
                ↓           ↓
              Output     Output
```

Example use cases:

- Route technical questions differently
- Select different prompts
- Handle different input types
- Choose specialized models
- Create conditional workflows

---

# 19. RunnableBranch Example

```python
from langchain_core.runnables import RunnableBranch

branch = RunnableBranch(
    (
        lambda x: "python" in x.lower(),
        python_chain
    ),
    general_chain
)
```

Conceptually:

```text
Input
  ↓
Does input contain "python"?
       │
    ┌──┴──┐
   Yes    No
    ↓      ↓
Python   General
Chain     Chain
    ↓      ↓
    └──┬──┘
       ↓
     Output
```

The first matching condition determines the branch.

The final Runnable acts as the default branch.

---

# 20. Putting LCEL Components Together

LCEL components can be combined to create powerful workflows.

Example:

```text
                Input
                  ↓
          RunnableBranch
             /       \
            ↓         ↓
        Branch A    Branch B
            ↓         ↓
            └────┬────┘
                 ↓
        RunnableParallel
          /            \
         ↓              ↓
     Operation A     Operation B
         ↓              ↓
         └──────┬───────┘
                ↓
        RunnableSequence
                ↓
             Output
```

The important idea is that LCEL allows you to compose workflows from small reusable pieces.

---

# 21. LCEL Composition Patterns

### Sequential

```python
chain = prompt | model | parser
```

```text
A → B → C
```

### Parallel

```python
chain = {
    "result_a": chain_a,
    "result_b": chain_b
}
```

```text
    Input
   /     \
  A       B
   \     /
    Output
```

### Passthrough

```python
RunnablePassthrough()
```

```text
Input → Same Input
```

### Custom Function

```python
RunnableLambda(function)
```

```text
Input → Function → Output
```

### Conditional

```python
RunnableBranch(
    (condition, chain_a),
    chain_b
)
```

```text
Input → Condition → Appropriate Chain
```

---

# 22. Why LCEL is Powerful

LCEL provides:

### 1. Composition

```python
prompt | model | parser
```

### 2. Streaming

```python
for chunk in chain.stream(input):
    print(chunk)
```

### 3. Async Support

```python
result = await chain.ainvoke(input)
```

### 4. Batch Processing

```python
results = chain.batch(inputs)
```

### 5. Reusable Components

Small Runnables can be reused in different chains.

### 6. Flexible Workflows

You can create sequential, parallel, and conditional workflows.

---

# 23. Complete LCEL Example

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_template(
    "Explain {topic} in simple words."
)

parser = StrOutputParser()

chain = prompt | model | parser

result = chain.invoke({
    "topic": "LangChain Expression Language"
})

print(result)
```

Flow:

```text
Input
  ↓
{
    "topic": "LCEL"
}
  ↓
ChatPromptTemplate
  ↓
Formatted Messages
  ↓
Chat Model
  ↓
AIMessage
  ↓
StrOutputParser
  ↓
String
```