# Rhythm

This project follows a clear rhythm for development and collaboration:

- **Branching:** All changes are made in feature branches (e.g., `rhythm-update`) before merging to main.
- **Commits:** Each logical change is committed with a clear, descriptive message.
- **Documentation:** Major discussions and decisions are summarized in `discussion_summary.md`.
- **Testing:** Code and workflow are tested before merging.
- **Review:** Changes are reviewed for clarity and correctness.

---

For more details, see `discussion_summary.md`.

# LangGraph JavaScript Setup Discussion Summary

**Date**: May 18, 2025, 09:09 AM +07  
**Participants**: Human, Grok 3 (AI)

## Overview

This document summarizes a discussion on setting up an AI agent using LangGraph.js in a JavaScript-only environment on WSL, avoiding Python’s complexities. Topics included Python vs. JavaScript comparisons, local model hosting, MCP server usage (Arcade, Tavily Search), testing, and a minimal “Hello World” agent setup.

## Initial Setup and Concerns

- **Context**: The user analyzed an X post about the Open Agent Platform (OAP), which enables non-developers to build AI agents using LangGraph, MCP tools, and LangConnect for RAG.
- **User Concerns**:
  - Preferred JavaScript to avoid Python’s virtual environments and legacy issues.
  - Questioned “maturity” (tutorials, community support, integrations).
  - Asked about Yarn vs. npm/pnpm, local vs. cloud, environment variables, API costs, interaction methods, MCP servers (Arcade, web search), and unit testing.
- **Initial Responses**:
  - LangGraph.js allows a JavaScript-only setup.
  - **Maturity**:
    - Python: More tutorials (often outdated, e.g., using `gpt-3.5-turbo`), larger community (LangChain Discord), more integrations (e.g., local models).
    - JavaScript: Fresher tutorials, focused community, fewer integrations (no local model support).
  - **Yarn**: Chosen for speed and consistency (`yarn.lock`); npm/pnpm are alternatives.
  - **Local vs. Cloud**:
    - Local: LangGraph server at `http://localhost:8123`, OAP web app at `http://localhost:3000`.
    - Cloud: LangGraph Cloud offers 10,000 invocations/day, 10 GB storage (requires API key).
  - **Environment Variables**: `GOOGLE_API_KEY` for Gemini model, `NEXT_PUBLIC_DEPLOYMENTS` for OAP.
  - **Interaction**: Terminal (`langgraph invoke`) or OAP web interface.
  - **MCP Servers**: Suggested Arcade for email/database/file operations, Tavily Search for web search.

## Python vs. JavaScript for Beginners

- **Syntax**:
  - **Python**: `print("Hello")`, no curly braces (`if x > 0:`), forces new line and indentation.
    - Issue: Indentation errors (e.g., mixing tabs/spaces).
    - Example: `if x > 0: print("Positive")` requires indentation.
  - **JavaScript**: `console.log("Hello")`, curly braces optional for one-liners (`if (x > 0) console.log("Positive");`).
    - Advantage: No indentation issues, simpler for small scripts.
    - Example: `if (x > 0) { console.log("Positive"); } else { console.log("Non-positive"); }` (verbose for `if-else`).
  - **Conclusion**: JavaScript is simpler for one-liners; Python’s indentation can be a hassle.
- **Async Handling**:
  - **JavaScript**: `async/await` is straightforward.

    ```javascript
    async function fetchData() {
      const response = await fetch('https://api.example.com');
      return response.json();
    }
    ```

    - Issues: Forgetting `await`, error handling with `try/catch`.
  - **Python**: `asyncio` is more complex.

    ```python
    import asyncio
    async def fetch_data():
        await asyncio.sleep(1)
        return "data"
    asyncio.run(fetch_data())
    ```

    - Issues: Requires `asyncio.run()`, event loop errors (e.g., `RuntimeError: asyncio.run() cannot be called from a running event loop`).
  - **Conclusion**: JavaScript’s `async/await` is simpler than Python’s `asyncio`.
- **Tooling**:
  - **Python**: Virtual environments (`venv`) are error-prone (e.g., forgetting `source venv/bin/activate`).
  - **JavaScript**: `npm`/`yarn` is intuitive, no virtual environments needed.
- **Surprises**:
  - Python: Indentation errors, mutable default arguments.
  - JavaScript: `async/await` mistakes, scoping (`var` vs. `let`).
- **Revised Stance**: Python’s “beginner-friendly” label (readable syntax, tutorials) is undermined by tooling and surprises. JavaScript is better for this use case due to simpler syntax, async handling, and dependency management.

## Local Model Hosting

- **Hugging Face with Python**:
  - **Can You Run Locally?**: Yes, for open-source models (e.g., `distilbert-base-uncased`) or restricted models (e.g., `meta-llama/Llama-3-8b`) with permission.

    ```python
    from transformers import AutoModelForCausalLM, AutoTokenizer
    model_name = "distilbert-base-uncased"
    model = AutoModelForCausalLM.from_pretrained(model_name)
    tokenizer = AutoTokenizer.from_pretrained(model_name)
    ```

  - **Space Requirements**:
    - Small models: ~250 MB (`distilbert-base-uncased`), ~1 GB RAM.
    - Large models: 16 GB (`Llama-3-8b`), ~16 GB RAM.
    - Dependencies: Python (~50 MB), `transformers` (~50 MB), PyTorch (~1-2 GB).
    - Total: 2-3 GB for small models, 20-60 GB for large models.
  - **Risk**: Large models can strain storage/RAM on WSL (e.g., small SSD).
- **JavaScript Alternative**:
  - JavaScript’s `@langchain/huggingface` only supports the Inference API, not local models.
  - **Ollama**: Run models like LLaMA 3 locally.

    ```bash
    ollama pull llama3
    ollama run llama3
    ```

    ```javascript
    async function callOllama(prompt) {
      const response = await fetch('http://localhost:11434/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ model: 'llama3', prompt })
      });
      return await response.json();
    }
    ```

    - Space: Similar to Hugging Face (16 GB for LLaMA 3 8B).

## Arcade MCP Server

- **Setup**:

  ```bash
  mkdir ~/mcp-servers
  cd ~/mcp-servers
  git clone https://github.com/modelcontextprotocol/arcade
  cd arcade
  npm install
  npm install google-auth-library
  ```

- **Email Sending (Secure with OAuth 2.0):**
  - Concern: Storing Gmail passwords in config.json is insecure.
  - Solution: Use OAuth 2.0.
    - Set up credentials in Google Cloud Console (Gmail API, OAuth 2.0 Client ID).
    - Update config.json:

      ```json
      {
        "tools": {
          "email/send": {
            "provider": "gmail",
            "auth": {
              "type": "oauth2",
              "clientId": "your-client-id",
              "clientSecret": "your-client-secret",
              "redirectUri": "http://localhost:8787/oauth2callback"
            }
          }
        }
      }
      ```

    - Run: `npm run dev`, authenticate via the provided URL.
    - Usage:

      ```javascript
      async function sendEmailTool(input) {
        const response = await fetch('http://localhost:8787/mcp/email/send', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            to: 'recipient@example.com',
            subject: 'Agent Email',
            body: input
          })
        });
        return await response.json();
      }
      ```

    - Emails are sent from the authenticated Gmail account to the recipient.

- **Database Access:**
  - Configure SQLite in config.json:

    ```json
    {
      "tools": {
        "database/query": {
          "type": "sqlite",
          "path": "./database.db"
        }
      }
    }
    ```

  - Create table: `sqlite3 database.db "CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT); INSERT INTO users (name) VALUES ('Alice');"`
  - Usage:

    ```javascript
    async function queryDatabaseTool(query) {
      const response = await fetch('http://localhost:8787/mcp/database/query', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ query })
      });
      return await response.json();
    }
    ```

- **File Operations:**
  - Write to file:

    ```javascript
    async function writeFileTool(content) {
      const response = await fetch('http://localhost:8787/mcp/file/write', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          path: './output.txt',
          content
        })
      });
      return await response.json();
    }
    ```

## Search MCP

- **Tavily Search:**
  - Requires API key (free tier: 1,000 searches/month, no credit card, Tavily Pricing).
  - Setup:

    ```bash
    mkdir ~/mcp-servers
    cd ~/mcp-servers
    git clone https://github.com/tavily-ai/tavily-mcp-server
    cd tavily-mcp-server
    npm install
    npm run dev
    ```

  - **No-Key Alternative:** Not practical—most search APIs require keys. Scraping with puppeteer is complex and often against terms of service.

## Testing and Function Design

- **Implicit State:**
  - LangGraph passes state to node functions (runLlm) during execution.
  - In tests, pass state explicitly:

    ```javascript
    const result = await runLlm(
      { input: { value: 'Tell me a joke' } },
      { modelName: 'gemini-1.5-pro' }
    );
    ```

  - No hidden state in tests—ensures predictability.
- **Node Function:**
  - runLlm is standalone, taking state and config (optional):

    ```javascript
    async function runLlm(state, config = { modelName: 'gemini-1.5-pro' }) {
      const model = new ChatGoogleGenerativeAI({ modelName: config.modelName });
      const response = await model.invoke([{ role: 'user', content: state.input.value }]);
      return { output: { value: response.content } };
    }
    ```

  - LangGraph passes state implicitly during execution.
- **Testing:**
  - Test runLlm in isolation and the full graph:

    ```javascript
    describe('Agent Tests', () => {
      test('runLlm should respond to input', async () => {
        const result = await runLlm(
          { input: { value: 'Tell me a joke' } },
          { modelName: 'gemini-1.5-pro' }
        );
        expect(result.output.value).toBeTruthy();
      });

      test('agent should respond to input', async () => {
        const graph = buildGraph({ modelName: 'gemini-1.5-pro' });
        const result = await graph.invoke({ input: { value: 'Tell me a joke' } });
        expect(result.output.value).toBeTruthy();
      });
    });
    ```

## Final Code Example

### File: graph.js

```javascript
const { StateGraph, START, END } = require('@langchain/langgraph');
const { ChatGoogleGenerativeAI } = require('@langchain/google-genai');
const fetch = require('node-fetch');

require('dotenv').config();

const stateSchema = {
  input: { value: null },
  output: { value: null }
};

async function sendEmailTool(input) {
  const response = await fetch('http://localhost:8787/mcp/email/send', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      to: 'recipient@example.com',
      subject: 'Agent Email',
      body: input
    })
  });
  return await response.json();
}

async function queryDatabaseTool(query) {
  const response = await fetch('http://localhost:8787/mcp/database/query', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ query })
  });
  return await response.json();
}

async function writeFileTool(content) {
  const response = await fetch('http://localhost:8787/mcp/file/write', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      path: './output.txt',
      content
    })
  });
  return await response.json();
}

async function runLlm(state, config = { modelName: 'gemini-1.5-pro' }) {
  const model = new ChatGoogleGenerativeAI({ modelName: config.modelName });
  let message = state.input.value;

  if (message.includes('send email')) {
    const emailResult = await sendEmailTool('Hello from the agent!');
    message = `Email sent: ${emailResult.status}`;
  } else if (message.includes('query database')) {
    const dbResult = await queryDatabaseTool('SELECT * FROM users');
    message = `Database query result: ${JSON.stringify(dbResult)}`;
  } else if (message.includes('write file')) {
    const fileResult = await writeFileTool('Hello, world!');
    message = `File written: ${fileResult.status}`;
  } else {
    const response = await model.invoke([{ role: 'user', content: message }]);
    message = response.content;
  }

  return { output: { value: message } };
}

function buildGraph(config = { modelName: 'gemini-1.5-pro' }) {
  const graph = new StateGraph({ channels: stateSchema });
  graph.addNode('llm', runLlm, config);
  graph.addEdge(START, 'llm');
  graph.addEdge('llm', END);
  return graph.compile();
}

module.exports = { buildGraph, runLlm };
```

## Setup Steps

- **Project Setup:**

  ```bash
  mkdir my_agent_js
  cd my_agent_js
  npm init -y
  npm install @langchain/langgraph @langchain/google-genai dotenv node-fetch
  npm install --save-dev jest
  ```

- **Environment Variables:**

  ```makefile
  GOOGLE_API_KEY=your-google-api-key
  ```

- **Test:**

  ```bash
  npm test
  ```

- **Arcade MCP Server:**

  ```bash
  mkdir ~/mcp-servers
  cd ~/mcp-servers
  git clone https://github.com/modelcontextprotocol/arcade
  cd arcade
  npm install
  npm install google-auth-library
  ```

- **config.json:**

  ```json
  {
    "tools": {
      "email/send": {
        "provider": "gmail",
        "auth": {
          "type": "oauth2",
          "clientId": "your-client-id",
          "clientSecret": "your-client-secret",
          "redirectUri": "http://localhost:8787/oauth2callback"
        }
      },
      "database/query": {
        "type": "sqlite",
        "path": "./database.db"
      },
      "file/write": {
        "path": "./output.txt"
      }
    }
  }
  ```

- **Create database:**

  ```bash
  sqlite3 database.db "CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT); INSERT INTO users (name) VALUES ('Alice');"
  ```

- **Run:**

  ```bash
  npm run dev
  ```

- **Deploy Locally:**

  ```bash
  npm install -g @langchain/langgraph-cli
  langgraph dev
  ```

- **OAP Setup:**

  ```bash
  git clone https://github.com/langchain-ai/open-agent-platform.git
  cd open-agent-platform/apps/web
  npm install
  ```

- **.env.local:**

  ```makefile
  NEXT_PUBLIC_DEPLOYMENTS=[{"id":"my-agent","deploymentUrl":"http://localhost:8123","isDefault":true,"defaultGraphId":"agent"}]
  NEXT_PUBLIC_MCP_SERVER_URL=http://localhost:8787
  ```

- **Run:**

  ```bash
  npm run dev
  ```

- **Interact:**
  - Terminal: `npm run invoke -- "send email"`
  - Web: <http://localhost:3000>

## Conclusion and Next Steps

- **Takeaways:**
  - JavaScript with LangGraph.js is preferred, avoiding Python’s complexities (virtual environments, indentation issues).
  - Ollama enables local model hosting in JavaScript, bypassing Hugging Face.
  - Arcade MCP server provides secure email sending (OAuth 2.0), database access, and file operations.
  - Testing ensures predictability by passing state explicitly.
- **Next Steps:**
  - Upload this file to a GitHub repository.
  - Explore additional MCP servers (e.g., Tavily Search).
  - Extend the agent with multi-node workflows.

---

**Verification of Full Content:**

The file content above is complete and not truncated. The last line is “- Extend the agent with multi-node workflows.”, which is the intended end of the “Next Steps” section. All sections (Overview, Initial Setup, Python vs. JavaScript, Local Model Hosting, Arcade MCP Server, Search MCP, Testing, Final Code Example, Setup Steps, and Conclusion) are fully included, with concise yet comprehensive details, code examples, and setup steps.

To upload to GitHub:

1. Copy the content above into a file named `discussion_summary.md`.
2. Create a repository on GitHub (e.g., `langgraph-discussion`).
3. Upload the file via the GitHub UI (“Add file” → “Upload files”) or CLI:

   ```bash
   git clone https://github.com/your-username/langgraph-discussion.git
   cd langgraph-discussion
   # Copy discussion_summary.md to the directory
   git add discussion_summary.md
   git commit -m "Add discussion summary for LangGraph setup"
   git push origin main
   ```
