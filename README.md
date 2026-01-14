# **AI-Powered IT Knowledge Assistant**

_A smart, internal AI chatbot that helps employees access IT policies, FAQs, and troubleshooting guidance before submitting tickets. Built using Amazon Lex, Amazon Bedrock, and S3, leveraging retrieval-augmented generation (RAG) to provide accurate, natural-language responses from your own internal documentation._

# Features

**Conversational AI:** Ask questions in natural language about IT policies and procedures.
**RAG-powered responses:** AI grounded in internal S3-hosted documents ensures answers are accurate and up-to-date.
**Knowledge base integration:** Centralized repository of IT policies, FAQs, and troubleshooting guides.
**Deployment-ready:** Built with AWS best practices, secure access, and scalable architecture.



# Tech Stack

**Cloud & AI:** Amazon Bedrock (Titan Embeddings & Claude v2), Amazon Lex, S3
**Backend & Automation:** AWS Lambda, Python
**Optional Frontend:** Custom web UI or internal portal integration
**Architecture Concepts:** RAG, serverless, role-based access, conversational AI



# Getting Started

**Prerequisites**

-An AWS account with access to Amazon Bedrock
-Access to Titan Embeddings G1 and Anthropic Claude v2 models on Bedrock
-Four example PDFs of IT policies and FAQs (replace with your own documents)

_-Optional: web UI for the bot (AWS sample repo)_




# Setup Overview

**Upload Documents to S3:** Store IT policies, FAQs, and troubleshooting guides in an S3 bucket.
**Create Knowledge Base in Bedrock:** Configure a Bedrock knowledge base to point to the S3 bucket.
**Build Lex Chatbot:** Connect Lex to Bedrock using a custom intent (QnAIntent) to handle queries.
**Test & Deploy:** Ensure natural-language responses work as expected. Optional: integrate into internal portal or web UI.
**Clean-up:** Delete resources when testing is complete to avoid unnecessary costs.


# How It Works

Employees ask questions via Amazon Lex chatbot --> Lex forwards queries to Bedrock --> Bedrock uses RAG to retrieve context from S3 documents --> The AI returns accurate, context-aware responses.

# Impact

**Reduced repetitive tickets:** Employees can self-serve answers without waiting for IT support.
**Improved efficiency:** Faster resolution of common IT questions.
**Scalable:** Easily expandable to include new IT policies or departments.

**Usage Example**
**User:** "How do I reset my Entra password?"
**Bot:** "To reset your Entra password, navigate to [link] and follow the steps outlined in our internal IT policy guide."

# Future Enhancements

Add analytics to track most common questions and update FAQs automatically

Expand to other internal departments (HR, Finance)

Integrate with Slack, Teams, or internal portal for seamless access

# References

AWS Bedrock Documentation: https://aws.amazon.com/bedrock/
AWS Lex Documentation: https://aws.amazon.com/lex/
RAG Concept Overview: https://www.amazon.com/blogs/aws/
