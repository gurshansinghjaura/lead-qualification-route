# Lead Qualification Router

## What it does
An AI system that scores and routes inbound leads the moment they submit a form — separating the leads worth calling immediately from the ones that can wait, instead of a human manually reading and prioritizing every submission.

## Demo
🎥 [Watch the 4-minute walkthrough](YOUR_LOOM_LINK_HERE)

## Architecture
A form submission (name, email, company, automation need, budget range, timeline) triggers an AI agent (Claude Sonnet 4.6) that scores the lead 0-100 based on stated budget, urgency language, how specific the automation need is, and whether the business fits the target niche. The agent returns a score, a category (Hot/Warm/Cold), and a one-line reason. That output is parsed out of the model's response and used to branch the workflow: every lead is logged to Google Sheets regardless of category, Hot leads trigger an immediate Slack alert to the sales channel with the lead's details and the reason they scored high, and Warm/Cold leads receive an automated thank-you email confirming their submission was received.

![n8n canvas screenshot](YOUR_SCREENSHOT_HERE)

## Key technical decisions
- Gave the AI agent an explicit, weighted scoring rubric (budget stated and realistic, urgency language, specificity of the need, business-type fit, with "just exploring + vague budget" as a negative signal) rather than an open-ended "rate this lead" prompt, so scores are consistent across submissions.
- Forced the model to respond in a strict JSON format only, then parsed that in a dedicated code step — stripping markdown code fences defensively — to make the score/category/reason machine-actionable for the switch node instead of relying on free text.
- Split routing into three distinct outcomes (Hot/Warm/Cold) with different actions per tier, so the sales team's attention goes to the leads worth an immediate reply instead of a single generic notification for every submission.

## Tech stack
n8n, Claude Sonnet 4.6, Google Sheets API, Slack API, Gmail API

## A real bug I fixed
The AI agent's raw output sometimes came wrapped in markdown code fences (```json ... ```) even when the prompt asked for JSON only, which broke a plain `JSON.parse()` on the first character. I added a stripping step that removes ```json and ``` markers before parsing, so the score/category/reason extraction doesn't silently fail — and fail-fast is worse here than fail-loud, because a broken parse meant a lead that never got logged or routed at all.

## Setup
1. Import `lead_qualifier_p1.json` into n8n.
2. Reconnect credentials — Google Sheets, Slack, Gmail, and Anthropic (Claude). The exported file uses placeholder credential references (`YOUR_CREDENTIAL_ID`); n8n does not export real API keys.
3. Point your lead-capture form at the `LeadForm` trigger, or swap it for your own form tool's webhook.
4. Activate the workflow.
