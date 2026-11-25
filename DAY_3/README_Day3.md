# Day 3 – TechFix Electronics Repair Center

## Project Overview

A customer support AI chatbot built with Python, LangChain and Groq. Day 3 focuses on prompt templates, conversation memory, automated CSV testing and creating a basic AI agent.

## Business Rules

- Repairs smartphones and laptops, including screen replacement, battery replacement and software troubleshooting.
- Every device requires diagnosis before repair; costs depend on device condition.
- Open Monday through Saturday.
- Never guarantee repair completion times or quote exact prices before diagnosis.
- For unknown information, respond with "I'll check with the owner."
- Keep responses friendly, under 50 words, in English or Telugu matching the customer.

## Technologies Used

- Python and Google Colab
- LangChain and LangChain Core
- Groq API and `openai/gpt-oss-120b`
- CSV-based testing

## Setup

Install dependencies in Colab:

```python
!pip install -U langchain langchain-groq
```

Add your Groq API key to Colab Secrets as `GROK_API_KEY` and enable notebook access.

```python
import os
from google.colab import userdata
from langchain_groq import ChatGroq

os.environ['GROQ_API_KEY'] = userdata.get('GROK_API_KEY')
llm = ChatGroq(model='openai/gpt-oss-120b', temperature=0.3)
```

## Prompt Engineering and Conversation Memory

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.output_parsers import StrOutputParser

SYSTEM = """ROLE: You are the friendly assistant for TechFix Electronics Repair Center.
FACTS: We repair smartphones and laptops. Services include screen replacement,
       battery replacement and software troubleshooting. Every device requires
       diagnosis before repair. Repair costs depend on device condition.
       We operate Monday through Saturday.
RULES: Never promise a repair completion time. Never quote exact prices
       without diagnosis. If it isn't in FACTS, say "I'll check with the owner."
FORMAT: Under 50 words, friendly, English or Telugu matching the customer."""

prompt = ChatPromptTemplate.from_messages([
    ('system', SYSTEM),
    MessagesPlaceholder('history'),
    ('human', '{question}'),
])
chain = prompt | llm | StrOutputParser()

history = []

def ask(question: str) -> str:
    answer = chain.invoke({'history': history, 'question': question})
    history.extend([('human', question), ('ai', answer)])
    return answer

print(ask("Hi, I'm Ravi. Do you repair smartphone screens?"))
print(ask("What's my name?"))
```

The history list stores customer questions and AI responses during the session.

## Automated CSV Testing

Upload the existing `test.csv` file to Colab. It must contain `question` and `must_include` columns.

```python
import csv

def run_tests(path: str = '/content/test.csv') -> None:
    with open(path, encoding='utf-8') as file:
        rows = list(csv.DictReader(file))
    passed = 0
    for row in rows:
        answer = chain.invoke({'history': [], 'question': row['question']})
        ok = row['must_include'].lower() in answer.lower()
        passed += ok
        print('PASS' if ok else 'FAIL', '|', row['question'], '→', answer[:70])
    if rows:
        print(f'\n{passed}/{len(rows)} passed ({passed / len(rows):.0%})')
    else:
        print('No test cases found.')

run_tests('/content/sample_data/test.csv')
```

Adjust the quoted file path to match the uploaded CSV. Each test starts with an empty history. The function reports keyword-based PASS/FAIL results and the pass percentage.

## Creating an AI Agent

```python
from langchain.agents import create_agent

agent_order = create_agent(model=llm, tools=[], system_prompt=SYSTEM)
result_order = agent_order.invoke({
    'messages': [{'role': 'user', 'content': 'Do you repair smartphone screens?'}]
})
print(result_order['messages'][-1].content)
```

This creates a basic agent using the same model and business instructions, with an empty tool list for Day 3.

## Workflow

```text
Customer question → Prompt template + history → Groq LLM → Text output
                                                         → Update history
CSV test cases → Chain → Keyword checks → PASS/FAIL report
Customer message → Basic LangChain agent → Final response
```

## Learning Outcomes

- Configure a Groq model using Colab Secrets.
- Build reusable prompts and LangChain pipelines.
- Maintain conversation context using MessagesPlaceholder.
- Evaluate chatbot responses using CSV test cases.
- Create and invoke a basic LangChain agent.
