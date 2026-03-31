`cd mockchat`

Make sure you're at the **root** (not inside be or fe) so agents can see both folders.

---

## Step 2: Use This Prompt Format For End-to-End Features

Inside Claude Code, use this pattern:
```
Build a [feature name] end-to-end.

Backend (mockchat-be - Laravel):
- Migration for [table]
- Model with relationships
- Controller with CRUD methods
- API routes in api.php

Frontend (mockchat-fe - Vue 3):
- API service file
- Pinia store
- Vue component
- Connect to backend API
```

---

## Step 3: Real Example For Mockchat

Since mockchat is a **customer service/sales training app**, here's a real prompt you can use right now:
```
Build a "scenarios" feature end-to-end for mockchat.

Backend (mockchat-be - Laravel):
- Migration: scenarios table (id, title, description, difficulty, created_at, updated_at)
- Model: Scenario with fillable fields
- Controller: ScenarioController with index, store, show, update, destroy
- API Resource: ScenarioResource
- Routes: RESTful routes in api.php

Frontend (mockchat-fe - Vue 3):
- scenario.service.js for API calls
- useScenarioStore (Pinia) for state management
- ScenarioList.vue component
- ScenarioForm.vue component

### SAMPLE 
I'm working on mockchat — a customer service/sales training app.
Stack: Laravel (mockchat-be) + Vue 3 with Pinia (mockchat-fe).

Build these 3 features end-to-end (backend + frontend):

---

FEATURE 1: More Customer Personality Types
Currently there are only 3 types: interested, irate, want to return.
- Add at least 10 more personality types (e.g. confused, impatient, friendly, skeptical, demanding, indecisive, bargain-hunter, loyal, first-time buyer, silent/unresponsive)
- Backend: update the enum or lookup table for customer types
- Frontend: update the dropdown/selector where customer type is chosen

---

FEATURE 2: LLM Model Options with API Key Authorization
Currently only Ollama is available.
- Add support for: OpenAI (ChatGPT), Anthropic (Claude), Google (Gemini), Ollama
- Ollama remains the DEFAULT model
- Backend: create a user_llm_settings table (user_id, provider, api_key, is_default)
- Backend: controller and API routes for saving/updating LLM settings
- Frontend: settings page where user can input and save their API keys per provider
- Frontend: model selector dropdown during chat session

---

FEATURE 3: Dynamic Products to Sell
Currently hardcoded to tech products.
- Seller (user) can create and manage their own products
- Backend: create a products table (id, user_id, name, description, price, category, image_url)
- Backend: ProductController with CRUD, API routes, ProductResource
- Frontend: product management page (add, edit, delete products)
- Frontend: product selector before starting a chat session
- The selected product details are passed to the AI so the conversation is about that specific product

---

Important notes:
- Follow existing code structure and conventions in the project
- Use Laravel best practices (FormRequest, Resource, Policy if needed)
- Use Vue 3 Composition API and Pinia for state management

```
---
Security Prompt:

```
Review this code as a senior security engineer.

Check for:
1. Security vulnerabilities
2. Injection risks (SQL, NoSQL, Command, XSS)
3. Authentication flaws
4. Authorization issues (RBAC, privilege escalation)
5. Hardcoded secrets or API keys
6. Insecure data handling (plaintext passwords, tokens)
7. Missing input validation or sanitization
8. Unsafe file uploads
9. Insecure API endpoints
10. Improper error handling (leaking sensitive info)
11. Missing rate limiting or brute-force protection
12. Dependency vulnerabilities
13. Logging of sensitive data
14. Misuse of environment variables
15. Insecure external API calls

For each issue:
- Explain the risk
- Show example attack scenario
- Suggest exact fix (code-level if possible)

Be strict and assume this will be deployed in production.
```
