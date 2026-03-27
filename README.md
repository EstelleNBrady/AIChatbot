# Broughton Partners IT Help Chatbot

An AI-powered IT support chatbot built for Broughton Partners employees. The bot handles troubleshooting, answers questions using internal IT documentation, routes urgent issues based on business hours, and automatically creates Jira support tickets.

---

## Features

- **Conversational AI** — Powered by Amazon Bedrock (Claude Haiku 4.5) via LangChain for natural, context-aware responses
- **Persistent Memory** — Conversation history stored per user in DynamoDB, keyed to their Google account email so memory persists across sessions
- **RAG from Internal Docs** — Lex QnA intent searches internal IT PDFs stored in S3 and passes relevant context to Claude for grounded responses
- **Business Hours Routing** — Urgent issues route to a phone number after hours or a Jira ticket link during business hours
- **Jira Auto-Ticket Creation** — Bot offers to create a Jira Service Request ticket automatically; user confirms via response card buttons
- **Google SSO** — Employees sign in with their Broughton Partners Google accounts via Cognito

---

## Architecture
```
Employee (Browser)
        ↓
CloudFront → S3 (Lex Web UI)
        ↓
Amazon Cognito (Google SSO)
        ↓
Amazon Lex V2
    ├── QnA Intent → S3 PDF Knowledge Base
    ├── UrgentITIssueIntent
    ├── CreateTicketIntent
    └── FallbackIntent
        ↓
AWS Lambda (ITChatbotMemoryHandler)
    ├── LangChain + Amazon Bedrock (Claude Haiku 4.5)
    ├── DynamoDB (conversation memory per user)
    └── Jira API (ticket creation via Secrets Manager)
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | AWS Lex Web UI (CloudFormation), CloudFront, S3 |
| Auth | Amazon Cognito with Google OAuth |
| Bot | Amazon Lex V2 |
| AI | LangChain + Amazon Bedrock (Claude Haiku 4.5) |
| Memory | Amazon DynamoDB |
| Knowledge Base | S3 + Lex QnA (PDF documents) |
| Ticketing | Jira REST API v3 |
| Secrets | AWS Secrets Manager |
| Runtime | AWS Lambda (Python 3.12) |

---

## How It Works

1. Employee logs in with their Broughton Partners Google account
2. They describe an IT issue in the chat
3. Lex routes the message to the appropriate intent:
   - **QnA intent** — searches internal IT PDFs and passes the result to Lambda
   - **UrgentITIssueIntent** — checks business hours and routes to phone or ticket
   - **FallbackIntent / General** — sends message + PDF context to Claude via LangChain
4. Lambda loads the user's conversation history from DynamoDB (keyed by email)
5. Claude responds using the PDF context and conversation history
6. The response and new message are saved back to DynamoDB
7. If the issue can't be resolved, the bot offers to create a Jira ticket via response card buttons

---

## Lambda Function Overview
```python
lambda_handler()                            # Routes by intent and checks Jira ticket flag
handle_urgent_issue()                       # Business hours check, offers ticket creation
handle_general_issue()                      # LangChain + Bedrock + DynamoDB memory
handle_ticket_creation()                    # Creates Jira Service Request via REST API
get_user_id()                               # Decodes Cognito JWT to extract user email
get_history() / save_history()              # DynamoDB read/write
get_awaiting_ticket() / set_awaiting_ticket()  # Jira confirmation flag
```

---

## AWS Resources

| Resource | Name / ID |
|---|---|
| Lambda | `ITChatbotMemoryHandler` |
| Lambda Layer | `langchain-bedrock` (version 4) |
| DynamoDB Table | `ITChatbotMemory` (partition key: `sessionId`) |
| Lex Bot ID | `1DPWAHDTIL` |
| Lex Alias | `Timecheckalias` (`Z1ODV2CRD2`) |
| Secrets Manager | `broughton-jira-token` |
| S3 Knowledge Base | `troubleshooting-and-policies` |

---

## Lex Intents

| Intent | Purpose |
|---|---|
| `UrgentITIssueIntent` | Routes urgent issues to phone/ticket based on business hours |
| `FallbackIntent` | Catches general messages, triggers Lambda |
| `WelcomeIntent` | Greeting handler |
| `CreateTicketIntent` | Triggered when user confirms ticket creation |
| QnA Intent | Searches S3 PDFs, passes results to Lambda via session attributes |

---

## Setup

### Lambda IAM Permissions Required

- `AmazonDynamoDBFullAccess`
- `AmazonBedrockFullAccess`
- `SecretsManagerReadWrite`

### Lambda Configuration

- **Runtime:** Python 3.12
- **Timeout:** 30 seconds
- **Memory:** 512 MB
- **Layer:** `langchain-bedrock` (includes `langchain-core`, `langchain-aws`)

### DynamoDB Table
```
Table name: ITChatbotMemory
Partition key: sessionId (String)
```

### Secrets Manager

Store your Jira API token as JSON with secret name `broughton-jira-token`:
```json
{
  "JIRA_API_TOKEN": "your-token-here"
}
```

### Update these constants in `lambda_function.py`
```python
JIRA_BASE_URL = "https://your-domain.atlassian.net"
JIRA_EMAIL    = "your-email@company.com"
JIRA_PROJECT_KEY = "IT"
```

---

## Key Design Decisions

**Why LangChain?**
LangChain's `ChatBedrock` and message classes (`HumanMessage`, `AIMessage`, `SystemMessage`) provide a clean abstraction over the raw Bedrock API and make it easy to inject structured conversation history into each prompt.

**Why DynamoDB for memory instead of session attributes?**
Lex session attributes reset between sessions and can be overwritten by the QnA intent. DynamoDB provides true persistence keyed to the user's identity, surviving across browser sessions and devices.

**Why decode the JWT for user ID?**
Using the Cognito JWT `preferred_username` (email) instead of the random Lex `sessionId` means the same employee's history is accessible regardless of which device or browser session they use.

**Why a DynamoDB flag for ticket confirmation?**
The QnA intent intercepts messages before Lambda, which means session attributes set by Lambda may not survive the round trip. Storing the `awaitingTicket` flag in DynamoDB ensures it persists reliably across intent boundaries.

---

## License

Internal use — Broughton Partners
