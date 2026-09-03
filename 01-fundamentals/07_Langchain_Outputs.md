# LangChain Output

## 1. What is Output?

When we send input to a chat model, it generates a response.

```python
response = model.invoke("What is LangChain?")
```

A chat model typically returns an `AIMessage`.

```text
Input
  ↓
Chat Model
  ↓
AIMessage
```

The output may contain text, content blocks, tool calls, and metadata.

---

## 2. `AIMessage`

An `AIMessage` represents a response from an AI model.

```python
response = model.invoke("What is Python?")

print(type(response))
print(response.content)
```

Conceptually:

```text
AIMessage
 ├── content
 ├── content_blocks
 ├── tool_calls
 └── response_metadata
```

---

## 3. Basic Model Output

```python
response = model.invoke(
    "Explain LangChain in one sentence."
)

print(response.content)
```

Possible output:

```text
LangChain is a framework for building applications powered by language models.
```

---

## 4. `content`

The `content` attribute contains the primary content of the model response.

```python
response = model.invoke("What is LangChain?")

print(response.content)
```

For simple applications, this may be all you need.

---

## 5. `content_blocks`

LangChain also provides `content_blocks` as a standardized representation of message content.

```python
print(response.content_blocks)
```

Depending on the model and response, content blocks can represent:

- Text
- Reasoning
- Images
- Audio
- Files
- Tool calls

```text
AIMessage
    ↓
content_blocks
    ├── Text
    ├── Reasoning
    ├── Image
    ├── Audio
    ├── File
    └── Tool Call
```

---

# 6. Why Do We Need Output Parsers?

Model output is often designed for humans, but applications may need a specific format.

The model might return:

```text
Python, Java, JavaScript
```

But an application may need:

```python
[
    "Python",
    "Java",
    "JavaScript"
]
```

An output parser transforms model output into an application-friendly format.

```text
Model Output
     ↓
Output Parser
     ↓
Application-Friendly Output
```

---

# 7. Output Parsers

An output parser processes a model response and converts it into a desired format.

| Parser | Purpose |
|---|---|
| `StrOutputParser` | Returns text as a string |
| `JsonOutputParser` | Parses JSON output |
| `PydanticOutputParser` | Parses output into a Pydantic model |
| `CommaSeparatedListOutputParser` | Parses comma-separated values |
| `XMLOutputParser` | Parses XML |

---

# 8. `StrOutputParser`

`StrOutputParser` is one of the simplest output parsers.

```python
from langchain_core.output_parsers import StrOutputParser

parser = StrOutputParser()
```

Example:

```python
response = model.invoke("Tell me a joke.")

result = parser.invoke(response)

print(result)
```

```text
AIMessage
    ↓
StrOutputParser
    ↓
String
```

---

# 9. `StrOutputParser` in a Chain

Output parsers work well with LCEL composition.

```python
parser = StrOutputParser()

chain = prompt | model | parser
```

Then:

```python
result = chain.invoke({
    "topic": "LangChain"
})

print(result)
```

Flow:

```text
Input
  ↓
Prompt
  ↓
Model
  ↓
AIMessage
  ↓
StrOutputParser
  ↓
String
```

---

# 10. `JsonOutputParser`

Sometimes applications need JSON data.

Desired output:

```json
{
    "name": "John",
    "age": 25,
    "city": "Mumbai"
}
```

LangChain provides:

```python
from langchain_core.output_parsers import JsonOutputParser

parser = JsonOutputParser()
```

Conceptually:

```text
Model Output
    ↓
JsonOutputParser
    ↓
Python Dictionary / List
```

---

# 11. `PydanticOutputParser`

Pydantic can be used when you want structured and validated data.

First, define a schema:

```python
from pydantic import BaseModel

class Person(BaseModel):
    name: str
    age: int
    city: str
```

Then create the parser:

```python
from langchain_core.output_parsers import PydanticOutputParser

parser = PydanticOutputParser(
    pydantic_object=Person
)
```

```text
Model Output
    ↓
PydanticOutputParser
    ↓
Validated Person Object
```

---

# 12. Why Structured Output?

Free-form text is easy for humans to understand.

```text
John is 25 years old and lives in Mumbai.
```

But applications often need predictable fields:

```python
{
    "name": "John",
    "age": 25,
    "city": "Mumbai"
}
```

Structured output is useful for:

- Information extraction
- Classification
- API responses
- Database operations
- UI rendering
- Automation

---

# 13. Structured Output

Structured output means defining the format that the model should return.

Example:

```python
from pydantic import BaseModel

class Person(BaseModel):
    name: str
    age: int
    city: str
```

```text
User Input
    ↓
Model
    ↓
Structured Response
    ↓
Python Object
```

---

# 14. `with_structured_output()`

Many LangChain chat models support structured output.

```python
from pydantic import BaseModel, Field

class Movie(BaseModel):
    title: str = Field(description="Movie title")
    year: int = Field(description="Release year")
    rating: float = Field(description="Rating out of 10")

structured_model = model.with_structured_output(Movie)
```

Then:

```python
result = structured_model.invoke(
    "Inception was released in 2010 and has a rating of 8.8."
)

print(result)
```

Possible result:

```python
Movie(
    title="Inception",
    year=2010,
    rating=8.8
)
```

```text
Model
  ↓
with_structured_output(Schema)
  ↓
Structured Result
```

---

# 15. Schema Types

## Pydantic

```python
class Movie(BaseModel):
    title: str
    year: int
```

Advantages:

- Type validation
- Field descriptions
- Nested models

## TypedDict

```python
from typing_extensions import TypedDict

class Movie(TypedDict):
    title: str
    year: int
```

## JSON Schema

```python
schema = {
    "type": "object",
    "properties": {
        "title": {"type": "string"},
        "year": {"type": "integer"}
    },
    "required": ["title", "year"]
}
```

---

# 16. Structured Output vs Output Parser

## Output Parser

The model generates a response first, and the parser processes it.

```text
Model
 ↓
Raw Output
 ↓
Parser
 ↓
Structured Data
```

```python
chain = prompt | model | parser
```

## Structured Output

The model is configured to produce data according to a schema.

```text
Model
 ↓
Schema
 ↓
Structured Data
```

```python
structured_model = model.with_structured_output(Movie)
```

### Modern Recommendation

When a model supports native structured output, using structured output directly is often preferable.

Output parsers are still useful when:

- A model does not support native structured output
- Custom transformations are needed
- Additional parsing is required

---

# 17. Streaming Output

Models can return output gradually using streaming.

```python
for chunk in model.stream("Tell me a story"):
    print(chunk)
```

```text
Model
  ↓
AIMessageChunk
  ↓
AIMessageChunk
  ↓
AIMessageChunk
```

Streaming is useful when you want to display output progressively.

---

# 18. Output Parsing Errors

A parser may fail if the model response does not match the expected format.

Expected:

```json
{
    "name": "John"
}
```

Actual:

```text
The person's name is John.
```

```text
Model
 ↓
Unexpected Format
 ↓
Parser
 ↓
Parsing Error
```

To reduce errors:

1. Clearly describe the desired format.
2. Use structured output when supported.
3. Validate important data.
4. Use appropriate schemas.

---

# 19. Complete String Output Example

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful teacher."),
    ("human", "Explain {topic} in simple words.")
])

parser = StrOutputParser()

chain = prompt | model | parser

result = chain.invoke({
    "topic": "LangChain"
})

print(result)
```

```text
Input
  ↓
ChatPromptTemplate
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

# 20. Complete Structured Output Example

```python
from pydantic import BaseModel, Field

class Person(BaseModel):
    name: str = Field(description="Person's name")
    age: int = Field(description="Person's age")
    city: str = Field(description="Person's city")

structured_model = model.with_structured_output(Person)

result = structured_model.invoke(
    "John is 25 years old and lives in Mumbai."
)

print(result)
```

Possible result:

```python
Person(
    name="John",
    age=25,
    city="Mumbai"
)
```

```text
User Input
    ↓
Chat Model
    ↓
Structured Output Schema
    ↓
Person Object
```