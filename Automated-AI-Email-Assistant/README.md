# AI Email Assistant Workflow

<a href="https://www.loom.com/share/cfe408a4bb984cd2bc69670fe070b53a">
<img width="1365" height="619" alt="image" src="https://github.com/user-attachments/assets/84fb98fe-4b33-415b-8b83-efcb13f54e2f" />
</a>

## Objective
Automatically process incoming Gmail messages, summarize them, draft an AI-generated reply when needed, and route the draft for human approval via Telegram before it's ever sent — keeping a human in the loop while removing the manual triage work.

## How It Works

1. **Ingest** — A Gmail Trigger polls the inbox every minute for new unread messages.
2. **AI Processing** — The email is passed to a Groq-hosted LLM (`openai/gpt-oss-20b`), which returns a structured JSON output containing a one-line summary, a Yes/No on whether a reply is needed, and a suggested HTML-formatted reply if so.
3. **Decision Filter** — An IF node checks the `reply_needed` field:
   - **No** → Sends a Telegram notification with just the summary, no action needed.
   - **Yes** → Continues to the approval step below.
4. **Draft & Human Approval** — If a reply is needed, the workflow creates a Gmail draft (threaded to the original email) and sends a Telegram message with the summary and suggested reply, along with **Approve / Disapprove** buttons.
5. **Action** — Based on the Telegram response:
   - **Approved** → Calls the Gmail API directly to send the drafted reply.
   - **Disapproved** → Takes no further action (the draft remains in Gmail for manual editing if desired).

## Stack
- **n8n** — workflow orchestration
- **Groq (gpt-oss-20b)** — LLM summarization & reply drafting, free-tier model
- **Gmail API** — trigger, draft creation, sending
- **Telegram Bot API** — human-in-the-loop approval interface

## Why this design
Rather than auto-sending AI replies blindly, this workflow keeps a human approval gate in the loop via Telegram — a practical pattern for real-world AI automation where accuracy and trust matter more than full autonomy.
