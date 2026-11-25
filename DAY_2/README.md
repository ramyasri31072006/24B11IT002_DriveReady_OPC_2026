# TechFix Electronics Repair Center — Day 2 Learning

Day 2 of the AI Agents Workshop, exploring Python fundamentals and a conversational customer support chatbot using the OpenAI Python SDK with Groq. TechFix is a fictional business for learning purposes.

## Business rules

- Repair smartphones and laptops, including screen replacement, battery replacement and software troubleshooting.
- Diagnose every device before repair; actual costs depend on device condition.
- Operate Monday through Saturday.
- Never guarantee completion times or confirm exact repair costs before diagnosis.
- For unknown information, say “I'll check with the owner.”
- Respond in friendly English or Telugu, matching the customer, in under 50 words.

## Technologies

Python, Google Colab, Groq API, OpenAI Python SDK, `openai/gpt-oss-120b`, and Python's built-in `json` and `functools` modules.

## Setup in Google Colab

```python
!pip install -q openai
```

Add your Groq API key to Colab Secrets as `GROK_API_KEY` (the custom name used in our exercises), enable notebook access, and initialize the client:

```python
from google.colab import userdata
from openai import OpenAI

client = OpenAI(
    api_key=userdata.get("GROK_API_KEY"),
    base_url="https://api.groq.com/openai/v1"
)
```

Never print or commit your API key. Run the subsequent exercises in order.

## 1. Dictionaries and safe lookup

```python
SERVICES = {
    "screen replacement": 1500,
    "battery replacement": 1000,
    "software troubleshooting": 500
}

SERVICES["screen replacement"]
SERVICES.get("camera repair", "Service not available")

order = {
    "name": "Ravi",
    "device": "smartphone",
    "service": "screen replacement",
    "qty": 2
}

print(order["qty"] * SERVICES[order["service"]])  # 3000
```

All prices are fictional practice estimates, not verified business prices. A missing dictionary key raises `KeyError` with direct indexing; `.get()` provides a fallback.

## 2. JSON serialization

```python
import json
text = json.dumps(order)
back = json.loads(text)
print(text)
print(back)
```

`dumps()` converts a dictionary to a JSON string for storage or transmission; `loads()` converts it back to a Python object.

## 3. Functions, type hints and docstrings

```python
def get_price(service: str, qty: int = 1) -> int:
    """Return the estimated total repair price in rupees."""
    return SERVICES[service.lower().strip()] * qty

print(get_price("screen replacement", 2))  # 3000
print(get_price("Battery Replacement"))    # 1000
```

Type hints describe expected types but do not enforce them at runtime. The improved version discussed separately validates quantity and handles unknown services; the original function above can raise `KeyError`.

## 4. Decorators and logging

```python
import functools

def log_call(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        print(f"   [log] {func.__name__}{args} -> {result!r}")
        return result
    return wrapper

@log_call
def get_price(service: str, qty: int = 1) -> int:
    """Return the estimated total repair price in rupees."""
    return SERVICES[service.lower().strip()] * qty
```

The decorator logs successful calls and their results while `functools.wraps` preserves function metadata. Avoid logging sensitive customer information in production.

## 5. System prompt and conversational chatbot

```python
from openai import RateLimitError, APIConnectionError

MODEL = "openai/gpt-oss-120b"
SYSTEM = f"""ROLE: You are the friendly assistant for TechFix Electronics Repair Center.
FACTS: Estimated service prices in rupees: {json.dumps(SERVICES)}
       We repair smartphones and laptops.
       Every device requires diagnosis before repair.
       We operate Monday through Saturday.
RULES: Never promise a repair completion time.
       Never confirm exact repair costs without diagnosis.
       All listed prices are estimates only.
       If it isn't in FACTS, say "I'll check with the owner."
       Remember the customer's name, device and repair requirements during the chat.
FORMAT: Under 50 words, friendly, English or Telugu matching the customer."""

history = [{"role": "system", "content": SYSTEM}]
MAX_TURNS = 8

def chat(user_text: str) -> str:
    history.append({"role": "user", "content": user_text})
    recent = [history[0]] + history[1:][-MAX_TURNS:]
    try:
        response = client.chat.completions.create(
            model=MODEL, messages=recent, temperature=0.3
        )
        reply = response.choices[0].message.content
    except RateLimitError:
        reply = "We're a little busy. Please try again in a minute."
    except APIConnectionError:
        reply = "I can't reach the server right now. Please try again shortly."
    history.append({"role": "assistant", "content": reply})
    return reply

while True:
    text = input("You: ")
    if text.lower().strip() in ("bye", "exit", "quit"):
        print("Bot: Thank you for visiting TechFix!")
        break
    print("Bot:", chat(text))
```

The system prompt stays in context alongside the latest eight non-system messages, not eight full conversation turns. Older details may be forgotten, and the history list itself is not bounded. Memory is temporary and resets when the history is reinitialized.

## 6. Manual testing

| Test prompt | Expected behavior |
| --- | --- |
| Hi, my name is Ravi. | Acknowledge customer name |
| My smartphone screen is broken. | Explain repair and diagnosis |
| How much does it cost? | Give only a clearly labeled estimate |
| What is the cost for 2 screen replacements? | Estimated Rs 3000, not a confirmed quote |
| What is my name? | Recall Ravi if still in context |
| Can you repair it within 1 hour? | Do not guarantee completion |
| Are you open on Sunday? | Explain Monday–Saturday operation |
| What is your shop address? | Say the owner must be consulted |
| bye | Exit the chat loop |

These are expected behaviors, not measured test results. The live API and test pass rate have not been verified for these exercises.

## 7. Prompt Guard and next steps

Prompt Guard and a `secure_ask()` wrapper were also discussed as a way to screen suspicious user inputs before calling the main model. The discussed model was `meta-llama/llama-prompt-guard-2-22m`. Its current availability and response format have not been verified; validate both before integrating it. Do not assume it returns text labels rather than scores. Treat unexpected guard outputs conservatively.

The current chatbot does not register or invoke `get_price()` as an LLM tool. The model can calculate from the prompt, but this is not deterministic tool calling. Next steps include connecting the price function through tool calling, validating inputs, integrating and testing Prompt Guard, improving bounded memory, and adding CSV-based evaluation.

## Project status

This README documents the Day 2 exercises and discussed improvements. It does not claim that the separate project source files have been updated or that the proposed improvements are deployed.
