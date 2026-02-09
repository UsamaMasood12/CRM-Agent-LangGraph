# 🤖 CRM Agent - Ralph the Marketing AI

An intelligent Customer Relationship Management (CRM) system powered by **LangGraph**, **OpenAI**, and **Model Context Protocol (MCP)**. Meet **Ralph** - your AI-powered marketing assistant who analyzes customer behavior, creates personalized marketing campaigns, and automates email communications.

![Architecture Diagram](.excalidraw.png)

---

## 📑 Table of Contents

- [Features](#-features)
- [Project Architecture](#-project-architecture)
- [Project Structure](#-project-structure)
- [File Documentation](#-file-documentation)
- [Prerequisites](#-prerequisites)
- [Installation](#-installation)
- [Database Setup](#-database-setup)
- [Running the Application](#-running-the-application)
- [Understanding the Data](#-understanding-the-data)
- [Adding New Tools](#-adding-new-tools)
- [Learning Resources](#-learning-resources)

---

## 🚀 Features

| Feature | Description |
|---------|-------------|
| 🧠 **Intelligent Customer Analysis** | Ralph analyzes customer purchase history and behavior patterns using SQL queries |
| 📧 **Personalized Email Campaigns** | Creates targeted marketing emails with customer-specific content |
| 🎯 **Customer Segmentation** | Uses RFM (Recency, Frequency, Monetary) analysis to categorize customers |
| ✋ **Human-in-the-Loop** | Requires human review for sensitive actions like sending campaigns |
| 📊 **Real-time Data** | Works with actual retail transaction data from PostgreSQL |
| 💬 **Slack Integration** | Communicates campaign updates and insights via Slack |

### Campaign Types

- **Re-engagement**: Win back inactive customers
- **Referral**: Leverage high-value customers for referrals  
- **Loyalty**: Thank and retain valuable customers

---

## 🏗️ Project Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Ralph Agent   │    │   Database      │
│   (Chat UI)     │◄──►│   (LangGraph)   │◄──►│   (PostgreSQL)  │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                              │
            ┌─────────────────┼─────────────────┐
            ▼                 ▼                 ▼
     ┌───────────┐     ┌───────────┐     ┌───────────┐
     │ PostgreSQL│     │ Marketing │     │   Slack   │
     │MCP Server │     │MCP Server │     │MCP Server │
     └───────────┘     └───────────┘     └───────────┘
```

### Data Flow

1. **User Input** → User sends a message via the chat interface
2. **Agent Processing** → Ralph (LangGraph agent) processes the request
3. **Tool Execution** → Agent uses MCP tools to query database or execute marketing actions
4. **Human Review** → Protected tools (campaigns, emails) require human approval
5. **Response** → Agent streams response back to the user

---

## 📁 Project Structure

```
crm-agent/
├── .env.example              # Environment variables template
├── .excalidraw.png           # Architecture diagram
├── .gitignore                # Git ignore rules
├── .python-version           # Python version specification (3.13)
├── pyproject.toml            # Project dependencies and metadata
├── README.md                 # This documentation file
│
├── frontend/                 # User interface layer
│   └── chat_local.py         # CLI chat interface with streaming
│
├── src/                      # Source code
│   └── ralph/                # Main agent package
│       ├── __init__.py       # Package initializer
│       ├── graph.py          # LangGraph agent definition
│       ├── prompts.py        # System prompts and configurations
│       │
│       └── my_mcp/           # MCP (Model Context Protocol) integration
│           ├── __init__.py   # Package initializer
│           ├── config.py     # MCP configuration loader
│           ├── mcp_config.json # MCP servers configuration
│           │
│           └── servers/      # Custom MCP servers
│               ├── __init__.py
│               └── marketing_server.py  # Marketing tools server
│
└── db/                       # Database assets
    ├── generate_data_tables.py    # Data generation script
    ├── migration-create-tables.sql # Database schema migration
    │
    └── data/                 # Sample data files
        ├── customers.csv     # Customer information
        ├── items.csv         # Product catalog
        ├── transactions.csv  # Purchase history
        ├── rfm.csv           # RFM analysis results
        └── online_retail_II_2010-2011.csv  # Source dataset
```

---

## 📄 File Documentation

### Root Configuration Files

#### `pyproject.toml`
**Purpose**: Project metadata and dependency management using `uv`.

```toml
[project]
name = "crm-agent"
version = "0.1.0"
requires-python = ">=3.13"
```

**Key Dependencies**:
| Package | Version | Purpose |
|---------|---------|---------|
| `langgraph` | ≥0.4.7 | Stateful AI agent framework |
| `langchain-openai` | ≥0.3.18 | OpenAI LLM integration |
| `langchain-mcp-adapters` | ≥0.1.1 | MCP protocol adapters |
| `psycopg2-binary` | ≥2.9.10 | PostgreSQL database driver |
| `sqlalchemy` | ≥2.0.41 | SQL ORM toolkit |
| `faker` | ≥37.3.0 | Fake data generation |
| `python-dotenv` | ≥1.1.0 | Environment variable loading |

---

#### `.env.example`
**Purpose**: Template for environment variables. Copy to `.env` and fill in your values.

```bash
# Required
OPENAI_API_KEY=           # Your OpenAI API key
SUPABASE_PASSWORD=        # Database password
SUPABASE_URI=             # PostgreSQL connection string

# Optional
SLACK_BOT_TOKEN=          # For Slack integration
SLACK_TEAM_ID=            # Your Slack workspace ID
LANGSMITH_API_KEY=        # For observability/tracing
```

---

### Frontend Layer

#### `frontend/chat_local.py`
**Purpose**: Command-line chat interface for interacting with Ralph.

**Key Components**:

| Function | Description |
|----------|-------------|
| `stream_graph_responses()` | Async generator that streams LLM responses in real-time |
| `main()` | Main event loop handling user input and human approval workflow |

**Features**:
- Real-time streaming of AI responses
- Human-in-the-loop approval for protected tools
- Support for `continue`, `update`, and `feedback` actions during tool review
- Exit commands: `exit` or `quit`

**Usage**:
```bash
cd frontend
uv run python chat_local.py
```

---

### Agent Core (`src/ralph/`)

#### `src/ralph/graph.py`
**Purpose**: Defines the LangGraph agent with nodes, edges, and routing logic.

**Key Components**:

| Component | Type | Description |
|-----------|------|-------------|
| `AgentState` | Pydantic Model | State container with messages, protected tools list, and yolo_mode flag |
| `build_graph()` | Async Function | Constructs the compiled LangGraph with all nodes and edges |
| `assistant_node()` | Node | Invokes the LLM with system prompt and messages |
| `human_tool_review_node()` | Node | Handles human approval workflow with interrupt |
| `assistant_router()` | Edge Router | Routes between tools, human review, or end based on tool calls |
| `inspect_graph()` | Utility | Visualizes the graph using Mermaid diagrams |

**Graph Structure**:
```
START → assistant_node → [tools | human_tool_review_node | END]
              ↑                    │
              └────────────────────┘
```

**Protected Tools** (require human approval):
- `create_campaign`
- `send_campaign_email`

---

#### `src/ralph/prompts.py`
**Purpose**: Contains Ralph's system prompt with database schema, RFM logic, and behavioral guidelines.

**Sections**:

| Section | Description |
|---------|-------------|
| `DB_TABLE_DESCRIPTIONS` | Brief description of each database table |
| `DB_SCHEMA` | Complete PostgreSQL schema definitions |
| `RFM` | RFM scoring and segmentation logic |
| `MARKETING_CAMPAIGNS` | Campaign types and descriptions |
| `MARKETING_EMAILS` | Email writing guidelines |
| `SLACK_INTEGRATION` | Slack communication guidelines |

**RFM Segments**:
| Segment | RFM Score Pattern | Description |
|---------|-------------------|-------------|
| Champion | 555 | Best customers - high R, F, M |
| Recent Customer | 5XX | Recently active |
| Frequent Buyer | X5X | Buys often |
| Big Spender | XX5 | High monetary value |
| At Risk | 1XX | Haven't purchased recently |
| Others | - | Everyone else |

---

### MCP Integration (`src/ralph/my_mcp/`)

#### `src/ralph/my_mcp/config.py`
**Purpose**: Loads and processes MCP configuration, resolving environment variables and relative paths.

**Key Functions**:

| Function | Description |
|----------|-------------|
| `get_project_root()` | Finds project root by locating `pyproject.toml` |
| `resolve_relative_paths()` | Converts relative paths to absolute paths for local servers |
| `resolve_env_vars()` | Substitutes `${VAR}` placeholders with actual environment values |

**Process Flow**:
1. Load `mcp_config.json`
2. Resolve relative paths to absolute paths
3. Resolve environment variables from `.env`
4. Export final `mcp_config` dictionary

---

#### `src/ralph/my_mcp/mcp_config.json`
**Purpose**: Configuration file defining MCP servers used by the agent.

**Configured Servers**:

| Server | Command | Purpose |
|--------|---------|---------|
| `postgres` | `npx @modelcontextprotocol/server-postgres` | Database queries via MCP |
| `marketing` | `python marketing_server.py` | Custom marketing tools |
| `slack` | `npx @modelcontextprotocol/server-slack` | Slack communication |

---

#### `src/ralph/my_mcp/servers/marketing_server.py`
**Purpose**: Custom MCP server providing marketing-specific tools.

**Tools**:

| Tool | Parameters | Description |
|------|------------|-------------|
| `create_campaign` | `name`, `type`, `description` | Creates a marketing campaign in the database |
| `send_campaign_email` | `campaign_id`, `customer_id`, `subject`, `body` | Records and sends campaign emails |

**Campaign Types**: `loyalty`, `referral`, `re-engagement`

---

### Database Layer (`db/`)

#### `db/migration-create-tables.sql`
**Purpose**: SQL migration script to create all required database tables.

**Tables Created**:

| Table | Primary Key | Description |
|-------|-------------|-------------|
| `customers` | `Customer ID` | Customer information with name, email, country |
| `transactions` | `Invoice`, `StockCode` | Purchase history with prices and quantities |
| `items` | `StockCode` | Product catalog with descriptions and prices |
| `rfm` | `Customer ID` | RFM scores and customer segments |
| `marketing_campaigns` | `id` (UUID) | Campaign metadata and tracking |
| `campaign_emails` | `id` (UUID) | Email records with status tracking |

**Security**: All tables have Row Level Security (RLS) enabled.

---

#### `db/generate_data_tables.py`
**Purpose**: Python script to process raw retail data and generate sample datasets.

**Functions**:

| Function | Description |
|----------|-------------|
| `preprocess_data()` | Cleans raw data: removes duplicates, cancelled invoices, NaNs |
| `generate_fake_customer_data()` | Creates realistic fake names/emails using Faker |
| `generate_core_tables()` | Splits data into transactions, items, customers tables |
| `generate_rfm()` | Calculates RFM scores and assigns customer segments |
| `sample_data()` | Samples 100 customers for development/testing |
| `export_data()` | Exports processed data to CSV files |

**Usage**:
```bash
cd db
python generate_data_tables.py
```

---

#### `db/data/` (CSV Files)

| File | Records | Description |
|------|---------|-------------|
| `customers.csv` | 100 | Sampled customer data with fake PII |
| `transactions.csv` | ~4,500 | Transaction history for sampled customers |
| `items.csv` | ~3,600 | Product catalog |
| `rfm.csv` | 100 | Pre-calculated RFM scores |
| `online_retail_II_2010-2011.csv` | ~500K | Original UCI retail dataset |

---

## 📋 Prerequisites

- **Python 3.13+** 
- **uv** - Fast Python package manager ([installation guide](https://docs.astral.sh/uv/guides/install-python/))
- **PostgreSQL** database (Supabase recommended)
- **OpenAI API key**
- **Node.js** (for MCP servers via npx)

---

## ⚡ Installation

### 1. Clone the Repository

```bash
git clone https://github.com/UsamaMasood12/CRM-Agent-LangGraph.git
cd CRM-Agent-LangGraph
```

### 2. Install Dependencies

```bash
uv sync
```

### 3. Configure Environment

```bash
cp .env.example .env
```

Edit `.env` with your credentials:
```bash
OPENAI_API_KEY=sk-...
SUPABASE_URI=postgresql://postgres:password@host:port/postgres
```

---

## 🗄️ Database Setup

### Using Supabase (Recommended)

1. **Create Account**: Go to [supabase.com](https://supabase.com) and create a free account

2. **Create Project**: 
   - Click "New Project"
   - Generate a secure password (save it!)
   - Copy password to `.env` as `SUPABASE_PASSWORD`

3. **Get Connection String**:
   - Go to Project Settings → Database
   - Copy the **Transaction Pooler** connection string
   - Add to `.env` as `SUPABASE_URI`
   - Replace `[YOUR-PASSWORD]` with your actual password

4. **Create Tables**:
   - Open Supabase SQL Editor
   - Copy contents of `db/migration-create-tables.sql`
   - Run the query

5. **Import Data**:
   - Go to Table Editor
   - For each table (`customers`, `transactions`, `items`, `rfm`):
     - Click "Insert" → "Import data from CSV"
     - Upload corresponding file from `db/data/`

---

## 🎮 Running the Application

### Start the Chat Interface

```bash
cd frontend
uv run python chat_local.py
```

### Example Conversation

```
---- 🤖 Assistant ----

Hi there! I'm Ralph, your customer service agent and marketing expert...

User: Show me our top 5 customers by total spending

---- 🤖 Assistant ----

Let me query the database to find your top customers...

User: Create a re-engagement campaign for at-risk customers

----- ✅ / ❌ Human Approval Required -----
{
  "tool_call": {
    "name": "create_campaign",
    "args": { "name": "Win Back Campaign", "type": "re-engagement", ... }
  }
}
Action (continue, update, feedback): continue
```

### Commands

| Command | Description |
|---------|-------------|
| Type any message | Chat with Ralph |
| `exit` or `quit` | End the session |
| `continue` | Approve tool execution |
| `update` | Modify tool arguments |
| `feedback` | Provide feedback to agent |

---

## 📊 Understanding the Data

### RFM Analysis

RFM (Recency, Frequency, Monetary) analysis segments customers based on:

| Metric | Score Range | Meaning |
|--------|-------------|---------|
| **R**ecency | 1-5 | Days since last purchase (5=recent) |
| **F**requency | 1-5 | Number of purchases (5=frequent) |
| **M**onetary | 1-5 | Total spending (5=high value) |

### Database Schema

```sql
-- Core Tables
customers (Customer ID, Country, Name, Email)
transactions (Invoice, InvoiceDate, StockCode, Quantity, Price, TotalPrice, Customer ID)
items (StockCode, Description, Price)
rfm (Customer ID, recency, frequency, monetary, R, F, M, RFM_Score, Segment)

-- Marketing Tables
marketing_campaigns (id, name, type, description, created_at)
campaign_emails (id, campaign_id, customer_id, subject, body, sent_at, status, opened_at, clicked_at)
```

---

## 🔧 Adding New Tools

### Create a New MCP Tool

Add tools to `src/ralph/my_mcp/servers/marketing_server.py`:

```python
@mcp.tool()
async def your_new_tool(param: str) -> str:
    """Your tool description.
    
    Args:
        param: Description of the parameter.
        
    Returns:
        Description of return value.
    """
    # Your implementation here
    return "Tool result"
```

### Add a New MCP Server

1. Create server file in `src/ralph/my_mcp/servers/`
2. Add configuration to `mcp_config.json`:
   ```json
   "your_server": {
     "command": "python",
     "args": ["src/ralph/my_mcp/servers/your_server.py"],
     "transport": "stdio"
   }
   ```

### Protect a Tool (Require Human Approval)

Add the tool name to `protected_tools` in `AgentState`:

```python
class AgentState(BaseModel):
    protected_tools: List[str] = ["create_campaign", "send_campaign_email", "your_new_tool"]
```

---

## 📚 Learning Resources

### Key Concepts

| Concept | Description | Learn More |
|---------|-------------|------------|
| LangGraph | Stateful AI agent framework | [Documentation](https://langchain-ai.github.io/langgraph/) |
| MCP | Model Context Protocol for tool integration | [Protocol Spec](https://modelcontextprotocol.io/) |
| RFM Analysis | Customer segmentation methodology | [Wikipedia](https://en.wikipedia.org/wiki/RFM_(market_research)) |
| Human-in-the-Loop | AI systems with human oversight | [LangGraph Guide](https://langchain-ai.github.io/langgraph/concepts/human_in_the_loop/) |

### Recommended Reading

- [LangGraph Tutorial](https://langchain-ai.github.io/langgraph/tutorials/)
- [MCP Server Examples](https://github.com/modelcontextprotocol/servers)
- [OpenAI API Documentation](https://platform.openai.com/docs)
- [Supabase Documentation](https://supabase.com/docs)

---

## 📝 License

This project is for educational purposes.

---

## 👤 Author

**Usama Masood**
- GitHub: [@UsamaMasood12](https://github.com/UsamaMasood12)
- Email: umasood.bee17seecs@seecs.edu.pk
