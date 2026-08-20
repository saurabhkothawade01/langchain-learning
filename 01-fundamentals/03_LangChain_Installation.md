# LangChain Installation

# 1. Prerequisites

You should have:

- Python installed
- `pip`
- A code editor such as VS Code
- An API key from an LLM provider

Check Python:

```bash
python --version
```

Check pip:

```bash
pip --version
```

---

# 2. Create a Folder

Create a folder for your LangChain learning:

```bash
mkdir langchain-learning
cd langchain-learning
```

---

# 3. Create a Virtual Environment

A virtual environment keeps project dependencies isolated.

Create it:

```bash
python -m venv .venv
```

---

# 4. Activate the Virtual Environment

## macOS / Linux

```bash
source .venv/bin/activate
```

## Windows CMD

```cmd
.venv\Scripts\activate
```

---

# 5. Upgrade pip

Run:

```bash
python -m pip install --upgrade pip
```

This ensures that your package installer is up to date.

---

# 6. Install LangChain

Install the main LangChain package:

```bash
pip install -U langchain
```

### What does `-U` mean?

`-U` means:

```text
--upgrade
```

So:

```bash
pip install -U langchain
```

means:

> Install LangChain and upgrade it to the latest available version if an older version is already installed.

---

# 7. Important: LangChain Uses Separate Integrations

Modern LangChain separates the core framework from many provider integrations.

For example:

```text
LangChain
   |
   +---- OpenAI integration
   |
   +---- Anthropic integration
   |
   +---- Google integration
   |
   +---- Groq integration
   |
   +---- OpenRouter / compatible integrations
```

This means you may install additional packages depending on the model provider you use.

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

Do not install every provider package unnecessarily.

Install the integration for the provider you actually use.

---

# 8. Using OpenRouter

Since OpenRouter can provide access to multiple models through one API, it is useful for experimentation.

Install:

```bash
pip install -U langchain-openrouter
```

---

# 9. Install Environment Variable Support

Install `python-dotenv`:

```bash
pip install -U python-dotenv
```

This allows your Python application to load values from a `.env` file.

---

# 10. Create a `.env` File

Create:

```text
.env
```

Example:

```env
OPENROUTER_API_KEY=your_api_key_here
```

Do not put your real API key directly inside Python source code.

Bad:

```python
api_key = "sk-xxxxxxxx"
```

Better:

```text
.env
    ↓
Environment Variable
    ↓
Python Application
```

---

# 11. Load Environment Variables

Python:

```python
import os
from dotenv import load_dotenv

load_dotenv()

api_key = os.getenv("OPENROUTER_API_KEY")

print(api_key)
```

For learning, printing the key can help verify configuration, but do not print API keys in real applications or logs.

A safer check:

```python
if api_key:
    print("API key loaded successfully")
else:
    print("API key not found")
```

---

# 12. Create `requirements.txt`

You can save project dependencies:

```text
langchain
langchain-openrouter
python-dotenv
```

Then install them using:

```bash
pip install -r requirements.txt
```

After installing everything, you can generate a pinned dependency list with:

```bash
pip freeze > requirements.txt
```

For learning projects, understand the difference between:

```text
Direct dependencies
```

and:

```text
All installed dependencies
```

`pip freeze` can include many transitive dependencies.

---

# 13. First LangChain Program

For an OpenRouter setup using the OpenAI model:

```python
from langchain_openrouter import ChatOpenRouter

model = ChatOpenRouter(
    model="openai/gpt-4o-mini"
)

response = model.invoke("Explain LangChain in one sentence.")

print(response.content)
```