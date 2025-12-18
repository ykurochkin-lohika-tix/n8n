# n8n Simple Screener Flow

This n8n workflow performs strategic sector and company screening to identify top U.S. companies across promising sectors using AI-powered analysis with automated Telegram notifications.

## Flow Description

The workflow consists of three sequential phases:

### Phase 1: Sector Analysis
1. **Manual Trigger**: Start the workflow execution
2. **Sector Analysis Chain**: Uses Claude Sonnet 4.5 to analyze and identify 5 promising U.S. sectors based on:
   - Strategic importance (technological sovereignty, national security, economic growth)
   - Demand/supply dynamics (explosive demand, inelastic supply, barriers to entry)
   - Innovation stage (emerging/growth/maturity, patents, R&D activity)
   - Geopolitical context (U.S. strategic advantage, supply chain importance)
   - Risks and resilience (raw material dependence, political risks)
3. **Structured Output**: Sectors returned as structured JSON data with overall scores (1-10 scale)

### Phase 2: Company Analysis
4. **Company Analysis Chain**: Takes sector results and uses Claude Sonnet 4.5 to find top 3 **outsider or dark horse** companies per sector (up to 15 total) with the strongest reasons and greatest chance to lead their sectors, based on:
   - Financial metrics (revenue growth, free cash flow, moderate debt levels)
   - Innovation indicators (R&D spending, patents, new products)
   - Management quality (experience, strategic decisions, execution capability)
   - Risk factors (debt burden, supplier dependence, regulatory risks)
   - Institutional ownership (major fund shareholders)
   - Scenario analysis (optimistic, base, pessimistic with 2-5 year horizon)
5. **Structured Output**: Companies returned with detailed investment thesis (up to 200 words) that synthesizes all analysis criteria including financials, innovation assessment, management evaluation, risk factors, institutional ownership, scenario planning, and valuation multiples. Each sector includes a winner analysis.

### Phase 3: Output Formatting & Notification
6. **Split into Companies**: JavaScript code node that transforms the structured output into individual formatted messages
7. **Telegram Delivery**: Sends formatted results to configured Telegram channels with emoji-enhanced formatting (📊 for sectors, 🏆 for sector winners)

## Setup Instructions

### Prerequisites
- n8n instance (self-hosted or cloud)
- Anthropic API access (Claude Sonnet 4.5)
- Telegram Bot (optional, for notifications)
  - Bot token from [@BotFather](https://t.me/botfather)
  - Chat ID for target channel/group

### Configuration Steps

1. **Import Workflow**
   - In n8n, go to Workflows > Import from File
   - Select `simple_screener_flow.json`

2. **Configure Anthropic Credentials**
   - Set up Anthropic API credentials in n8n
   - Update both "Claude for Sector Analysis" and "Claude for Company Analysis" nodes with your credential ID
   - Replace the placeholder credential ID `NasIdgbSm8kLTASn` with your actual Anthropic API credential

3. **Configure Telegram Notifications (Optional)**
   - Set up Telegram API credentials in n8n with your bot token
   - Update the "Private channel text message" node with your chat ID
   - Replace the placeholder chat ID `-5071543046` with your target chat/channel ID
   - Optionally enable the "Bot channel text message" node and configure its chat ID `267757685`
   - To disable Telegram notifications, disable both Telegram nodes

4. **Adjust Settings (Optional)**
   - **Temperature**: Currently set to 0.3 for more focused, analytical responses. Increase for more creative analysis (max 1.0)
   - **Model**: Using Claude Sonnet 4.5 (`claude-sonnet-4-5-20250929`). Can be changed to other Claude models if needed
   - Adjust settings in the respective Claude model nodes if needed

5. **Activate Workflow**
   - Click "Active" toggle to enable the workflow for scheduled or manual execution

## Usage

1. Click "Execute workflow" or use the manual trigger
2. Wait for Phase 1 (Sector Analysis) to complete - typically 30-60 seconds
3. Phase 2 (Company Analysis) will automatically start with sector results - typically 60-120 seconds
4. Phase 3 (Output Formatting) processes and sends results to Telegram
5. Review the results:
   - **In n8n**: View structured JSON output in the execution log
   - **In Telegram**: Receive formatted messages with:
     - 5 analyzed sectors with detailed criteria assessment and 1-10 scores
     - Up to 15 companies (top 3 per sector) with detailed investment thesis (up to 200 words each)
     - Each investment thesis includes: financials, innovation, management, risks, institutional ownership, scenarios (optimistic/base/pessimistic), and valuation multiples
     - Sector winner analysis identifying the best company per sector with comparative rationale
     - Emoji indicators (📊 for sectors, 🏆 for winners)

## Output Structure

### Sector Analysis Output (JSON)
```json
[{
  "sector_name": "Advanced Semiconductor Manufacturing and Design",
  "strategic_importance": "Detailed explanation of strategic importance...",
  "demand_supply_analysis": "Analysis of demand/supply dynamics and barriers to entry...",
  "innovation_stage": "Growth stage with innovation indicators...",
  "geopolitical_context": "Strategic advantage and supply chain importance...",
  "risks_resilience": "Key risks and resilience factors...",
  "overall_score": "10/10 - Rating with justification"
}]
```

### Company Analysis Output (JSON)
```json
[{
  "sector": "Advanced Semiconductor Manufacturing and Design",
  "companies": [{
    "rank": 1,
    "company_name": "Marvell Technology",
    "ticker": "MRVL",
    "investment_thesis": "Comprehensive analysis (up to 200 words) including: revenue growth rates, FCF generation, debt metrics, R&D intensity, patent portfolio, management track record, strategic acquisitions, institutional ownership percentages, scenario analysis with specific CAGRs (optimistic/base/pessimistic), valuation multiples vs peers, and competitive positioning."
  }],
  "sector_winner": "Which company is best and why - comparative analysis of strengths"
}]
```

### Telegram Message Format
Each company is sent as a separate message:
```
📊 **Advanced Semiconductor Manufacturing and Design**

1. **Marvell Technology** (MRVL)
[200-word investment thesis with financials, innovation metrics, scenarios, and valuation]

🏆 **Sector Winner:** [Winner analysis for the top-ranked company]

==================================================
```

## Analytical Frameworks Used

- **Sector Analysis**: PESTLE, SWOT, Porter's Five Forces, techno-nationalism, supply chain analysis, economic rent assessment
- **Company Analysis**: Fundamental analysis, DCF/Comps valuation, FCF quality metrics, capital allocation assessment, equity research methodologies

## Workflow Architecture

**Node Structure**:
1. Manual Trigger → 2. Sector Analysis Chain → 3. Company Analysis Chain → 4. Split into Companies → 5. Telegram Notifications (parallel)

**Connections**:
- Sector Analysis Chain uses Claude Sonnet 4.5 (via `ai_languageModel`) and Sector Output Parser (via `ai_outputParser`)
- Company Analysis Chain uses Claude Sonnet 4.5 (via `ai_languageModel`) and Company Output Parser (via `ai_outputParser`)
- Split into Companies receives company data and transforms it into individual message items
- Both Telegram nodes receive messages in parallel (one disabled, one active)

**Data Flow**:
- Sector results pass to Company Analysis via `{{ JSON.stringify($json.output, null, 2) }}`
- Company results transform through JavaScript code splitting
- Each company becomes a separate item for parallel Telegram delivery

## Notes

- **Execution time**: Typically 2-4 minutes total (30-60s for sectors, 60-120s for companies, <10s for formatting)
- **Sequential execution**: Phase 2 depends on Phase 1 results; Phase 3 depends on Phase 2
- **Parallel delivery**: Multiple Telegram messages sent simultaneously for faster notification
- **Investment focus**: Targets "outsiders or dark horses" with leadership potential, not just current market leaders
- **Data accuracy**: Results are based on AI analysis and should be validated with current financial data
- **Use case**: Best used as a screening tool for further research, not as sole investment advice
- **Model temperature**: Set to 0.3 for focused, analytical responses (not creative speculation)

## Customization

You can modify the workflow to:
- **Change analysis scope**: Edit prompts to analyze different numbers of sectors/companies
- **Adjust selection criteria**: Modify prompts to focus on established leaders vs dark horses
- **Customize output format**: Edit the "Split into Companies" JavaScript code for different message formatting
- **Add additional outputs**:
  - Google Sheets export for spreadsheet analysis
  - CSV/JSON file output for data archival
  - Database storage for historical tracking
  - Email notifications as alternative to Telegram
- **Enhance with real-time data**:
  - Add web scraping nodes for live financial data
  - Integrate with financial APIs (Alpha Vantage, Yahoo Finance)
  - Connect to SEC EDGAR for regulatory filings
- **Schedule execution**: Use n8n's workflow triggers for daily/weekly automated screening
- **Chain workflows**: Connect output to portfolio management or alert workflows
