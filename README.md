<<<<<<< HEAD
# AgentCore Project

This project was created with the [AgentCore CLI](https://github.com/aws/agentcore-cli).
=======
# Project: Building a Production-Grade Customer Support AI Agent with Amazon Bedrock AgentCore

**Udacity — AWS AI Engineering Nanodegree — Course 2**

---

## Overview

In this project you will build a fully functional, production-ready AI customer support agent for a fictional Amazon store. Starting from a simple local chatbot, you will progressively add cloud infrastructure, external tool integration, a knowledge base, persistent memory, a code interpreter, and a browser — finishing with a deployable agent that can handle real customer inquiries end-to-end.

By the end of the project your agent will be able to:

- Answer questions about products, return policies, and loyalty rewards using Retrieval-Augmented Generation (RAG)
- Look up order status and process refunds by calling Lambda functions through the AgentCore Gateway
- Remember customer preferences and conversation history across multiple sessions
- Calculate exact loyalty discounts using a secure code sandbox
- Navigate websites to fetch live information

---

## Learning Objectives

After completing this project you will be able to:

1. Deploy an AI agent to Amazon Bedrock AgentCore
2. Wire up external Lambda tools via the AgentCore Gateway using the Model Context Protocol (MCP)
3. Implement RAG with a Bedrock Knowledge Base
4. Add short-term (session) and long-term (cross-session) memory using AgentCore Memory
5. Use the AgentCore Code Interpreter for precise computation
6. Integrate the AgentCore Browser tool for live web access
7. Monitor and observe agent behaviour with Amazon CloudWatch

---

## Prerequisites

### AWS Account

- An active AWS account with permission to create and manage:
  - IAM roles and policies
  - Lambda functions
  - API Gateway REST APIs
  - Amazon Bedrock Knowledge Bases (with S3 and OpenSearch access)
  - Amazon Bedrock AgentCore resources (Runtime, Gateway, Memory)
  - Amazon CloudWatch
- All resources should be created in **us-east-1** (N. Virginia) unless stated otherwise.

### Local Development Environment

| Tool | Version |
|------|---------|
| Python | 3.14+ |
| [uv](https://docs.astral.sh/uv/) | Latest |
| AWS CLI | v2 |
| AgentCore CLI (`agentcore`) | Installed via the starter-toolkit |
| Node.js (for MCP Inspector) | 18+ |

### Model Access

Enable the following models in the Amazon Bedrock console under **Model access**:

- **Amazon Nova Lite** (`amazon.nova-lite-v1:0`)

---
>>>>>>> 2b9d04d3f297804b3fb06adb813dd67733a753dc

## Project Structure

```
<<<<<<< HEAD
my-project/
├── AGENTS.md               # AI coding assistant context
├── agentcore/
│   ├── agentcore.json      # Project config (agents, memories, credentials, gateways, evaluators)
│   ├── aws-targets.json    # Deployment targets (account + region)
│   ├── .env.local          # Secrets — API keys (gitignored)
│   ├── .llm-context/       # TypeScript type definitions for AI assistants
│   │   ├── agentcore.ts    # AgentCoreProjectSpec types
│   │   └── aws-targets.ts  # Deployment target types
│   └── cdk/                # CDK infrastructure (@aws/agentcore-cdk)
├── app/                    # Agent application code
└── evaluators/             # Custom evaluator code (if any)
```

## Getting Started

### Prerequisites

- **Node.js** 20.x or later
- **Python 3.10+** and **uv** for Python agents ([install uv](https://docs.astral.sh/uv/getting-started/installation/))
- **AWS credentials** configured (`aws configure` or environment variables)
- **Docker** (only for Container build agents)

### Development

Run your agent locally:

```bash
agentcore dev
```

### Validate Invocation Input

Validate runtime invocation payloads before forwarding them to an agent framework. Keep user prompts typed as strings
and pass only prompt text to the agent.

### Deployment

Deploy to AWS:

```bash
agentcore deploy
```

## Commands

| Command | Description |
| --- | --- |
| `agentcore create` | Create a new AgentCore project |
| `agentcore add` | Add resources (agent, memory, credential, gateway, evaluator, policy) |
| `agentcore remove` | Remove resources |
| `agentcore dev` | Run agent locally with hot-reload |
| `agentcore deploy` | Deploy to AWS via CDK |
| `agentcore status` | Show deployment status |
| `agentcore invoke` | Invoke agent (local or deployed) |
| `agentcore logs` | View agent logs |
| `agentcore traces` | View agent traces |
| `agentcore eval` | Run evaluations |
| `agentcore package` | Package agent artifacts |
| `agentcore validate` | Validate configuration |
| `agentcore pause` | Pause a deployed agent |
| `agentcore resume` | Resume a paused agent |
| `agentcore fetch` | Fetch remote resource definitions |
| `agentcore import` | Import existing resources |
| `agentcore update` | Check for CLI updates |

## Configuration

Edit the JSON files in `agentcore/` to configure your project. See `agentcore/.llm-context/` for type definitions and validation constraints.

The project uses a **flat resource model** — agents, memories, credentials, gateways, evaluators, and policies are top-level arrays in `agentcore.json`. Resources are independent; agents discover memories and credentials at runtime via environment variables or SDK calls.

## Resources

| Resource | Purpose |
| --- | --- |
| Agent (runtime) | HTTP, MCP, or A2A agent deployed to AgentCore Runtime |
| Memory | Persistent context storage with configurable strategies |
| Credential | API key or OAuth credential providers |
| Gateway | MCP gateway that routes tool calls to targets |
| Gateway Target | Tool implementation (Lambda, MCP server, OpenAPI, Smithy, API Gateway) |
| Evaluator | Custom LLM-as-a-Judge or code-based evaluation |
| Online Eval Config | Continuous evaluation pipeline for deployed agents |
| Policy | Cedar authorization policies for gateway tools |

### Agent Types

- **Template agents**: Created from framework templates (Strands, LangChain/LangGraph, GoogleADK, OpenAI Agents, Autogen)
- **BYO agents**: Bring your own code with `agentcore add agent --type byo`
- **Import agents**: Import existing Bedrock agents with `agentcore import`

### Build Types

- **CodeZip**: Python source packaged as a zip and deployed directly to AgentCore Runtime
- **Container**: Docker image built via CodeBuild (ARM64), pushed to ECR, and deployed to AgentCore Runtime

## Documentation

- [AgentCore CLI](https://github.com/aws/agentcore-cli)
- [AgentCore CDK Constructs](https://github.com/aws/agentcore-l3-cdk-constructs)
- [Amazon Bedrock AgentCore](https://aws.amazon.com/bedrock/agentcore/)
=======
project/
├── INSTRUCTIONS.md          ← this file
├── RUBRIC.md                ← grading criteria
├── starter/
│   ├── main.py              ← your starting point (fill in the TODOs)
│   └── lambda/
│       ├── order_tracker.py     ← provided; deploy as-is
│       └── refund_processor.py  ← provided; deploy as-is
└── solution/                ← reference implementation (do not copy)
    ├── main.py
    ├── product_catalog.txt
    ├── pyproject.toml
    ├── lambda/
    │   ├── order_tracker.py
    │   ├── refund_processor.py
    │   └── lambda_schema       ← JSON schema for Gateway tool registration
    └── step-by-step/           ← one file per build step (for reference)
```

---

## Part 1 — AWS Infrastructure Setup

Complete these steps **before** writing any agent code.

### Step 1.1 — Project Initialisation

```bash
# Create a new Python project managed by uv
uv init customer-support-agent
cd customer-support-agent

# Install core dependencies
uv add strands-agents strands-agents-tools
uv add bedrock-agentcore bedrock-agentcore-starter-toolkit
```

### Step 1.2 — Deploy the Lambda Functions

The two Lambda functions (`order_tracker.py` and `refund_processor.py`) are provided in `starter/lambda/`. Deploy them to AWS Lambda before proceeding.

1. In the AWS Lambda console, create two new functions (Python 3.12 runtime):
   - `order-tracker`
   - `refund-processor`
2. Paste the contents of each file into the inline code editor (or zip and upload).
3. Attach an execution role with basic Lambda permissions (CloudWatch Logs).
4. Note the ARN of each function — you will need them in the next step.

### Step 1.3 — Set Up the AgentCore Gateway

The Gateway exposes your Lambda functions as MCP tools that the agent can call.

1. Open the **Amazon Bedrock** console → **AgentCore** → **Gateways**.
2. Create a new Gateway named `CustomerSupportGateway`.
3. Add two **Lambda targets**:

   | Target Name | Lambda Function | Integration |
   |---|---|---|
   | `order_tracker` | `order-tracker` | API Gateway REST proxy |
   | `refund_processor` | `refund-processor` | Direct Lambda invocation |

4. For `order_tracker`, configure API Gateway routes:
   - `GET /orders/{order_id}`
   - `GET /customers/{customer_id}/orders`
   - `GET /customers/{customer_id}`

5. For `refund_processor`, import the tool schema from `solution/lambda/lambda_schema`.

6. Copy the **Gateway URL** (ends with `/mcp`) — paste it into `GATEWAY_URL` in your `main.py`.

**Verify with MCP Inspector:**
```bash
npx @modelcontextprotocol/inspector
# Connect to your Gateway URL and confirm all tools are listed.
```

### Step 1.4 — Create the Knowledge Base

1. Upload `solution/product_catalog.txt` to an **S3 bucket** in your account.
2. In the Bedrock console → **Knowledge Bases**, create a new Knowledge Base:
   - Name: `CustomerSupportKB`
   - Data source: the S3 bucket from above
   - Embeddings model: Amazon Titan Embeddings v2
   - Vector store: Amazon OpenSearch Serverless (auto-created)
3. **Sync** the data source.
4. Copy the **Knowledge Base ID** — paste it into `KB_ID` in your `main.py`.

**Verify:**
```bash
# In the console, use the Knowledge Base "Test" tab
# Query: "What is the return policy for electronics?"
# Expected: 15-day return window for electronics
```

### Step 1.5 — Create the AgentCore Memory Resource

1. In the Bedrock console → **AgentCore** → **Memory**, create a new Memory resource:
   - Name: `CustomerSupportMemory`
2. Add two **Memory Strategies**:

   | Strategy | Name | Namespace |
   |---|---|---|
   | Semantic extraction | `customer_facts` | `cs_agent/{actorId}/facts` |
   | User preference | `customer_preferences` | `cs_agent/{actorId}/preferences` |

3. Copy the **Memory ID** — paste it into `MEMORY_ID` in your `main.py`.

---

## Part 2 — Building the Agent

Open `starter/main.py`. It contains scaffolding and `# TODO` comments marking every section you need to implement. Work through the TODOs in order.

The step-by-step reference files in `solution/step-by-step/` show the state of the code after each section is complete — consult them if you get stuck, but try to implement each section yourself first.

### Section 1 — Configuration and Initialisation

Fill in your resource IDs and set up:
- `BedrockAgentCoreApp`
- `BedrockModel` with Amazon Nova Lite
- `MemoryClient` and `boto3` Bedrock runtime client

### Section 2 — Knowledge Base Tool

Implement `search_knowledge_base(query)`:
- Call the Bedrock Knowledge Base Retrieve API
- Join result chunks with `"\n---\n"`

**Test:**
```bash
agentcore invoke '{"prompt": "Is the Kindle Paperwhite waterproof?"}'
# Expected: mention of IPX8 rating
```

### Section 3 — Long-Term Memory Hook

Implement `MemoryHook` with two methods:
- `retrieve_customer_context` — query all memory namespaces and prepend results to the user message
- `save_support_interaction` — save the completed (user, assistant) turn after each response

### Section 4 — Loyalty Discount Tool (Code Interpreter)

Implement `calculate_loyalty_discount(loyalty_points, tier, order_total, product_category)`:
- Build a Python code string containing the discount logic
- Execute it with `code_session()` and return the JSON result
- Include a fallback for when the Code Interpreter is unavailable

**Test:**
```bash
agentcore invoke '{"prompt": "I am a Gold member with 4250 points. Calculate my discount on a $150 order.", "customer_id": "CUST-123", "session_id": "s1"}'
```

### Section 5 — Main Entrypoint

Implement the `invoke(payload, context)` function:
- Extract `prompt`, `customer_id`, and `session_id` from the payload
- Instantiate `MemoryHook` and `AgentCoreBrowser`
- Connect to the Gateway via `MCPClient` and load gateway tools
- Build the `Agent` with all tools and hooks and return its response

### Section 6 — Deploy to AgentCore

```bash
# Configure the AgentCore CLI (first time only)

agentcore create 
#follow the interactive wizard to complete

#Deploy locally
agentcore dev --no-browser --logs

#Test locally
agentcore dev --prompt "Can you track my order ORD-002"
# Deploy the agent
agentcore deploy --dry-run 
agentcore deploy

# Invoke the deployed agent
agentcore invoke '{"prompt": "Hello, what can you help me with?", "customer_id": "CUST-123", "session_id": "test-1"}'
```

---

## Part 3 — Functional Testing

Run the following test scenarios and verify the expected behaviour. Include screenshots or copy the terminal output in your submission.

### Test 1 — Order Tracking

```bash
agentcore invoke '{"prompt": "Can you track order ORD-001?", "customer_id": "CUST-123", "session_id": "t1"}'
# Expected: shipping status, tracking number TRK987654321, carrier UPS, estimated delivery
```

### Test 2 — Refund Processing

```bash
agentcore invoke '{"prompt": "I want to return my Kindle Paperwhite (ORD-002). Please initiate a refund.", "customer_id": "CUST-123", "session_id": "t2"}'
# Expected: refund ID, APPROVED status, 3-5 business days message
```

### Test 3 — Knowledge Base (RAG)

```bash
agentcore invoke '{"prompt": "What are the benefits of the Platinum loyalty tier?", "customer_id": "CUST-123", "session_id": "t3"}'
# Expected: free same-day shipping, 15% discount, priority support
```

### Test 4 — Memory (Long-Term)

```bash
# Session A — introduce yourself
agentcore invoke '{"prompt": "Hi, I am Jane. I prefer concise responses.", "customer_id": "CUST-123", "session_id": "s-A"}'

# Session B (new session) — verify recall
agentcore invoke '{"prompt": "Do you remember my name and communication preference?", "customer_id": "CUST-123", "session_id": "s-B"}'
# Expected: agent recalls "Jane" and "concise responses"
```

### Test 5 — Loyalty Discount Calculation

```bash
agentcore invoke '{"prompt": "I am a Gold member with 4250 points. Calculate my discount on a $150 standard order.", "customer_id": "CUST-123", "session_id": "t5"}'
# Expected: points redeemed, tier discount 10%, final total, remaining points
```

### Test 6 — Browser Tool

```bash
agentcore invoke '{"prompt": "Go to https://www.amazon.com and tell me the page title.", "customer_id": "CUST-123", "session_id": "t6"}'
# Expected: page title retrieved from live Amazon.com
```

---

## Part 4 — CloudWatch Monitoring

1. In the AWS console, navigate to **CloudWatch** → **Log Groups**.
2. Find the log group for your AgentCore Runtime (named after your deployment).
3. Create a **metric filter** on `ERROR` log entries.
4. Create a **CloudWatch Alarm** that triggers when the error count exceeds 5 in a 5-minute window.
5. Take a screenshot of the alarm configuration and include it in your submission.

---

## Demonstrations and evidence

| Scenario | Evidence | What to inspect |
| --- | --- | --- |
| Order tracking | [Test 1](screenshots/test1.JPG) | Shipment status, carrier, and tracking number |
| Refund processing | [Test 2](screenshots/test2.JPG) | Gateway-backed refund response and generated ID |
| Knowledge retrieval | [Test 3](screenshots/test3.JPG) | Catalog-backed Platinum benefits |
| Cross-session memory | [Test 4A](screenshots/test4a.JPG), [Test 4B](screenshots/test4b.png) | Recall of Jane's name and preference; inspect the prompt shown in each capture |
| Loyalty calculation | [Test 5](screenshots/test5.JPG) | Points redemption, tier discount, final total, and remaining points |
| Browser automation | [Test 6](screenshots/test6.JPG) | Retrieved Udacity page title |

These are manual integration demonstrations, not a load-test report or an automated regression suite. Some captures also show limitations worth addressing: a simulated refund defaults to $0 when an amount is omitted, and remembered context can distract the model from the current request. Review approval establishes completion of the capstone requirements; it does not replace production validation.

To reproduce the memory scenario, introduce yourself in one request and ask for recall in another, keeping the same `customer_id` while the `session_id` changes. Considering I used the new agentcore CLI, run the below to attach the created agentcore memory to my agentcore runtime.

```bash

agentcore import memory --arn <memory-arn> --name <memory-name>
```
---

## Helpful References

- [Amazon Bedrock AgentCore Documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/agentcore.html)
- [Strands Agents Documentation](https://strandsagents.com)
- [MCP Inspector](https://github.com/modelcontextprotocol/inspector)
- [uv Package Manager](https://docs.astral.sh/uv/)
>>>>>>> 2b9d04d3f297804b3fb06adb813dd67733a753dc
