---
layout: post
title: Yet Another Way to Play with LLM; Chat Template
categories: [LLM]
---

In earlier posts, we used [meta-llama/Llama-3.2-1B-Instruct](https://huggingface.co/meta-llama/Llama-3.2-1B-Instruct/tree/main) as our base model—a model specifically trained for tooling. Let's now take a closer look at the mechanism that enables function calling.

This is a short note about prompt templates, serving as a prelude to my next post on mechanistic interpretation.

# How function calling relates to chat templates

The template acts like a blueprint, telling the model how to present system prompts, user messages, and available tools. Without the template, the model would have no consistent way to know when and how to call a function.

# Chat Templates

Hugging Face introduced [chat templates](https://huggingface.co/blog/chat-templates). Chat templates are written in Jinja and allow you to write Python-like code. Let's see what template our basic model has.

```python

from transformers import AutoTokenizer

model_id = "meta-llama/Llama-3.2-1B-Instruct"
tok = AutoTokenizer.from_pretrained(model_id)
print(tok.chat_template)

```

Let's try to apply this template to a simple prompt.

```python
messages = [
{"role": "system", "content": "You are a friendly chatbot"},
{"role":"user","content":"Hello!"}
]

prompt = tok.apply_chat_template(
    messages,
    add_generation_prompt=True,
    tokenize=False
)
print (prompt)
```

The output is:

```bash
<|begin_of_text|><|start_header_id|>system<|end_header_id|>

Cutting Knowledge Date: December 2023
Today Date: 25 Aug 2025

You are a friendly chatbot<|eot_id|><|start_header_id|>user<|end_header_id|>

Hello!<|eot_id|><|start_header_id|>assistant<|end_header_id|>
```

Our template has instructions to add information about the current date and the model’s knowledge cutoff date.
Our template has the following instructions:

```jinja
{% raw %}
{%- if not date_string is defined %}
    {%- if strftime_now is defined %} 
        {%- set date_string = strftime_now("%d %b %Y") %}
    {%- else %}
        {%- set date_string = "26 Jul 2024" %}
    {%- endif %}                                                                                
{%- endif %}

...

{{- "Cutting Knowledge Date: December 2023\n" }}                
{{- "Today Date: " + date_string + "\n\n" }}
{% endraw %}
```

The template also allows users to set the current date. 

```python

prompt = tok.apply_chat_template(
    messages,
    add_generation_prompt=True,
    tokenize=False,
    date_string="01 Januar 2042"
)
print (prompt)
```

```bash
>>>
<|begin_of_text|><|start_header_id|>system<|end_header_id|>

Cutting Knowledge Date: December 2023
Today Date: 01 Januar 2042

You are a friendly chatbot<|eot_id|><|start_header_id|>user<|end_header_id|>

Hello!<|eot_id|><|start_header_id|>assistant<|end_header_id|>

```

The template also says that we can use tool parameter to specify functions available for calling.

```jinja
{% raw %}
{%- if tools is not none and not tools_in_user_message %}                                                                                                                 
    {{- "You have access to the following functions. To call a function, please respond with JSON for a function call." }}
    {{- 'Respond in the format {"name": function name, "parameters": dictionary of argument name and its value}.' }}
    {{- "Do not use variables.\n\n" }}                                               
    {%- for t in tools %}                                                            
        {{- t | tojson(indent=4) }}                                                  
        {{- "\n\n" }}                                                                
    {%- endfor %}
{%- endif %}
{% endraw %}
```

Now let’s try passing a tool name into the prompt

```python
def get_weather(city: str) -> str:
"""
Get the current weather for a given city.
Args:
    city: The city name, e.g., "Berlin".
"""
return "Sunny, 28°C"

messages = [
{"role":"system","content":"You are a helpful assistant with tool-calling capabilities."},
{"role":"user","content":"What's the weather in London? Use tools."},
]

prompt = tok.apply_chat_template(
messages,
tools=[get_weather],
add_generation_prompt=True,
tokenize=False
)
```

And this is what the prompt turns out to be:

```bash
<|begin_of_text|><|start_header_id|>system<|end_header_id|>

Environment: ipython
Cutting Knowledge Date: December 2023
Today Date: 30 Aug 2025

You are a helpful assistant with tool-calling capabilities.<|eot_id|><|start_header_id|>user<|end_header_id|>

Given the following functions, please respond with a JSON for a function call with its proper arguments that best answers the given prompt.

Respond in the format {"name": function name, "parameters": dictionary of argument name and its value}.Do not use variables.

{
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "Get the current weather for a given city.",
        "parameters": {
            "type": "object",
            "properties": {
                "city": {
                    "type": "string",
                    "description": "The city name, e.g., \"Berlin\"."
                }
            },
            "required": [
                "city"
            ]
        },
        "return": {
            "type": "string"
        }
    }
}

What's the weather in London? Use tools.<|eot_id|><|start_header_id|>assistant<|end_header_id|>
```

And that was the model generates when we pass the prompt to it:

```bash

Environment: ipython
Cutting Knowledge Date: December 2023
Today Date: 30 Aug 2025

You are a helpful assistant with tool-calling capabilities.user

Given the following functions, please respond with a JSON for a function call with its proper arguments that best answers the given prompt.

Respond in the format {"name": function name, "parameters": dictionary of argument name and its value}.Do not use variables.

{
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "Get the current weather for a given city.",
        "parameters": {
            "type": "object",
            "properties": {
                "city": {
                    "type": "string",
                    "description": "The city name, e.g., \"Berlin\"."
                }
            },
            "required": [
                "city"
            ]
        },
        "return": {
            "type": "string"
        }
    }
}

What's the weather in London? Use tools.assistant

{"type": "function", "function": "get_weather", "parameters": {"city": "London"}}

```

So the model produces the correct function name and parameter schema. In the next post, we’ll explore the mechanism that makes this possible.
