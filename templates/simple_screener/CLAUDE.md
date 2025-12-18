# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a template directory for an n8n workflow called "simple_screener" within a larger n8n templates repository. The workflow is designed to perform strategic sector and company screening for top U.S. companies across promising sectors.

## Workflow Purpose

The workflow has two main phases:

1. **Sector Search Phase**: Identify 5 potential U.S. sectors based on strategic importance, demand/supply dynamics, innovation maturity, geopolitical context, and risk resilience
2. **Company Search Phase**: Within identified sectors, find and rank top 3 public companies per sector (up to 15 companies total) based on financial metrics, innovativeness, management quality, and risk factors

**Expected Output**: A ranked list of up to 15 companies from potential sectors, each with a concise investment thesis (50 words) synthesizing financial metrics, innovation indicators, management quality, risk assessment, scenarios, and valuation. Each sector includes a comparative winner analysis.

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
- `templates/simple_screener/` - This workflow (currently in requirements phase)

## n8n Workflow Development

### File Format
n8n workflows are JSON files with the following structure:
- `name`: Workflow name
- `nodes`: Array of workflow nodes (triggers, actions, AI models, data transformations)
- `connections`: Defines how data flows between nodes
- `pinData`: Optional test data pinned to nodes
- `settings`: Execution settings

### Node Types (from example workflow)
- `n8n-nodes-base.manualTrigger` - Manual workflow trigger
- `n8n-nodes-base.set` - Set/edit field values
- `@n8n/n8n-nodes-langchain.chainLlm` - Basic LLM chain for prompts
- `@n8n/n8n-nodes-langchain.lmChatGoogleGemini` - Google Gemini AI model
- `@n8n/n8n-nodes-langchain.lmChatAnthropic` - Anthropic Claude model
- `@n8n/n8n-nodes-langchain.outputParserStructured` - Structured JSON output parser

### Workflow Design Pattern
From the reference workflow, a typical pattern is:
1. Trigger node (manual or webhook)
2. Set initial variables
3. LLM Chain with prompt
4. Language Model (connected via `ai_languageModel`)
5. Output Parser (connected via `ai_outputParser`)

### Structured Output
Use `outputParserStructured` with `jsonSchemaExample` to define expected JSON output structure. This ensures consistent, parseable responses from LLMs.

**Current Output Schemas**:
- **Sector Output**: Array of sectors with fields: sector_name, strategic_importance, demand_supply_analysis, innovation_stage, geopolitical_context, risks_resilience, overall_score
- **Company Output**: Array with sector, companies (rank, company_name, ticker, investment_thesis limited to 50 words), and sector_winner (limited to 50 words)

The simplified company output format consolidates detailed analysis (financials, innovation, management, risks, scenarios, valuation) into a concise investment_thesis field.

## Development Notes

- Each node requires unique `id` (UUID format)
- Nodes have `position` arrays `[x, y]` for visual layout in n8n UI
- Use `{{ $json['field_name'] }}` syntax to reference data from previous nodes
- AI model nodes require credentials configuration (`googlePalmApi`, `anthropicApi`)
- Test workflows with `pinData` to verify behavior before live execution

## Git Workflow

The repository uses conventional commits:
- Recent commits: "update questions", "first commit"
- Keep commit messages descriptive but concise
