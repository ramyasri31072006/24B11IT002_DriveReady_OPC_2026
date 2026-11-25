# Day 1 – TechFix Solutions AI Chatbot

## Project Overview

A beginner-friendly AI chatbot for TechFix Solutions, a smartphone and laptop repair service center. The project uses Python, LangChain and Groq to answer customer questions according to defined business facts and rules.

## Business Knowledge

- Smartphone and laptop repair
- Screen replacement, battery replacement and software troubleshooting
- Device diagnosis required before repair
- Repair costs depend on device condition
- Open Monday through Saturday

The assistant must not guarantee repair completion times, quote exact prices before diagnosis, or invent business policies. Responses should be polite, professional and under 60 words.

## Technologies Used

| Technology | Purpose |
|---|---|
| Python | Application logic |
| Google Colab | Development environment and secrets |
| LangChain (`langchain_groq`) | LLM integration |
| Groq API | Model access |
| `openai/gpt-oss-120b` | Customer question answering |
| `meta-llama/llama-prompt-guard-2-22m` | Prompt safety experiment |

## Setup and Execution

1. Open a Google Colab notebook and copy the code from `day_1.py` into notebook cells.
2. Install the dependency:

   ```python
   !pip install langchain_groq
   ```

3. Add your Groq API key to Colab Secrets under the exact name `GROK_API_KEY` and enable notebook access. Never commit the key to GitHub.
4. Initialize the model:

   ```python
   import os
   from google.colab import userdata
   from langchain_groq import ChatGroq

   api_key = userdata.get("GROK_API_KEY")
   os.environ["GROQ_API_KEY"] = api_key
   llm = ChatGroq(model="openai/gpt-oss-120b", temperature=0)
   ```

5. Test connectivity with `llm.invoke("Hello!")`.
6. Run the original `SYSTEM` prompt and `ask()` function cells, followed by the sample questions.

The exported file contains Colab-specific `!pip` syntax and is not directly executable as a standard Python script without adaptation.

## Prompt Engineering

The system prompt defines the assistant's role, business facts, rules and response format. The `ask()` function sends the system instructions and customer question to the model:

```python
def ask(question):
    return llm.invoke([
        ("system", SYSTEM),
        ("human", question)
    ]).content
```

## Architecture

```text
Customer question
       |
       v
ask(question) + SYSTEM prompt
       |
       v
LangChain ChatGroq -> Groq LLM
       |
       v
Customer response
```

## Test Cases

| Question | Expected behavior |
|---|---|
| My phone screen is broken. Can you fix it? | Explain screen replacement and diagnosis requirement |
| How much does laptop repair cost? | Explain that cost depends on diagnosis and device condition |
| Can you repair my phone in 2 hours? | Avoid guaranteeing completion time |
| Are you open on Sunday? | Explain Monday–Saturday operating schedule |

These are expected behaviors, not verified API results. The source contains four printed test calls but no automated evaluation or measured pass rate.

## Prompt Guard Experiment

The source also defines `check_prompt(question)`, which requests the `meta-llama/llama-prompt-guard-2-22m` model through `client.chat.completions.create()` and returns the response content.


## Repository Structure

```text
TechFix-Solutions/
├── day_1.py
└── README_Day1.md
```

## Learning Outcomes

- Configure an LLM securely using Colab Secrets.
- Integrate Groq with LangChain.
- Write business-specific system prompts.
- Use a Python function to generate customer responses.
- Test business constraints using sample questions.
- Explore prompt safety with a separate Prompt Guard function.
