# Plan for Granny Smith AI Agent with Recipe API Integration

## Table of Contents

1. [Introduction](#introduction)
2. [Overview](#overview)
3. [OpenAI Tool Spec for Recipe API](#openai-tool-spec-for-recipe-api)
4. [SWAIG Function Schema](#swaig-function-schema)
   - [Function: `search_recipes`](#function-search_recipes)
5. [Example API Call in SWAIG Format](#example-api-call-in-swaig-format)
6. [Python Code for Granny Smith AI Agent](#python-code-for-granny-smith-ai-agent)
7. [System Prompt](#system-prompt)

---

## 1. Introduction

This document outlines the design for **Granny Smith**, an AI assistant specializing in recipes. Granny Smith helps users discover delicious recipes using the **Recipe API** from [API Ninjas](https://api.api-ninjas.com/v1/recipe).

---

## 2. Overview

- **Purpose**: Provide users with recipe suggestions based on their queries.
- **Integration**: Utilizes the Recipe API to fetch recipes.
- **Functionality**:
  - Search for recipes using a query.
  - Support pagination using the `offset` parameter.
- **Implementation**:
  - Flask application using SWAIG for API integration.
  - Defines functions according to the OpenAI Tool Spec schema.

---

## 3. OpenAI Tool Spec for Recipe API

### Function: `search_recipes`

- **Description**: Fetches a list of recipes matching the user's search query.
- **Parameters**:

  ```json
  {
    "type": "object",
    "properties": {
      "query": {
        "type": "string",
        "description": "Query text to search for recipes."
      },
      "offset": {
        "type": "integer",
        "description": "Number of results to offset for pagination."
      }
    },
    "required": ["query"]
  }
  ```

- **Headers**:

  - `X-Api-Key` (required): API Key associated with the account.

- **Endpoint**:

  - `GET https://api.api-ninjas.com/v1/recipe`

---

## 4. SWAIG Function Schema

### Function: `search_recipes`

- **Purpose**: Search for recipes based on a query.
- **Schema**:

  ```json
  {
    "function": "search_recipes",
    "description": "Search for recipes based on a query.",
    "parameters": {
      "type": "object",
      "properties": {
        "query": {
          "type": "string",
          "description": "Query text to search for recipes."
        },
        "offset": {
          "type": "integer",
          "description": "Number of results to offset for pagination."
        }
      },
      "required": ["query"]
    }
  }
  ```

---

## 5. Example API Call in SWAIG Format

```json
{
  "ai_session_id": "example-session-id",
  "app_name": "granny_smith_app",
  "argument": {
    "parsed": [
      {
        "query": "apple pie",
        "offset": 0
      }
    ],
    "raw": "{\"query\":\"apple pie\",\"offset\":0}",
    "substituted": ""
  },
  "parameters": {
    "type": "object",
    "properties": {
      "query": {
        "type": "string",
        "description": "Query text to search for recipes."
      },
      "offset": {
        "type": "integer",
        "description": "Number of results to offset for pagination."
      }
    },
    "required": ["query"]
  },
  "function": "search_recipes",
  "description": "Search for recipes based on a query.",
  "content_type": "text/swaig",
  "meta_data_token": "example-meta-data-token"
}
```

---

## 6. Python Code for Granny Smith AI Agent

Below is the Python code implementing the **Granny Smith** AI agent using Flask and SWAIG, adjusted to match your requirements.

```python
from flask import Flask
from dotenv import load_dotenv
import os
import requests
import random
import logging
from signalwire_swaig.core import SWAIG, SWAIGArgument

# Load environment variables from a .env file
load_dotenv()

# Set logging level for Flask's built-in server
logging.getLogger('werkzeug').setLevel(logging.WARNING)

# Enable debug mode if specified in environment variables
if os.environ.get('DEBUG', False):
    print("Debug mode is enabled")
    debug_pin = f"{random.randint(100, 999)}-{random.randint(100, 999)}-{random.randint(100, 999)}"
    os.environ['WERKZEUG_DEBUG_PIN'] = debug_pin
    logging.getLogger('werkzeug').setLevel(logging.DEBUG)
    print(f"Debugger PIN: {debug_pin}")

# Initialize Flask and SWAIG
app = Flask(__name__)
swaig = SWAIG(
    app,
    auth=(os.getenv('HTTP_USERNAME'), os.getenv('HTTP_PASSWORD'))
)

# Retrieve API key from environment variables
API_KEY = os.getenv('API_NINJAS_KEY')

# Function to fetch recipes from the Recipe API
def fetch_recipes(query, offset=0):
    url = 'https://api.api-ninjas.com/v1/recipe'
    headers = {
        'X-Api-Key': API_KEY
    }
    params = {
        'query': query,
        'offset': offset
    }
    response = requests.get(url, headers=headers, params=params)
    if response.status_code == 200:
        return response.json()
    else:
        return {"error": "Failed to fetch recipes", "status_code": response.status_code}

# SWAIG endpoint to search for recipes
@swaig.endpoint("Search for recipes",
    query=SWAIGArgument("string", "Query text to search for recipes.", required=True),
    offset=SWAIGArgument("integer", "Number of results to offset for pagination.", default=0, required=False))
def search_recipes(query, offset=0, meta_data_token=None, meta_data=None):
    recipes = fetch_recipes(query, offset)
    if "error" in recipes:
        return recipes, {}  # Return the error response with an empty dictionary
    else:
        # Format the recipes into a human-readable text blob
        formatted_recipes = ["Here are some recipes I found for you:"]
        for recipe in recipes:
            title = recipe.get('title', 'No title')
            ingredients = recipe.get('ingredients', 'No ingredients')
            instructions = recipe.get('instructions', 'No instructions')
            formatted_recipe = f"Title: {title}\nIngredients: {ingredients}\nInstructions: {instructions}\n"
            formatted_recipes.append(formatted_recipe)
        return "\n\n".join(formatted_recipes), {}  # Return the formatted recipes as a string and an empty dictionary

# Run the Flask application
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=os.getenv('PORT', 5000), debug=os.getenv('DEBUG')) 
```

**Notes**:

- **Return Statement Adjustment**:

  - The `search_recipes` function now returns `"\n\n".join(formatted_recipes), {}` to match your requirement.

    ```python
    return "\n\n".join(formatted_recipes), {}
    ```

- **Default Offset Value**:

  - Set the default value of `offset` to `0` to retrieve the first page of results.

- **Formatting the Response**:

  - The recipes are formatted into a string with titles, ingredients, and instructions, suitable for the AI assistant to present to the user.

---

## 7. System Prompt

```
You are Granny Smith, a friendly AI assistant and an expert in recipes from all cuisines. You help users find delicious recipes based on their search queries. When a user asks for a recipe, you use the Recipe API to find matching recipes and present them in a warm and engaging manner.

Your responses should be in first person singular, as if you are speaking directly to the user. Provide clear and concise recipe suggestions, including the title, a brief list of ingredients, and a summary of the instructions. Offer additional assistance if needed, such as alternative recipes or cooking tips.

Remember to be cheerful and personable, embodying the helpful nature of Granny Smith.
```

---

# Full Draft Plan and Outline

The goal is to create **Granny Smith**, an AI agent specializing in providing recipe suggestions to users by integrating the **Recipe API** from API Ninjas. The agent will be built using Flask and SWAIG, adhering to the OpenAI Tool Spec schema.

**Key Components**:

1. **API Integration**:
   - Utilize the Recipe API to fetch recipes based on user queries.
   - Handle API authentication using the `X-Api-Key` header.

2. **Function Definitions**:
   - Define the `search_recipes` function according to the OpenAI Tool Spec.
   - Ensure parameters match the API requirements.

3. **SWAIG Implementation**:
   - Use SWAIG to create API endpoints that the AI agent can interact with.
   - Implement authentication for secure access.

4. **AI Agent Behavior**:
   - Craft a system prompt that guides the AI assistant to respond as Granny Smith.
   - Ensure responses are friendly, helpful, and align with the persona.

5. **Error Handling**:
   - Implement error handling for API failures.
   - Provide meaningful messages to the user in case of issues.

6. **Deployment Considerations**:
   - Set up environment variables for sensitive information.
   - Configure logging and debug settings appropriately.

**Steps to Implement**:

- **Step 1**: Set up the Flask application and SWAIG instance.
- **Step 2**: Define the `search_recipes` function with the required parameters.
- **Step 3**: Implement the `fetch_recipes` helper function to interact with the Recipe API.
- **Step 4**: Handle API responses and format them for the AI assistant, ensuring the return statements match the required format.
- **Step 5**: Write the system prompt to define Granny Smith's behavior.
- **Step 6**: Test the application to ensure it works as expected.
- **Step 7**: Document the API and functions according to the OpenAI Tool Spec.

---
