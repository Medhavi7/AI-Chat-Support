# OrderlyMeds AI Customer Support Workflow

## Overview

This n8n workflow implements an intelligent customer support chatbot for OrderlyMeds, a GLP-1 weight management platform. The workflow processes customer inquiries through multiple stages of medical safety detection, AI-powered response generation, sentiment classification, logging, and escalation alerts.

## What the Workflow Does

### Core Capabilities

1. **Medical Safety Detection**: Immediately identifies urgent medical emergencies (chest pain, severe reactions, breathing difficulties, etc.) and routes them with strict safety protocols, bypassing the AI agent entirely for critical cases.

2. **AI Agent Response**: Uses a Groq-powered Llama model with 10-message conversation memory to handle general support queries. The agent is equipped with a support ticket creation tool and can assist with account issues, order tracking, and general platform questions.

3. **Classification & Scoring**: Analyzes each interaction for:
   - **Sentiment**: angry, frustrated, happy, or neutral
   - **Topic**: medical, billing, orders, account, prescription, or general
   - **Urgency**: critical, high, medium, or low
   - **Escalation needs**: automatic detection based on severity and ticket creation

4. **Logging & Escalation**: Records all interactions to Google Sheets for analytics and sends formatted HTML email alerts to support staff for critical issues, medical concerns, angry customers, or when support tickets are automatically created.

5. **Response Formatting**: Returns clean, conversational responses to customers through the chat interface.

### Key Features

- **PII Extraction**: Automatically extracts customer email addresses and names from messages using regex patterns
- **Duplicate Prevention**: Checks for existing tickets within the same session to avoid creating duplicates
- **Context-Aware Routing**: Medical questions are always escalated; simple greetings don't trigger unnecessary alerts
- **Conversation Memory**: Maintains a 10-message rolling window for context across the conversation
- **Safety-First Design**: Medical questions never receive AI-generated medical advice; all are referred to healthcare professionals

## How to Trigger It

### Setup Requirements

1. **n8n Installation**: Deploy this workflow in your n8n instance (self-hosted or cloud)

2. **Credentials Required**:
   - **Groq API**: Sign up at [groq.com](https://groq.com) and add your API key to n8n credentials
   - **Google Sheets OAuth2**: Connect your Google account to log chat interactions
   - **Gmail OAuth2**: Connect Gmail for sending escalation alerts

3. **Google Sheet Setup**: Create a Google Sheet with a tab named "Chat Logs" with the following columns:
   - Timestamp
   - Session ID
   - Customer Email
   - Customer Name
   - Message
   - AI Response
   - Sentiment
   - Topic
   - Urgency
   - Needs Escalation
   - Ticket Created

4. **Configuration**: Update the "Config" node with:
   - `supportEmail`: Email address to receive escalation alerts
   - `medicalHotline`: Phone number for medical emergencies (default: 1-800-ORDERLY)
   - `groqModel`: LLM model to use (default: llama-4-scout-17b-16e-instruct)

### Activation

1. Import the JSON workflow into n8n
2. Configure all credentials (replace REDACTED_* placeholders)
3. Update the Google Sheet ID in the "Log to Google Sheets" node
4. Activate the workflow

### Using the Chat Interface

**Access URL**: `https://your-n8n-instance.com/webhook/orderlymeds-chat-support/chat`

The chat interface will:
- Accept natural language messages from customers
- Maintain conversation context using session IDs
- Automatically detect and extract PII (email, name)
- Route medical emergencies immediately
- Provide helpful, conversational responses

**Input Format**: The trigger expects JSON with:
```json
{
  "chatInput": "Customer's message here",
  "sessionId": "optional-session-id"
}
```

If no `sessionId` is provided, one will be auto-generated.

## Assumptions & Design Decisions

### Current Assumptions

1. **Single Support Channel**: The workflow assumes a single support email address and medical hotline number configured in the Config node

2. **Keyword-Based Medical Detection**: Medical emergencies are detected using exact phrase matching against a predefined list. This is fast but may miss nuanced cases or generate false positives.

3. **Google Services**: Relies on Google Sheets for logging and Gmail for alerts. Organizations using different tools would need to modify these nodes.

4. **Session-Based Memory**: Conversation context is limited to 10 messages per session. Cross-session history is not maintained.

5. **Manual Ticket System**: The "Create Support Ticket" tool generates ticket IDs but doesn't integrate with actual ticketing systems like Zendesk, Freshdesk, or Jira.

6. **No Authentication**: The chat webhook is publicly accessible. Production deployments should add authentication.

7. **Missing Order Status Tool**: The AI agent references a `check_order_status` tool in prompts, but this tool is not implemented in the workflow.

### Safety Mechanisms

- **Medical Disclaimer**: All medical questions receive standardized responses directing users to healthcare providers
- **Emergency Bypass**: Urgent medical keywords immediately bypass the AI agent to prevent any potential harmful AI-generated advice
- **Escalation Redundancy**: Multiple triggers for escalation (medical + angry + ticket creation + high urgency)
- **PII Validation**: The ticket creation tool requires both email and name before proceeding

## Recommended Improvements

1. **Enhanced Medical Detection**: Replace keyword matching with ML-based intent classification for better accuracy and fewer false positives.

2. **Authentication & Rate Limiting**: Add API key authentication and rate limiting to prevent abuse of the public webhook.

3. **Real Ticketing Integration**: Connect to actual ticketing systems (Zendesk, Freshdesk, Jira) instead of just generating ticket IDs.

4. **Duplicate Ticket Prevention**: Query Google Sheets log before creating new tickets to prevent cross-session duplicates.

5. **Customer History Lookup**: Query CRM/database for customer history to personalize responses and identify VIP customers.

## Security & Compliance Notes

### Current State

- **PII Handling**: Customer emails and names are extracted and logged to Google Sheets without encryption
- **Data Retention**: No automatic purging of old chat logs
- **Audit Trail**: Basic logging exists but lacks comprehensive audit trails for compliance

### Production Recommendations

1. **HIPAA Compliance** (if handling real medical data):
   - Encrypt PII both in transit and at rest
   - Implement access controls and audit logging
   - Add Business Associate Agreements (BAAs) with all third-party services
   - Set data retention policies with automatic purging

2. **GDPR Compliance**:
   - Add consent capture before collecting PII
   - Implement right-to-deletion workflows
   - Add data processing agreements with vendors
   - Provide data export functionality

3. **Security Hardening**:
   - Add webhook authentication (API keys or OAuth)
   - Implement rate limiting (currently no protection against spam)
   - Use secrets management (e.g., HashiCorp Vault) instead of storing credentials in n8n
   - Add input sanitization to prevent injection attacks

## Troubleshooting

### Common Issues

1. **Workflow not triggering**: Ensure the workflow is activated and the webhook URL is correct
2. **Credentials errors**: Verify all API keys and OAuth connections are valid
3. **Google Sheets not logging**: Check sheet ID, tab name ("Chat Logs"), and column headers match exactly
4. **No escalation emails**: Verify Gmail credentials and check spam folder
5. **AI responses are generic**: Increase context window or improve system prompt with more specific examples

### Testing

Test the workflow with these sample inputs:

```
"Hi" → Should respond without escalation
"I'm having severe chest pain" → Should trigger emergency response
"I want a refund!" → Should create ticket and escalate
"My name is John Doe and email is john@example.com" → Should extract PII
```

## File Structure

```
├── orderlymeds-workflow.json            # Main workflow file
└── README.md                            # This documentation
```

## Import Instructions

1. Open your n8n instance
2. Click "Workflows" → "Import from File"
3. Select `orderlymeds-workflow.json`
4. Update all REDACTED credential IDs with your own
5. Configure the Google Sheet ID
6. Test with sample messages
7. Activate the workflow

## Support

For questions about this workflow:
- Review the n8n documentation: https://docs.n8n.io
- Check n8n community forum: https://community.n8n.io
- Review Groq API docs: https://console.groq.com/docs

## License

This workflow is provided as-is for educational and commercial use. Modify as needed for your organization's requirements.
