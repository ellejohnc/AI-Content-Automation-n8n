# AI-Content-Automation-n8n

A multi-client AI content automation engine built on n8n. It produces video, static graphics and carousels, and publishes them to social platforms, with a human approval gate before every credit-heavy step.

This repository documents the architecture, design decisions and lessons from the build. It contains no client data, credentials or IDs.

## What it does

- Produces short-form video, static posts, carousels, infographics and ebooks from approved ideas
- Runs many clients and brands from one set of parameterised workflows
- Publishes through an approval queue instead of paid third-party posting apps
- Guards spend with locks, caps and alerts

## Architecture

- n8n handles all deterministic work: generation steps, polling, webhooks, publishing, alerts
- An AI agent tool handles judgment work on a schedule: selecting, scoring, planning, supervising
- One canonical set of parameterised workflows reads per-client config from a control base (a Clients table and a Client_Outputs table keyed "<Client>::output_type" with Config_JSON)
- Onboarding a client means adding config rows and duplicating schema-identical content bases, never forking workflows
- Config precedence: record override, then client config, then fallback default
- Airtable is the queue and review layer; Drive is the single source of truth for files

## Approval gates

- Video: idea approval, then script approval, before avatar, B-roll and motion graphics
- Statics: one approval before generation
- Agent tasks propose and move approved records through the existing status chain; they never act alone
- Approvals can be tick boxes inside the agent chat, so reviewers never open the database

## Publishing

- Content enters the publishing queue only when final
- Imported rows land as Ready for Review; changing to Ready for Publishing is the approval
- n8n posts on database webhooks, following Scheduled_For on the queue row
- Carousels publish from the PDF only (PDF for LinkedIn, converted JPEG pages for Instagram)
- After posting, the whole content folder moves to the brand's Posted folder

## Async agent jobs

- Dispatch keeps the processing status; a separate poller marks Ready only after the final JSON parses and the expected files exist
- A fixed JSON output contract is appended to every agent prompt
- Aspect ratio is pinned in graphic prompts, since agents default to 16:9

## Spend guards

- Lock before render, one item per run, per-record and daily caps from config
- Spend watchdog email, alerts to a shared channel
- Wait between dispatch and check; reuse existing clip folders; no automatic re-queue of errors
- Keep at least 14 days of n8n execution history

## n8n MCP edit sequence

1. Close the workflow's browser editor tab
2. Edit through MCP
3. Re-apply the Drive upload operation in a separate batch
4. Rebind credentials by hand in the editor
5. Publish
6. Verify activeVersionId matches versionId

## Stack

n8n, Airtable, AI agent platform, Claude and OpenAI models, Google Drive, HyperFrames and FFmpeg for video, Apify and Supadata for source data.

## Status

Reference documentation. Treat every item as a proven default to re-confirm per client, not as a fixed rule.

## License

MIT
