# Lead Enrichment with an AI Agent (n8n)

An n8n workflow that takes a raw contact-form submission, checks that the lead is real, researches the person and company on the web, scores the lead, and then asks me on Telegram whether to keep it or drop it.

I built this as a practical project to get comfortable with AI agents, tool calling, and human-in-the-loop approvals in n8n.

src="<img width="1365" height="629" alt="ai pt2" src="https://github.com/user-attachments/assets/8559e1f8-14c0-461d-a728-06c7b6f692c9" />


## The problem

Anyone who has run a contact form knows the drill. Some submissions are great, some are typos, some are bots, and every one of them needs someone to google the name and company before deciding whether it's worth a reply. That's slow and repetitive, and it's exactly the sort of thing a workflow can do for you.

This workflow does the boring first pass and leaves the final call to a person.

## How it works

1. **A form gets submitted.** A webhook receives the data: name, work email, company, phone number, what they want to solve, and a consent checkbox.

2. **Consent check.** If the person didn't tick "I agree to be contacted", they're logged in a separate Airtable table (`Does Not Consent`) with a note and the flow stops there. I didn't want to enrich or store anyone who hadn't agreed to be contacted.

3. **Email check.** Hunter verifies whether the email is actually deliverable. The result is merged back with the original form data. If the email isn't deliverable, the lead goes to the same `Does Not Consent` table with the note "Email undeliverable".

4. **AI research.** The remaining leads go to an AI agent (Groq, `openai/gpt-oss-20b`, temperature 0). It has one tool, a Tavily web search, and a strict budget of two searches:
   - Search 1: does this person exist and work at this company?
   - Search 2: how big is the company? (Skipped if search 1 already answered it.)

   The prompt tells the agent not to guess. If it can't verify something, it says so, and company size comes back as `Unknown` instead of an estimate.

5. **Structured output.** The agent has to return a fixed JSON shape: the original fields plus `Lead Score` (High / Medium / Low), a short `Score Reasoning`, and `Enriched Company Size`. A structured output parser enforces that format.

6. **Routing by score.**
   - **Low** goes straight into Airtable with the status `Low priority`. No human time spent.
   - **Medium and High** go into Airtable with the status `New`, then move on to a review.

7. **Human approval on Telegram.** I get a message with the lead's details, the score, and the AI's reasoning, with Approve / Disapprove buttons. The workflow waits until I answer.

8. **Final status.** Approve sets the Airtable record to `Qualified`. Disapprove sets it to `Disqualified`.

## What it's built with

| Tool | What it does here |
| --- | --- |
| [n8n](https://n8n.io) | Runs the whole workflow |
| Webhook | Receives the form submission |
| [Hunter](https://hunter.io) | Email deliverability check |
| [Groq](https://groq.com) | The language model behind the agent |
| [Tavily](https://tavily.com) | Web search tool for the agent |
| [Airtable](https://airtable.com) | Stores leads and their statuses |
| Telegram | Sends the approval request and waits for my reply |

## Setting it up

You'll need accounts (free tiers are fine for testing) for Hunter, Groq, Tavily, Airtable and a Telegram bot.

1. In n8n, go to **Workflows → Import from file** and choose `Lead_enrichment_using_AI_agent_sanitized.json`.
2. Create credentials for Airtable, Hunter, Groq and Telegram, and attach them to the nodes that need them. n8n will highlight anything that's missing.
3. Replace the placeholders I left in the file:
   - `YOUR_AIRTABLE_BASE_ID`, `YOUR_AIRTABLE_TABLE_ID`, `YOUR_AIRTABLE_VIEW_ID` in the Airtable nodes (easiest way: reselect your base, table and view from the dropdowns).
   - `YOUR_TELEGRAM_CHAT_ID` in the Telegram node.
   - `YOUR_TAVILY_API_KEY` in the `Internet Research` node. Better still, move it into a Header Auth credential so the key never sits in the workflow itself.
4. Set up your Airtable base with two tables:
   - **All Leads:** `Full Name`, `Work Email`, `Company Name`, `Phone Number`, `What are you looking to solve`, `Lead Score`, `Score Reasoning`, `Enriched Company Size`, `Date Submitted`, `Status`
   - **Does Not Consent:** `Full Name`, `Work Email`, `Company Name`, `Date Submitted`, `Note`
5. Point your form at the webhook URL and send a test submission.

One thing that might trip you up: the form field names in the workflow match my form exactly, including some stray spaces (for example `"  Work Email  "`). If your form sends different names, update the expressions in the Webhook-related nodes to match.

## Things I'd improve next

I'd rather be upfront about where this is still rough:

- **Approval can update the wrong record.** After I approve or reject, the workflow looks up an Airtable record with `Status = New` and updates it. That's fine when leads trickle in one at a time, but if two are waiting at once, it could update the wrong one. The fix is to carry the record ID from the earlier upsert, or match on the email address.
- **Only "deliverable" emails pass.** Hunter also returns results like "risky", and those are currently treated the same as undeliverable.
- **Medium and High leads get the same treatment.** Both go to me for approval. It might make sense to auto-approve strong High leads or notify a teammate for them.
- **Research is deliberately shallow.** Two searches with two results each keeps it cheap and fast, but small companies often come back as `Unknown` for size.
- **No error handling yet.** If an API call fails, the run just stops. Adding an error workflow that pings me would make this safer to run for real.

## A note on the JSON file

The workflow file here has been cleaned before publishing: credentials, API keys, IDs, webhook URLs and test data were removed or replaced with placeholders. It won't run until you add your own.

## License

Feel free to use this as a starting point for your own projects. If you build on it, I'd love to hear how it went.
