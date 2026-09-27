# REPORT — New Enquiry Intake (n8n workflow)

Workflow name: **New Enquiry Intake**
Workflow id: `NXve0XT8qAEjlaU4`
URL: https://n8npravin.app.n8n.cloud/workflow/NXve0XT8qAEjlaU4
Project: personal — `Pravin <mailprp@gmail.com>` (`WKcZBJSSu3nNSRA1`)

## Status per part
- Form (name, phone, email, practice-area dropdown, source dropdown): DONE
  evidence: `n8n_validate_workflow` -> `{"valid":true,"nodeCount":7}`
- Edge validation (name present; phone OR email present; any phone exactly 10 digits; practice area present): DONE
  evidence: execution 10 -> `"valid":false, problems:["Name is missing.","Practice area is missing.","Provide a phone number or an email address."]`
  evidence: execution 12 -> `"valid":false, problems:["Phone must be exactly 10 digits (the number given has 5)."]` (email was present, phone was not 10 digits)
  evidence: execution 11 -> `"valid":true, problems:[]`
- Good enquiry written to intake sheet with date, practice area and source (plus name/phone/email): DONE for routing, BLOCKED for the real write
  evidence: execution 11 -> `Write to intake sheet` executionStatus success, lastNodeExecuted `Show success page` (the Sheets call was pinned, not a live write)
- Bad enquiry down a separate error branch, person told which field to fix: DONE
  evidence: executions 10 and 12 -> `Is the enquiry complete?` output index 1, `Log rejected enquiry` success, lastNodeExecuted `Show what to fix`
- Google Sheets connection + spreadsheet/tabs: BLOCKED — no Google Sheets credential exists in the n8n instance and no spreadsheet was chosen
  evidence: `n8n_list_credentials` -> only `DeepSeek account` (deepSeekApi); `googleSheetsOAuth2Api` -> `{"data":[],"count":0}`
- Live form page and Form Ending pages rendering in a browser: UNVERIFIED — tests used pinned trigger data, not a real form submission

## What broke and how I fixed it
- First validation pass warned: `Node "Enquiry Form": Field "parameters.responseMode": Must be an n8n expression (={{...}})`. I had assumed `responseMode: 'responseNode'`. The official n8n docs for the Form Trigger list only "Form Is Submitted" (`onReceived`) and "Workflow Finishes" (`lastNode`), and the Form node's "Form Ending" page type is the ending mechanism. Changed to `responseMode: 'lastNode'`; re-validated clean (`valid:true`, no warnings).

## Claims ledger
- Workflow "New Enquiry Intake" exists with id NXve0XT8qAEjlaU4 — proven by `n8n_create_workflow_from_code` response and by three executions run against that id.
- Rejects a record with any required field missing and saves nothing — proven by executions 10 and 12 (false branch, nothing routed to the intake sheet).
- A supplied phone must be exactly 10 digits, even when an email is present — proven by execution 12.
- Writes date, name, phone, email, practice area and source on the good path — routing proven by execution 11; the actual Google Sheets write is UNVERIFIED (pinned).
- Sends a rejected enquiry to a separate Errors tab — UNVERIFIED (Sheets pinned).
- The person is shown exactly which field to fix — the Form Ending node is reached on the false branch (proven), but the rendered page text is UNVERIFIED until viewed in a browser.

## What I would tell the next person
1. Connect Google Sheets first (OAuth2). Both Sheets nodes currently reference a credential named "Google Sheets"; point both at the one connection.
2. In the workflow, set `Write to intake sheet` to your spreadsheet, tab `Enquiries`; set `Log rejected enquiry` to the same spreadsheet, tab `Errors`.
3. Tab headers must match exactly — Enquiries: `Date | Name | Phone | Email | Practice Area | Source`; Errors: `Date | Name | Phone | Email | Practice Area | Source | Errors`.
4. Publish the workflow, then use the Form Trigger's Production URL for the website and answering service.
5. The phone rule is deliberately strict: if a phone is given it must be 10 digits, even when an email is also given. Dropdowns have a leading "Please choose" option so a missing practice area or source is genuinely detectable.


# REPORT — CounselGrid Front Desk Chatbot (n8n workflow)

Workflow name: **CounselGrid Front Desk Chatbot**
Workflow id: `fFL2RWBt1HiD0ugr`
Editor URL: https://n8npravin.app.n8n.cloud/workflow/fFL2RWBt1HiD0ugr
Public chat link: https://n8npravin.app.n8n.cloud/webhook/b4512682-3883-4f09-89f1-03f1c832515e/chat
Project: personal — `Pravin <mailprp@gmail.com>`
Nodes: Chat Trigger (public, hosted chat) -> AI Agent, with DeepSeek Chat Model and Simple Memory attached as subnodes.

## Status per part
- Chat Trigger is public and the workflow is published: DONE
  evidence: `n8n_publish_workflow` -> `{"success":true,"activeVersionId":"fa0d96aa-9a05-45fb-9e4e-a667a7550100"}`; `n8n_get_workflow_details` -> `"active":true`
- Public chat link served: DONE
  evidence: `webfetch https://n8npravin.app.n8n.cloud/webhook/b4512682-3883-4f09-89f1-03f1c832515e/chat` -> n8n chat HTML (`<title>Chat</title>`, `createChat({ mode: 'fullscreen', webhookUrl: "https://n8npravin.app.n8n.cloud/webhook/b4512682-3883-4f09-89f1-03f1c832515e/chat" ...})`)
- DeepSeek model connected: DONE
  evidence: `n8n_create_workflow_from_code` -> `autoAssignedCredentials: [{ credentialName: "DeepSeek account", credentialType: "deepSeekApi", source: "user" }]`
- Simple Memory connected to the Agent: DONE
  evidence: workflow details connection `"Simple Memory":{"ai_memory":[[{"node":"Front Desk Agent",...}]]}`; executions 13-15 tracing shows `"ai.agent.memory.loads":1,"ai.agent.memory.saves":1`
- Answers only from the FAQ: DONE (sampled)
  evidence: execution 13 -> the price/setup answer matches FAQ 3, 4 and 7
- Follow-up capture: DONE after one fix
  evidence: execution 15 -> `"Would you like the owner to reach out and walk you through it? If so, what's your name?"`
- Memory retention across turns in one live session: UNVERIFIED — each MCP test run gets a fresh session id, so continuity across messages was not exercised here; the widget passes a stable session id in real use

## What broke and how I fixed it
- Execution 14 asked "Would you like the owner to get back to you?" but did not ask for name or contact. Part 5 of the system message said to ask for name and phone/email. Changed part 5 to "Then, in the same reply, ask for their name and their phone number or email so the owner can reach them", re-published, and re-tested. Execution 15 now asks for the name.
- Note: the Chat Trigger's own `responseMode` remains `streaming` (n8n's preferred mode for Agent-backed chat), and the Agent streams to the widget.

## Claims ledger
- Workflow exists and is active — proven by `n8n_create_workflow_from_code`, `n8n_publish_workflow` and `n8n_get_workflow_details` (`"active":true`).
- The public link serves the chat UI — proven by `webfetch` returning the n8n chat page with that webhookUrl.
- DeepSeek replies — proven by executions 13-15 (`llm.tokens.in`/`out` recorded, `deepseek-chat`).
- Memory is attached and used each run — proven by the connection graph and `ai.agent.memory.loads/saves`.
- Multi-turn recall within one chat session — UNVERIFIED (see above).

## What I would tell the next person
1. The link is public. Anyone with it can chat; each run costs DeepSeek tokens. Publish/unpublish from the editor to take it offline.
2. The bot captures a name and phone/email in the chat only. There is no node writing those details to storage, because the build was limited to Chat Trigger, AI Agent, DeepSeek and Simple Memory. Add a storage branch if you want the leads saved.
3. Simple Memory is in-process and per-session; it is not durable across n8n restarts and is not a lead database.
4. To change what the bot knows, edit the FAQ text inside the Front Desk Agent system message (part 6) and re-publish.
