# n8n Simple Screener Flow

This n8n workflow performs strategic sector and company screening to identify top U.S. companies across promising sectors using AI-powered analysis.

## Flow Description

The workflow consists of two sequential analysis phases:

### Phase 1: Sector Analysis
1. **Manual Trigger**: Start the workflow execution
2. **Sector Analysis Chain**: Uses Claude Sonnet 4.5 to analyze and identify 5 promising U.S. sectors based on:
   - Strategic importance (technological sovereignty, national security, economic growth)
   - Demand/supply dynamics (explosive demand, inelastic supply, barriers to entry)
   - Innovation stage (emerging/growth/maturity, patents, R&D activity)
   - Geopolitical context (U.S. strategic advantage, supply chain importance)
   - Risks and resilience (raw material dependence, political risks)
3. **Structured Output**: Sectors returned as structured JSON data

### Phase 2: Company Analysis
4. **Company Analysis Chain**: Takes sector results and uses Claude Sonnet 4.5 to find top 3 companies per sector (up to 15 total) based on:
   - Financial metrics (revenue growth, free cash flow, moderate debt)
   - Innovation indicators (R&D spending, patents, new products)
   - Management quality (experience, strategic decisions, execution)
   - Risk factors (debt burden, supplier dependence, regulatory risks)
   - Institutional ownership (major fund shareholders)
5. **Structured Output**: Companies returned with concise investment thesis (50 words) that synthesizes all analysis criteria including detailed financials, innovation assessment, management evaluation, risk factors, institutional ownership, scenario planning (optimistic/base/pessimistic), and valuation multiples

## Setup Instructions

### Prerequisites
- n8n instance (self-hosted or cloud)
- Anthropic API access (Claude Sonnet 4.5)

### Configuration Steps

1. **Import Workflow**
   - In n8n, go to Workflows > Import from File
   - Select `simple_screener_flow.json`

2. **Configure Credentials**
   - Set up Anthropic API credentials in n8n
   - Update both "Claude for Sector Analysis" and "Claude for Company Analysis" nodes with your credential ID
   - Replace the placeholder credential ID `NasIdgbSm8kLTASn` with your actual Anthropic API credential

3. **Adjust Settings (Optional)**
   - **Temperature**: Currently set to 0.3 for more focused, analytical responses. Increase for more creative analysis (max 1.0)
   - **Max Tokens**:
     - Sector Analysis: 8000 tokens
     - Company Analysis: 16000 tokens (more detailed output)
   - Adjust these in the respective Claude model nodes if needed

4. **Activate Workflow**
   - Click "Active" toggle to enable the workflow

## Usage

1. Click "Execute workflow" or use the manual trigger
2. Wait for Phase 1 (Sector Analysis) to complete - typically 30-60 seconds
3. Phase 2 (Company Analysis) will automatically start with sector results
4. Review the final structured output containing:
   - 5 analyzed sectors with detailed criteria assessment
   - Up to 15 companies (top 3 per sector) with concise investment thesis (50 words each)
   - Each investment thesis includes: financials, innovation, management, risks, scenarios, and valuation
   - Sector winner analysis with comparative rationale (50 words per sector)

## Output Structure

### Sector Analysis Output
```json
[{
  "sector_name": "Sector name",
  "strategic_importance": "Analysis...",
  "demand_supply_analysis": "Analysis...",
  "innovation_stage": "Stage and indicators",
  "geopolitical_context": "Strategic advantages",
  "risks_resilience": "Risk factors",
  "overall_score": "1-10 rating"
}]
```

### Company Analysis Output
```json
[{
  "sector": "Sector name",
  "companies": [{
    "rank": 1,
    "company_name": "Company name",
    "ticker": "TICK",
    "investment_thesis": "Comprehensive analysis including financial metrics (revenue growth, FCF, debt), innovation indicators (R&D, patents), management quality, risk factors, institutional ownership, scenarios (optimistic/base/pessimistic), and valuation multiples. Limited to 50 words."
  }],
  "sector_winner": "Best company in the sector and rationale. Limited to 50 words."
}]
```

## Analytical Frameworks Used

- **Sector Analysis**: PESTLE, SWOT, Porter's Five Forces, techno-nationalism, supply chain analysis, economic rent assessment
- **Company Analysis**: Fundamental analysis, DCF/Comps valuation, FCF quality metrics, capital allocation assessment, equity research methodologies

## Notes

- Execution time: Typically 2-5 minutes total for both phases
- The workflow uses sequential execution (Phase 2 depends on Phase 1 results)
- Results are based on AI analysis and should be validated with current financial data
- Best used as a screening tool for further research, not as sole investment advice

## Customization

You can modify the workflow to:
- Change the number of sectors analyzed (edit the prompt in Sector Analysis Chain)
- Adjust companies per sector (edit the prompt in Company Analysis Chain)
- Add export nodes (Google Sheets, CSV, database) to save results
- Add web scraping nodes to pull real-time financial data
- Chain with other workflows for automated reporting
