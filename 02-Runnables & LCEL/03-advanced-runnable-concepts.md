# Advanced Runnable Concepts

After learning the Runnable abstraction and LCEL, the next step is learning how to control and configure Runnable execution.

---

# 1. Overview

A simple Runnable pipeline looks like this:

```python
chain = prompt | model | parser
```

But real applications often need additional capabilities:

- What input does a Runnable expect?
- What output does it produce?
- How can execution be configured?
- How can a Runnable be identified during tracing?
- How can concurrency be controlled?
- What happens when an operation fails?
- Can we provide a backup Runnable?

Advanced Runnable concepts help answer these questions.

```text
                    Runnable
                       │
       ┌───────────────┼────────────────┐
       ↓               ↓                ↓
    Schemas         Config          Reliability
       │               │                │
 Input / Output    Tags / Metadata   Retries / Fallbacks
                       │
                       ↓
               Runtime Configuration
                       │
                       ↓
                  Concurrency
```

---

# 2. Input Schema

An input schema describes the type or structure of data a Runnable expects.

Conceptually:

```text
Expected Input
      ↓
Input Schema
      ↓
Runnable
```

For example, a chain may expect:

```python
{
    "topic": "LangChain"
}
```

The schema helps us understand what should be provided to the Runnable.

A Runnable can expose an input schema using:

```python
schema = chain.get_input_schema()
```

Conceptually:

```text
Runnable
   ↓
get_input_schema()
   ↓
Input Schema
```

---

# 3. Output Schema

An output schema describes the type or structure produced by a Runnable.

```text
Runnable
   ↓
Output Schema
   ↓
Expected Output
```

You can inspect it with:

```python
schema = chain.get_output_schema()
```

For example:

```text
Prompt
   ↓
Model
   ↓
AIMessage
   ↓
Output Parser
   ↓
String
```

The final output schema depends on the final Runnable in the chain.

---

# 4. Why Schemas Matter

Schemas make Runnable workflows easier to understand.

They help with:

- Validation
- Debugging
- Documentation
- Tooling
- Type-aware applications

Example:

```text
Input Schema
      ↓
Valid Input?
      ↓
Runnable
      ↓
Output Schema
```

When building larger workflows, understanding the input and output of every component becomes important.

---

# 5. Runnable Configuration

Runnables can accept configuration during execution.

A common pattern is:

```python
result = runnable.invoke(
    input,
    config={
        ...
    }
)
```

The configuration provides information about how the Runnable should execute.

This can include:

- Tags
- Metadata
- Concurrency settings
- Runtime configuration

Conceptually:

```text
                 Config
                   │
                   ↓
Input ───────→ Runnable ───────→ Output
```

The input contains application data.

The configuration controls execution behavior.

---

# 6. `RunnableConfig`

LangChain uses the concept of `RunnableConfig` for execution configuration.

A conceptual example:

```python
config = {
    "tags": ["learning"],
    "metadata": {
        "project": "langchain-learning"
    }
}

result = chain.invoke(
    input_data,
    config=config
)
```

The exact configuration options depend on the Runnable and the use case.

---

# 7. Tags

Tags are labels attached to a Runnable execution.

Example:

```python
result = chain.invoke(
    input_data,
    config={
        "tags": ["production", "summarization"]
    }
)
```

Conceptually:

```text
Chain Execution
      │
      ├── production
      └── summarization
```

Tags are useful for:

- Organizing traces
- Filtering executions
- Debugging workflows
- Identifying workflow categories

Tags describe the execution.

They are not normally part of the model input.

---

# 8. Metadata

Metadata stores additional information about a Runnable execution.

Example:

```python
result = chain.invoke(
    input_data,
    config={
        "metadata": {
            "user_type": "student",
            "feature": "tutorial"
        }
    }
)
```

Conceptually:

```text
Runnable Execution
        │
        ├── Tags
        │     ├── production
        │     └── tutorial
        │
        └── Metadata
              ├── user_type: student
              └── feature: tutorial
```

Metadata is useful for attaching contextual information to executions.

---

# 9. Tags vs Metadata

| Feature | Tags | Metadata |
|---|---|---|
| Structure | Simple labels | Key-value data |
| Example | `"production"` | `{"project": "demo"}` |
| Primary use | Categorization | Additional context |

Example:

```python
config = {
    "tags": [
        "learning",
        "development"
    ],
    "metadata": {
        "project": "langchain-learning",
        "phase": 2
    }
}
```

A simple mental model:

```text
Tags     → Labels

Metadata → Details
```

---

# 10. Configuration with `with_config()`

Instead of passing configuration on every invocation, configuration can be attached to a Runnable.

Conceptually:

```python
configured_chain = chain.with_config(
    tags=["learning"]
)
```

Then:

```python
result = configured_chain.invoke(input_data)
```

Flow:

```text
Original Runnable
       ↓
with_config()
       ↓
Configured Runnable
       ↓
invoke()
```

This is useful when the same configuration should be reused.

---

# 11. Concurrency

Concurrency controls how multiple Runnable executions are handled.

Consider:

```python
inputs = [
    "Input 1",
    "Input 2",
    "Input 3",
    "Input 4"
]
```

A batch operation may process multiple independent tasks.

```text
Input 1 ──┐
Input 2 ──┤
Input 3 ──┼──→ Runnable Execution
Input 4 ──┘
```

The number of operations running simultaneously can be controlled.

---

# 12. `max_concurrency`

A configuration can limit the maximum number of concurrent operations.

Conceptually:

```python
results = chain.batch(
    inputs,
    config={
        "max_concurrency": 2
    }
)
```

This means the workflow should not attempt to run all inputs simultaneously.

Example:

```text
Inputs:  1  2  3  4  5

Maximum Concurrency = 2

Time 1 → [1] [2]
Time 2 → [3] [4]
Time 3 → [5]
```

Concurrency limits can help:

- Avoid overwhelming external services
- Respect resource limits
- Control API usage
- Manage system load

---

# 13. Retries

External operations can fail.

For example:

```text
Runnable
   ↓
Network Error
```

A retry mechanism can attempt the operation again.

```text
Attempt 1
   ↓
Failure
   ↓
Retry
   ↓
Attempt 2
   ↓
Success
```

LangChain Runnables can be configured with retry behavior using `with_retry()`.

Conceptually:

```python
reliable_chain = chain.with_retry()
```

Then:

```python
result = reliable_chain.invoke(input_data)
```

---

# 14. Why Use Retries?

Retries can be useful for temporary or transient failures.

Examples include:

- Temporary network problems
- Service availability issues
- Rate-related failures, when appropriate
- Other retryable exceptions

However, retries should not be used blindly.

```text
Failure
   ↓
Is the error temporary?
   │
   ├── Yes → Retry may help
   │
   └── No  → Retrying may not help
```

The goal is to improve resilience against appropriate failures.

---

# 15. Retry Configuration

A conceptual example:

```python
chain = chain.with_retry(
    stop_after_attempt=3
)
```

Conceptually:

```text
Attempt 1 → Fail
      ↓
Attempt 2 → Fail
      ↓
Attempt 3 → Success / Final Failure
```

The available retry behavior should be selected according to the needs of the application.

---

# 16. Fallbacks

A fallback provides an alternative Runnable when the primary Runnable fails.

Conceptually:

```text
             Input
               ↓
        Primary Runnable
               │
          ┌────┴────┐
       Success     Failure
          │           ↓
          │      Fallback Runnable
          │           ↓
          └─────→ Output
```

LangChain Runnables can use:

```python
with_fallbacks()
```

A conceptual example:

```python
chain_with_fallback = primary_chain.with_fallbacks(
    [fallback_chain]
)
```

---

# 17. Why Use Fallbacks?

Fallbacks can improve application reliability.

Example:

```text
Primary Model
     │
     ├── Success → Output
     │
     └── Failure
           ↓
      Backup Model
           ↓
         Output
```

Possible fallback strategies:

- Primary model → Backup model
- Primary API → Alternative API
- Complex chain → Simpler chain
- Primary retriever → Alternative retriever

---

# 18. Retries vs Fallbacks

Retries and fallbacks solve different problems.

### Retry

Try the same operation again.

```text
Runnable
   ↓
Failure
   ↓
Same Runnable
   ↓
Retry
```

### Fallback

Use another Runnable.

```text
Primary Runnable
       ↓
    Failure
       ↓
Fallback Runnable
```

| Feature | Retry | Fallback |
|---|---|---|
| Action | Try again | Use an alternative |
| Component | Same Runnable | Different Runnable |
| Best for | Temporary failures | Alternative execution paths |

---

# 19. Combining Reliability Strategies

Retries and fallbacks can be combined.

Conceptually:

```text
Primary Runnable
       ↓
    Attempt
       ↓
    Failure
       ↓
     Retry
       ↓
Still Fails?
       │
      Yes
       ↓
Fallback Runnable
       ↓
     Output
```

This creates a more resilient workflow.

---

# 20. Advanced Runnable Example

Consider a summarization workflow:

```python
chain = prompt | model | parser
```

Add configuration:

```python
configured_chain = chain.with_config(
    tags=["summarization"],
    metadata={
        "project": "langchain-learning"
    }
)
```

Add retries:

```python
reliable_chain = configured_chain.with_retry(
    stop_after_attempt=3
)
```

Conceptually:

```text
Input
  ↓
Configured Chain
  ├── Tags
  └── Metadata
  ↓
Retry Logic
  ↓
Prompt
  ↓
Model
  ↓
Parser
  ↓
Output
```
