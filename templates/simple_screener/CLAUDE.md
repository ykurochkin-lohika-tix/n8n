# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a template directory for n8n workflows called "simple_screener" within a larger n8n templates repository. The directory contains two workflow variants:

1. **simple_screener_flow.json** - Basic LLM chain implementation (legacy)
2. **simple_screener_flow_agent.json** - AI agent-based implementation (current/recommended)

Both workflows perform strategic sector and company screening for top U.S. companies across promising sectors, with automated Telegram notification delivery.

## Workflow Purpose

The workflow has two main phases with automated notification:

1. **Sector Search Phase**: Identify 5 potential U.S. sectors based on strategic importance, demand/supply dynamics, innovation maturity, geopolitical context, and risk resilience
2. **Company Search Phase**: Within identified sectors, find and rank top 3 public companies per sector (up to 15 companies total). Focus on outsiders or dark horses with the strongest reasons and greatest chance to lead their sectors, based on financial metrics, innovativeness, management quality, and risk factors
3. **Output Formatting & Notification**: Format results and deliver via Telegram channels

**Expected Output**: A ranked list of up to 15 companies from potential sectors, each with a detailed investment thesis (up to 200 words) synthesizing financial metrics, innovation indicators, management quality, risk assessment, institutional ownership, scenarios (optimistic/base/pessimistic), and valuation multiples. Each sector includes a comparative winner analysis.

## Key Requirements

### Sector Analysis Criteria
- Strategic importance to U.S. technological sovereignty, economic growth, and national security
- Explosive and irreversible demand with inelastic supply
- Innovation stage (emerging, growth, maturity) with patent and R&D activity
- Geopolitical advantage and supply chain importance
- Risk assessment (raw material dependence, political risks, technological failures)

### Company Analysis Criteria
- Financial: revenue, free cash flow (FCF), moderate debt levels
- Innovation: technologies, R&D, patents, new products
- Management: experience, strategic decisions, execution capability
- Risks: debt burden, supplier dependence, regulatory exposure
- Institutional investor presence (preferable)

### Analytical Frameworks
The workflow should leverage:
- **Sector phase**: PESTLE, SWOT, Porter frameworks for strategic analysis; geopolitical and macroeconomic analysis; assessment of "economic rent" (high demand + inelastic supply)
- **Company phase**: Fundamental analysis, DCF/Comps valuation, quality metrics (FCF over EPS), capital allocation assessment

## Repository Structure

This is part of the parent n8n templates repository at `D:\Programming\n8n`:
- `templates/question_generator/` - Example workflow for generating study questions using Gemini/Anthropic LLMs
- `templates/simple_screener/` - This directory with screening workflows

### Files in This Directory
- `simple_screener_flow.json` - Basic LLM chain workflow (legacy)
- `simple_screener_flow_agent.json` - AI agent-based workflow (current/recommended)
- `CLAUDE.md` - This file, project documentation and guidance
- `AGENT_WORKFLOW_README.md` - Detailed documentation for the agent-based workflow variant
- Other files from git status: Various configuration and documentation files

## Workflow Variants

### Basic LLM Chain Workflow (simple_screener_flow.json)
- **Architecture**: Uses `@n8n/n8n-nodes-langchain.chainLlm` nodes
- **Approach**: Static prompts with direct LLM responses
- **Tools**: None - relies entirely on Claude's training data
- **Memory**: None
- **Use Case**: Simple, predictable execution without external data needs
- **Status**: Legacy implementation, still functional

### Agent-Based Workflow (simple_screener_flow_agent.json) - CURRENT
- **Architecture**: Uses `@n8n/n8n-nodes-langchain.agent` nodes
- **Approach**: AI agents capable of using tools and autonomous reasoning
- **Tools**: SerpAPI for web search (currently disabled)
- **Memory**: None (stateless execution)
- **Test Data**: Includes comprehensive pinData for testing
- **Use Case**: Extensible architecture ready for web search and additional tools
- **Status**: Active development, recommended for new work

**Key Differences**:
- Agent workflow is designed for tool use (web search, APIs) but currently runs without tools enabled
- Agent workflow includes pinData allowing testing without API consumption
- Both workflows currently produce similar results (both rely on Claude's knowledge)
- Agent architecture provides better extensibility for future enhancements

**Documentation**:
- Basic workflow: Documented in this file (CLAUDE.md)
- Agent workflow: Detailed documentation in AGENT_WORKFLOW_README.md

## n8n Workflow Development

### File Format
n8n workflows are JSON files with the following structure:
- `name`: Workflow name
- `nodes`: Array of workflow nodes (triggers, actions, AI models, data transformations)
- `connections`: Defines how data flows between nodes
- `pinData`: Optional test data pinned to nodes
- `settings`: Execution settings

### Node Types (used in this workflow)
- `n8n-nodes-base.manualTrigger` - Manual workflow trigger
- `@n8n/n8n-nodes-langchain.chainLlm` - LLM chain for prompts with structured output
- `@n8n/n8n-nodes-langchain.lmChatAnthropic` - Anthropic Claude Sonnet 4.5 model
- `@n8n/n8n-nodes-langchain.outputParserStructured` - Structured JSON output parser with schema validation
- `n8n-nodes-base.code` - JavaScript code execution for data transformation
- `n8n-nodes-base.telegram` - Telegram bot integration for notifications

### Workflow Design Pattern (Basic LLM Chain Workflow)
The basic workflow (simple_screener_flow.json) follows this pattern:
1. **Manual Trigger** - Initiates workflow execution
2. **Sector Analysis Chain** (LLM Chain)
   - Connected to Claude Sonnet 4.5 via `ai_languageModel`
   - Connected to Sector Output Parser via `ai_outputParser`
3. **Company Analysis Chain** (LLM Chain)
   - Receives sector results from previous chain
   - Connected to Claude Sonnet 4.5 via `ai_languageModel`
   - Connected to Company Output Parser via `ai_outputParser`
4. **Split into Companies** (Code node) - JavaScript transformation to format output
5. **Telegram Notifications** - Two parallel channels (bot channel disabled, private channel active)

### Workflow Design Pattern (Agent-Based Workflow)
The agent workflow (simple_screener_flow_agent.json) follows this pattern:
1. **Manual Trigger** - Initiates workflow execution
2. **Sector Analysis Agent** (AI Agent)
   - Connected to Claude Sonnet 4.5 via `ai_languageModel`
   - Connected to Sector Output Parser via `ai_outputParser`
   - Includes prompt with current date injection
   - No tools currently connected
3. **Company Analysis Agent** (AI Agent)
   - Receives sector results from previous agent
   - Connected to Claude Sonnet 4.5 via `ai_languageModel`
   - Connected to Company Output Parser via `ai_outputParser`
   - Connected to SerpAPI via `ai_tool` (currently disabled)
   - Includes prompt limiting web search to 10 times
4. **Split into Companies** (Code node) - Same JavaScript transformation
5. **Telegram Notifications** - Same two parallel channels

**Note**: See AGENT_WORKFLOW_README.md for comprehensive documentation of the agent workflow.

### Structured Output
Use `outputParserStructured` with `jsonSchemaExample` to define expected JSON output structure. This ensures consistent, parseable responses from LLMs.

**Current Output Schemas**:
- **Sector Output**: Array of sectors with fields: sector_name, strategic_importance, demand_supply_analysis, innovation_stage, geopolitical_context, risks_resilience, overall_score
- **Company Output**: Array with sector, companies (rank, company_name, ticker, investment_thesis up to 200 words), and sector_winner

The company output format consolidates detailed analysis (financials, innovation, management, risks, institutional ownership, scenarios, valuation) into a comprehensive investment_thesis field with a 200-word limit enforced via prompt message.

## Development Notes

- Each node requires unique `id` (UUID format)
- Nodes have `position` arrays `[x, y]` for visual layout in n8n UI
- Use `{{ $json['field_name'] }}` syntax to reference data from previous nodes
- AI model nodes require credentials configuration (`anthropicApi`)
- Test workflows with `pinData` to verify behavior before live execution
- Telegram nodes require bot API credentials and valid chat IDs

### Telegram Integration
The workflow includes two Telegram output channels:
- **Bot channel** (disabled): chatId `267757685` - Can be enabled for bot channel notifications
- **Private channel** (active): chatId `-5071543046` - Currently active for receiving formatted results

Each company result is sent as a separate Telegram message with emoji-enhanced formatting (📊 for sectors, 🏆 for sector winners).

### Output Transformation
The "Split into Companies" node is a JavaScript code node that:
1. Iterates through all sectors and their companies
2. Formats each company into a readable message with markdown formatting
3. Adds the sector winner analysis to the first-ranked company message
4. Creates separate items for each company to enable parallel message sending
5. Includes metadata fields (sector, company, ticker, rank) for tracking

## Workflow Metadata

**Tags**: The workflow is tagged with:
- `investment` - Investment screening and analysis
- `finance` - Financial analysis workflow
- `analysis` - Analytical workflow category

**Settings**:
- Execution order: `v1` (sequential execution)
- Active status: Currently set to inactive (must be activated for scheduled runs)

**Model Configuration**:
- Model: Claude Sonnet 4.5 (`claude-sonnet-4-5-20250929`)
- Temperature: 0.3 (focused, analytical responses)
- Used for both sector and company analysis phases

## Git Workflow

The repository uses conventional commits:
- Recent commits: "update_company_output", "screnner_2", "screnner_1", "update questions", "first commit"
- Keep commit messages descriptive but concise
- Current branch: `main`
