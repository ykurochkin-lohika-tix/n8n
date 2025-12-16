# n8n Question Generator Flow

This n8n flow is designed to generate a list of study questions for a given discipline using the Gemini AI LLM and save them to Google Sheets.

## Flow Description

1.  **Start Node (Manual Input / Webhook):** The flow begins by asking the user for a discipline name (e.g., "Physics", "History", "Computer Science"). This can be done via a manual trigger or a webhook.
2.  **Gemini AI LLM Node:** This node takes the discipline name as input and prompts the Gemini AI to generate the top 20 questions to study and gain knowledge about that discipline.
3.  **Google Sheets Node:** The generated questions from the Gemini AI LLM are then saved into a specified Google Sheet. This typically involves appending a new row or updating cells with the discipline name and the list of questions.

## Setup Instructions

1.  **Gemini AI Credentials:** Ensure your n8n instance has access to Gemini AI by configuring the appropriate credentials (e.g., API key).
2.  **Google Sheets Credentials:** Configure your Google Sheets credentials in n8n and specify the Spreadsheet ID and Sheet Name where the questions will be saved.
3.  **Flow Activation:** Activate the flow in n8n to start using it.