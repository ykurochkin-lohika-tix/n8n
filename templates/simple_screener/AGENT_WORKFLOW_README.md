# AI Agent-Based Screener Workflow

## Overview

This document describes the **agent-based version** of the simple_screener workflow (`simple_screener_flow_agent.json`), which replaces the basic LLM chains with autonomous AI agents equipped with web search capabilities and memory.

## Key Differences from Basic LLM Chain Workflow

### Original Workflow (`simple_screener_flow.json`)
- Uses **basic LLM chains** (`@n8n/n8n-nodes-langchain.chainLlm`)
- Static prompts with no ability to search for real-time information
- No memory between different stages of analysis
- Relies on model's training data (cutoff date limitations)

### Agent-Based Workflow (`simple_screener_flow_agent.json`)
- Uses **AI agents** (`@n8n/n8n-nodes-langchain.agent`)
- **Web search capability** via SerpAPI tool (currently disabled)
- **No memory implementation** in current version (stateless agents)
- Can fetch **real-time information** when web search is enabled:
  - Current government policies and programs
  - Latest market trends and forecasts
  - Recent company financials and stock prices
  - Current news and analyst reports
  - Up-to-date institutional ownership data

## Workflow Architecture

### Node Structure

```
Manual Trigger
    ↓
Sector Analysis Agent ←─ Claude Sonnet 4.5 (temp: 0.3)
    └── Sector Output Parser (structured JSON)
    ↓
Company Analysis Agent ←─ Claude Sonnet 4.5 (temp: 0.3)
    ├── SerpAPI (tool - currently disabled)
    └── Company Output Parser (structured JSON)
    ↓
Split into Companies (Code node)
    ↓
Telegram Notifications (2 channels)
```

### 1. Sector Analysis Agent

**Purpose**: Identify 5 strategic U.S. sectors with high potential

**Capabilities**:
- **Agent Reasoning**: Uses AI agent capabilities to analyze and identify sectors
- **System Prompt**: Instructs agent to use web search tool for latest information on:
  - Government policies (CHIPS Act, IRA updates)
  - Geopolitical developments
  - Market trends and demand forecasts
  - Patent filings and R&D activities
  - Supply chain dynamics
- **Note**: Currently operates without connected web search tool (relies on model knowledge)

- **Structured Output**: Returns JSON array with:
  - `sector_name`
  - `strategic_importance`
  - `demand_supply_analysis`
  - `innovation_stage`
  - `geopolitical_context`
  - `risks_resilience`
  - `overall_score` (1-10)

**Model**: Claude Sonnet 4.5 (temperature: 0.3 for analytical focus)

### 2. Company Analysis Agent

**Purpose**: Find top 3 dark horse companies per sector (up to 15 total)

**Capabilities**:
- **Agent Reasoning**: Uses AI agent capabilities to analyze and rank companies
- **SerpAPI Tool**: Connected but currently disabled - when enabled, searches for:
  - Current stock prices and market caps
  - Recent quarterly earnings
  - Latest company announcements
  - Analyst reports and ratings
  - Institutional ownership data
  - Recent news affecting companies
- **System Prompt**: Includes instruction to limit web search usage to max 10 times
- **Note**: Currently operates without active web search (relies on model knowledge and pinned test data)

- **Structured Output**: Returns JSON array with:
  - `sector`
  - `companies` array:
    - `rank` (1-3)
    - `company_name`
    - `ticker`
    - `investment_thesis` (max 200 words)
  - `sector_winner` (comparative analysis)

**Model**: Claude Sonnet 4.5 (temperature: 0.3 for analytical focus)

### 3. Split into Companies

Code node that transforms structured output into individual Telegram messages with emoji formatting.

### 4. Telegram Notifications

Two parallel channels:
- **Bot channel** (disabled by default): chatId `267757685`
- **Private channel** (active): chatId `-5071543046`

## Memory Implementation

**Current Status**: No memory nodes are implemented in the current workflow version.

The agents operate in stateless mode, meaning:
- Each agent execution is independent
- Sector analysis results are passed to Company Analysis Agent via node connections (not memory)
- No conversation history is maintained between tool calls
- Agents rely on the prompt context and previous node outputs

**Future Enhancement**: Memory nodes (Window Buffer Memory) could be added to:
- Maintain context of web searches during execution
- Enable iterative refinement of analysis
- Support multi-turn reasoning within each agent phase

## Web Search Tools

The workflow includes **SerpAPI** tool (`@n8n/n8n-nodes-langchain.toolSerpApi`):
- **Connection**: Only connected to Company Analysis Agent
- **Status**: Currently disabled
- **Functionality** (when enabled):
  - Searches the web for relevant information via Google Search API
  - Returns search results to the agent
  - Enables real-time data access
- **Configuration**: Requires SerpAPI account credentials

**Note**: The prompts for both agents mention using web search, but only the Company Analysis Agent has a web search tool connected, and it's currently disabled. To enable live web searches, enable the SerpAPI node and ensure valid credentials are configured.

## Benefits of Agent-Based Approach

### Potential Benefits (when web search is enabled):
1. **Real-Time Data**: Access to current market conditions, news, and financial data
2. **Dynamic Analysis**: Agents can search for specific information based on context
3. **Better Quality**: More accurate and up-to-date investment theses
4. **Autonomous Decision-Making**: Agents decide what information to search for

### Current Implementation Benefits:
1. **Agent Architecture**: Structured for tool use and autonomous reasoning
2. **Date-Aware**: Prompts include current date (`{{ $now.format('yyyy-MM-dd') }}`)
3. **Flexible Framework**: Easy to enable web search or add additional tools
4. **Test Data**: Includes comprehensive pinData for testing and development
5. **Modular Design**: Clean separation between sector and company analysis phases

## Usage Instructions

### Prerequisites
- n8n instance running
- Anthropic API credentials configured (required for Claude Sonnet 4.5)
- Telegram bot credentials (required for notification delivery)
- SerpAPI credentials (optional - only needed if enabling web search tool)
- Internet connectivity (for API calls and optional web search)

### Importing the Workflow

1. Open n8n
2. Click "Import from File"
3. Select `simple_screener_flow_agent.json`
4. Verify credentials are connected:
   - Anthropic API (both Claude nodes)
   - Telegram API (both Telegram nodes)
   - SerpAPI (optional - if you want to enable web search)

**Note**: The workflow includes pinData (test data) for both Sector Analysis Agent and Company Analysis Agent nodes. This allows you to test the output formatting and Telegram delivery without consuming API credits. To run with live AI analysis, unpin the data from these nodes.

### Execution

#### With PinData (Testing Mode - Default):
1. Open the workflow in n8n
2. Click "Execute Workflow" or the manual trigger node
3. The workflow will:
   - Skip AI agent execution (use pinned test data)
   - Format the test results (5 sectors, 15 companies)
   - Send Telegram messages to configured channels
4. **Execution Time**: ~5-10 seconds (only runs Split and Telegram nodes)

#### Without PinData (Live AI Analysis):
1. Unpin data from "Sector Analysis Agent" and "Company Analysis Agent" nodes
2. (Optional) Enable SerpAPI node if you want live web search
3. Click "Execute Workflow"
4. Watch as agents:
   - Analyze and select 5 sectors based on current conditions
   - Analyze and rank 15 companies (3 per sector)
   - Format and send Telegram messages
5. **Execution Time**:
   - Without web search: 2-4 minutes (AI processing only)
   - With web search: 5-10 minutes (includes search API calls)

## Output Format

Same as the original workflow:
- 15 companies (3 per sector)
- Emoji-enhanced formatting (📊 for sectors, 🏆 for winners)
- Detailed investment thesis (max 200 words per company)
- Sector winner analysis
- Telegram delivery to configured channels

## Troubleshooting

### Agent Not Using Web Search
- **Check SerpAPI Status**: The SerpAPI node is disabled by default - enable it first
- **Connection**: Only the Company Analysis Agent has SerpAPI connected (Sector Agent has none)
- **Credentials**: Ensure valid SerpAPI credentials are configured
- **Prompt**: The system prompts mention web search, but tools must be enabled to actually search

### Using PinData vs Live Execution
- **Testing**: PinData allows testing output format without consuming API credits
- **Unpinning**: Click the pin icon on agent nodes to use live AI analysis
- **Validation**: Pinned data shows example results from December 2024 market conditions

### Structured Output Failing
- Verify JSON schema example is valid
- Check that output parser is connected via `ai_outputParser`
- Ensure agent prompt instructs to return data in specified format

### Rate Limiting
- Web searches may hit rate limits on some sites
- Anthropic API has rate limits - ensure you're within quota
- Consider adding delays between executions if needed

## Future Enhancements

Potential additions to improve the agent workflow:

1. **Additional Tools**:
   - Calculator tool for financial calculations
   - SQL database tool for historical data
   - Custom API tools for financial data providers (Alpha Vantage, Yahoo Finance)

2. **Advanced Memory**:
   - Vector store memory for semantic search across past analyses
   - Redis-backed memory for persistence across workflow runs

3. **Multi-Agent Collaboration**:
   - Separate agents for financial analysis, risk assessment, and competitive analysis
   - Coordinator agent to synthesize findings

4. **Validation Layer**:
   - Fact-checking agent to verify claims
   - Compliance agent to check regulatory requirements

## Comparison: When to Use Which Workflow

### Use Basic LLM Chain Workflow (`simple_screener_flow.json`) When:
- You want simpler, more predictable execution
- You don't need agent reasoning capabilities
- You prefer straightforward LLM chain architecture
- You want to minimize complexity

### Use Agent Workflow (`simple_screener_flow_agent.json`) When:
- You want the flexibility to add tools (web search, calculations, APIs)
- You prefer agent-based architecture for future extensibility
- You want to test with pinned data before live execution
- You plan to enable real-time web search capabilities
- You need autonomous reasoning capabilities

### Current State Comparison:
| Feature | Basic LLM Chain | Agent Workflow (Current) |
|---------|----------------|-------------------------|
| Architecture | LLM Chains | AI Agents |
| Web Search | Not available | Available but disabled |
| Memory | None | None (can be added) |
| Test Data | None | Comprehensive pinData |
| Execution Time | 2-3 minutes | 2-4 min (similar without web search) |
| Extensibility | Limited | High (tool-ready) |
| Cost | Lower | Similar (without web search) |

**Note**: With web search disabled, both workflows currently rely on Claude's training data and produce similar results. The agent workflow's main advantage is its extensibility and test data infrastructure.

## Cost Considerations

### Current Configuration (Web Search Disabled):
- **API Costs**: Similar to basic LLM workflow (same number of Claude API calls)
- **Token Usage**: Comparable between both approaches
- **SerpAPI**: No cost when disabled
- **Estimated Cost**: ~$0.10-0.30 per execution (Anthropic API only)

### With Web Search Enabled (Future):
- **Additional Costs**: SerpAPI charges per search (~$0.002-0.005 per search)
- **More LLM Calls**: Agents make additional calls for tool use decisions
- **Estimated Cost**: ~$0.50-1.00 per execution (varies with search frequency)
- **Cost Increase**: 2-5x compared to current configuration

### Cost Optimization Tips:
- Use pinData for testing to avoid API charges
- Limit web search frequency (prompt includes "use max 10 times" instruction)
- Consider caching frequent searches
- Monitor SerpAPI usage if enabled

## Tags

The workflow includes the following tags:
- `investment` - Investment screening and analysis
- `finance` - Financial analysis workflow
- `analysis` - Analytical workflow category
- `ai-agent` - AI agent-based workflow

## Version Information

- **Workflow Name**: `simple_screener_flow_agent`
- **Workflow ID**: `A19rEEogU8bZzHeJ`
- **Version ID**: `2f62e776-0e0a-4609-bcb9-c3fbde9b9f31`
- **Based On**: `simple_screener_flow` (LLM chain version)
- **Created**: December 2024
- **Last Updated**: December 2024
- **n8n Version**: Compatible with n8n v1.0+
- **Model**: Claude Sonnet 4.5 (claude-sonnet-4-5-20250929)
- **Configuration**:
  - Temperature: 0.3 (analytical focus)
  - Active: false (must be activated for scheduled runs)
  - Execution Order: v1 (sequential)
- **Test Data**: Includes pinData from December 2024 analysis

## Support

For issues or questions:
1. Check the troubleshooting section above
2. Review n8n agent documentation
3. Verify all connections in the workflow
4. Check n8n execution logs for detailed error messages
