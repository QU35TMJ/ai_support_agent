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

## Project Structure

```
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
| Order tracking | [Test 1](screenshots/test_1.JPG) | Shipment status, carrier, and tracking number |
| Refund processing | [Test 2](screenshots/test_2.JPG) | Gateway-backed refund response and generated ID |
| Knowledge retrieval | [Test 3](screenshots/test_3.JPG) | Catalog-backed Platinum benefits |
| Cross-session memory | [Test 4A](screenshots/test_4a.JPG), [Test 4B](screenshots/test_4b.png) | Recall of Jane's name and preference; inspect the prompt shown in each capture |
| Loyalty calculation | [Test 5](screenshots/test_5.JPG) | Points redemption, tier discount, final total, and remaining points |
| Browser automation | [Test 6](screenshots/test_6.JPG) | Retrieved Udacity page title |

These are manual integration demonstrations, not a load-test report or an automated regression suite. Some captures also show limitations worth addressing: a simulated refund defaults to $0 when an amount is omitted, and remembered context can distract the model from the current request. Review approval establishes completion of the capstone requirements; it does not replace production validation.

To reproduce the memory scenario, introduce yourself in one request and ask for recall in another, keeping the same `customer_id` while the `session_id` changes. Considering I used the new agentcore CLI, run the below to attach the created agentcore memory to my agentcore runtime.

```bash

agentcore import memory --arn <memory-arn> --name <memory-name>
```
---

## Troubleshooting and Lessons

This exercise reinforced that building AI agents in production environments is often less about writing the agent logic itself and more about understanding the surrounding infrastructure, tooling, and integration points. I spent significant time troubleshooting environment issues, runtime configurations, Python version compatibility, AWS permissions, API Gateway integrations, Lambda deployments, and AgentCore configuration. While these challenges were sometimes frustrating, they gave me a much deeper understanding of how the different components interact.

One key lesson was the importance of systematically isolating problems. Rather than assuming the issue was with my code, I learned to validate each layer independently. For example, when AgentCore was failing due to a SQLite error, I verified Python versions, tested imports directly, confirmed SQLite availability, and eventually identified that AgentCore was launching a different interpreter than my virtual environment. Similarly, when working with Gateway and Lambda integrations, I learned to distinguish between configuration issues, IAM permission issues, and application logic issues.

The exercise also highlighted the value of understanding the development lifecycle beyond coding. I gained hands-on experience creating and deploying Lambda functions, configuring API Gateway resources and operations, working with AgentCore Gateway targets, setting up memory services, and testing local versus deployed runtimes. These are practical skills that are essential for moving AI agents from prototypes into real-world systems.

Perhaps the biggest takeaway was developing persistence and troubleshooting discipline. Many of the issues were not solved by a single fix but by carefully gathering evidence, testing assumptions, and narrowing down possibilities. By the end of the exercise, I felt much more confident navigating AWS services, AgentCore tooling, and complex debugging scenarios than when I started.

---

## My Reflection

One design decision I made was to use a session-based approach for the agent by including a session_id in requests. This decision was important because the exercise involved AgentCore Memory, and maintaining a consistent session allows conversations and customer interactions to be linked across multiple requests. Using session identifiers also aligns with how production AI agents maintain context and provide a more personalized user experience. Additionally, I structured the agent to access customer, order, and refund information through Gateway tools rather than hardcoding business logic, making the solution more modular and extensible.

One challenge I encountered was troubleshooting the local development environment. While running the agent with agentcore dev, I repeatedly received a ModuleNotFoundError: No module named '_sqlite3' error. Initially, I assumed the issue was related to my code or dependencies, but after systematically testing different Python versions, I discovered that the system's Python 3.13 installation lacked SQLite support. I verified that SQLite worked correctly in Python 3.10 and that the agent could start successfully using Uvicorn directly. By tracing the runtime behavior and comparing environments, I determined that AgentCore was launching a different Python interpreter than the one in my virtual environment. This experience reinforced the importance of isolating issues and validating assumptions when debugging complex systems.

For a production environment, I would extend this agent in several ways. First, I would integrate authentication and authorization to ensure customer data is accessed securely. Second, I would enhance memory capabilities to maintain longer-term customer context and interaction history. I would also implement monitoring, logging, and alerting through services such as CloudWatch to improve observability and operational support. Finally, I would introduce automated testing, CI/CD pipelines, and infrastructure-as-code to support reliable deployments and simplify ongoing maintenance as the solution scales.

---

## Helpful References

- [Amazon Bedrock AgentCore Documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/agentcore.html)
- [Strands Agents Documentation](https://strandsagents.com)
- [MCP Inspector](https://github.com/modelcontextprotocol/inspector)
- [uv Package Manager](https://docs.astral.sh/uv/)
