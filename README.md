# Aceya Systems — Auto Substack Article Generator (n8n Workflow)

An n8n workflow that picks a random topic from a curated list, generates a full
Substack-style article with an AI agent (Google Gemini), and sends the
finished draft to a Telegram chat.

## What it does

1. **1. Start Workflow** — Manual trigger to kick off a run.
2. **Topic options** — A Code node picks one random topic from a list of ~50
   pre-written automation/business topics relevant to Aceya Systems.
3. **2. Set Topic & Constraints** — Passes the chosen topic downstream.
4. **AI Agent** — Calls an LLM (Google Gemini) with a system prompt that
   defines Aceya Systems' voice: warm, storyteller-meets-strategic-advisor
   tone, no em-dashes, teases automation concepts without giving away full
   how-to steps, and always plugs aceyasystems.com.ng.
5. **Google Gemini Chat Model** — The language model connected to the AI
   Agent.
6. **Structured Output Parser** — Forces the AI Agent's output into a strict
   JSON shape: `{ title, subtitle, body }`.
7. **Parser** — A Code node reshapes that JSON into a Telegram-friendly
   payload (adds a `full_text` field).
8. **Send a text message** — Posts the formatted article to a Telegram chat
   via the Telegram node.

## Secrets removed

The following values were stripped from the workflow JSON and replaced with
placeholders. You'll need to fill these in yourself before importing/running
the workflow in n8n:

| Location | Field | Placeholder |
|---|---|---|
| Google Gemini Chat Model node | `credentials.googlePalmApi.id` | `REPLACE_WITH_YOUR_GOOGLE_GEMINI_CREDENTIAL_ID` |
| Send a text message node | `credentials.telegramApi.id` | `REPLACE_WITH_YOUR_TELEGRAM_CREDENTIAL_ID` |
| Send a text message node | `parameters.chatId` | `REPLACE_WITH_YOUR_TELEGRAM_CHAT_ID` |
| Send a text message node | `webhookId` | `REPLACE_WITH_GENERATED_WEBHOOK_ID` |
| Workflow `meta` | `instanceId` | `REPLACE_WITH_YOUR_N8N_INSTANCE_ID` |

None of these are needed to understand or share the workflow's logic, but
n8n needs real values to actually run it in your own instance.

## Setup instructions

1. **Import the workflow**
   In n8n, go to *Workflows → Import from File* and select
   `aceya-substack-workflow.json`.

2. **Add credentials**
   - **Google Gemini**: In n8n, create a new *Google Gemini (PaLM) API*
     credential with your Google AI Studio / Vertex API key, then attach it
     to the "Google Gemini Chat Model" node.
   - **Telegram**: Create a new *Telegram API* credential using a bot token
     from [@BotFather](https://t.me/BotFather), then attach it to the
     "Send a text message" node.

3. **Set your Telegram chat ID**
   Message your bot (or add it to a group), then use a tool like
   `https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates` to find the numeric
   `chat.id`. Paste that into the `chatId` field on the "Send a text message"
   node.

4. **Webhook ID**
   n8n auto-generates a fresh `webhookId` when you save the Telegram node in
   your own instance — you don't need to set this manually, just leave the
   placeholder and save the node once in the n8n editor.

5. **Update the model name (optional)**
   The workflow references `models/gemini-3.6-flash`. Check your available
   Gemini models and update this string if needed.

6. **Customize the topic list and prompt**
   - Edit the `topics` array in the "Topic options" Code node to match your
     own content pillars.
   - Edit the system prompt in the "AI Agent" node to adjust tone, company
     name, or target URL (currently set to `https://www.aceyasystems.com.ng`).

7. **Run it**
   Click "Start Workflow" (manual trigger) to generate and send one article.
   To automate this, replace the Manual Trigger node with a Schedule Trigger
   (e.g. daily/weekly) once you're happy with the output.

## Notes

- The AI Agent is instructed to tease concepts without revealing full
  step-by-step implementations, and to always end with a CTA pointing to
  Aceya Systems and future posts.
- Output is strictly parsed as JSON (`title`, `subtitle`, `body`) to keep the
  downstream Telegram formatting reliable.
