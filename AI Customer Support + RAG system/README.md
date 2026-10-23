# AI Customer Support + RAG System

An n8n workflow portfolio project that demonstrates an AI-powered
website support flow. It classifies incoming questions, routes them to a
suitable support path, and uses retrieval-augmented generation (RAG) to
answer questions from a knowledge base. The workflow also includes
separate paths for order/account questions, billing, troubleshooting,
human escalation, and unknown intents.

> **Portfolio/demo project:** Configure your own credentials, database
> IDs, Telegram destination, and webhook URL before using this workflow.
> The included export is sanitized and is not ready to run until those
> items are configured.

## Workflow overview

![AI Customer Support + RAG workflow overview](workflow-screenshot.png)

*The screenshot above shows the overall workflow layout in n8n,
including the intent classifier, knowledge retrieval, support branches,
and data-loading flow.*

## What it does

1.  **Receives a message** --- the Webhook receives a chat request from
    the website or another client.
2.  **Normalizes the input** --- the `Edit Fields` node maps fields such
    as `message`, `session_id`, `channel`, `page_url`, and optional
    `customer_email` into a consistent structure.
3.  **Classifies the intent** --- a Basic LLM Chain uses a Groq chat
    model and a Structured Output Parser to return an intent label.
4.  **Routes the request** --- the Switch node sends the request to a
    matching support path.
5.  **Responds using the relevant path** --- the workflow includes:
    -   **Knowledge base:** retrieves relevant content from a Supabase
        vector store and uses an AI agent to draft an answer.
    -   **Order/account:** extracts order details, checks for matching
        records through the configured Notion data source, and prepares
        a response for found or missing records.
    -   **Billing:** retrieves billing-related context from a Supabase
        vector store and uses an AI agent to respond.
    -   **Troubleshooting:** retrieves troubleshooting context from a
        Supabase vector store and uses an AI agent to suggest next
        steps.
    -   **Human escalation:** prepares an escalation message and uses
        Telegram-related nodes for the configured support handoff/reply
        flow.
    -   **Unknown intent:** uses a separate AI agent to handle requests
        that do not match the defined categories.
6.  **Returns the response** --- the `Respond to Webhook` node sends the
    workflow's response back to the requesting client.

The exact response content depends on the prompts, records,
knowledge-base content, and credentials configured in your own n8n
instance.

## Intent categories

  -----------------------------------------------------------------------
  Intent                              Purpose
  ----------------------------------- -----------------------------------
  `knowledge_base`                    Product, feature, pricing,
                                      integration, and policy questions
                                      that can be answered from the
                                      knowledge base.

  `order_account`                     Questions that require checking
                                      order or account-related records.

  `billing`                           Billing, charge, refund, invoice,
                                      or cancellation questions.

  `troubleshooting`                   Technical issues and requests for
                                      help resolving errors.

  `human_escalation`                  Cases that should be passed to a
                                      human support contact.

  `unknown`                           Requests that do not clearly fit
                                      another category.
  -----------------------------------------------------------------------

## RAG knowledge-base loading flow

The workflow includes a separate form-triggered data-loading path:

1.  Open the **Upload your file here** form and submit a supported
    document (`.pdf` or `.csv`, according to the form configuration).
2.  The **Default Data Loader** extracts document content.
3.  The **Recursive Character Text Splitter** divides the content into
    smaller chunks.
4.  **Embeddings Google Gemini** converts the chunks into embeddings.
5.  **Store data** writes the chunks and embeddings into the configured
    Supabase vector store.

The retrieval paths use the same embedding model/settings as the loading
flow. Keeping embedding configuration consistent helps ensure that
stored vectors can be searched correctly.

## Main components

-   **n8n** --- orchestrates the workflow and connects the branches.
-   **Groq chat model** --- powers intent classification and the
    AI-agent response paths.
-   **Google Gemini embeddings** --- creates vector embeddings for
    knowledge retrieval.
-   **Supabase Vector Store** --- stores and retrieves embedded
    knowledge-base content.
-   **Notion** --- provides the configured data source for order/account
    lookup.
-   **Telegram** --- supports the configured human-escalation and
    support-reply flow.
-   **Website webhook client** --- sends a message to the workflow and
    receives the response.

## Setup checklist

1.  Import the sanitized workflow JSON into your n8n instance.
2.  Reconnect the required credentials for Groq, Google Gemini,
    Supabase, Notion, and Telegram.
3.  Replace all placeholder values, including the Telegram chat ID and
    Notion data-source/database ID, with your own values.
4.  Check the Supabase vector-store table names and dimensions against
    the embedding model configuration used by the workflow.
5.  Review the prompts, classifier labels, Switch conditions, and node
    connections. Make sure the intent values match exactly, including
    lowercase spelling (for example, `knowledge_base`).
6.  Configure the website or client to call your own webhook URL. Do not
    use an n8n test webhook URL for a live deployment.
7.  Test each route with representative inputs, including an unknown
    request and a human-escalation case.
8.  Confirm that the webhook returns the expected response and that no
    private data is exposed in logs or replies.

## Example test messages

-   **Knowledge base:** "What is ForgeFlow?"
-   **Order/account:** "Can you check the status of my order?"
-   **Billing:** "Why was I charged?"
-   **Troubleshooting:** "The integration is returning an error. What
    should I try?"
-   **Human escalation:** "I need help with a sensitive account-access
    issue."
-   **Unknown:** "Can you write a poem about the ocean?"

Use test records and non-sensitive sample data when validating the
workflow.

## Security notes

-   This repository version is intended for sharing as a portfolio
    example.
-   Do not commit API keys, access tokens, credential exports, private
    webhook URLs, customer records, or real chat IDs.
-   Credentials are removed from the sanitized workflow export, so you
    must reconnect them after importing.
-   Review every node and all other files in the repository before
    making the repository public. Sanitizing this JSON does not
    automatically sanitize other files.

## Limitations and customization

-   The workflow depends on external services and credentials that are
    not included.
-   Order/account lookup depends on the Notion data source and data
    structure you configure.
-   Human escalation depends on your Telegram setup and how you want
    support staff to respond.
-   RAG answers depend on the quality and coverage of documents uploaded
    to the vector store.
-   This repository describes the configured workflow design; results
    and reliability should be validated in your own environment before
    production use.

## Repository files

-   `README.md` --- project overview, workflow explanation, setup
    checklist, and test examples.
-   `AI_Customer_Support_RAG_GitHub_Sanitized.json` --- sanitized n8n
    workflow export.
-   `workflow-screenshot.png` --- screenshot of the workflow inside n8n.

## Reference

-   [n8n documentation: Retrieval-Augmented Generation
    (RAG)](https://docs.n8n.io/advanced-ai/rag-in-n8n/)
