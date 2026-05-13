---
name: talentsourcer-ai
description: Use TalentSourcer AI CLI/API to manage recruiting projects, briefings, candidates, shortlists, campaigns, analytics, and outreach-task workflows.
---

# TalentSourcer AI

Use this skill when the user wants to work with TalentSourcer AI projects, briefings, candidate requirements, campaigns, analytics, outreach tasks, or recruiting workflows through the CLI.

TalentSourcer AI is the recruiting system of record and orchestration layer. The CLI is the preferred interface for agents because it returns structured JSON with IDs, URLs, and navigation targets.

## Operating Rules

- Use the TalentSourcer CLI for TalentSourcer project, briefing, candidate, shortlist, campaign, analytics, and outreach-task actions.
- Prefer `--json` for all CLI commands when acting as an agent.
- Do not ask the user to paste personal access tokens into chat.
- If authentication is needed, instruct the user to run `talentsourcer auth token set` locally.
- Use returned IDs for follow-up commands.
- Use returned URLs when the user should review or open something in TalentSourcer AI.
- Do not automate or send LinkedIn messages from TalentSourcer backend capabilities.
- If local browser automation is involved, only mark work complete after the local/user-controlled action has actually happened.

## Generated Payload Files

Some CLI commands require JSON files via `--from-file`, such as briefing requirements, candidate imports, shortlist candidate-check imports, and campaign flows.

When you need to create one of these files:

- Prefer a workspace-local folder named `.talentsourcer-agent/`.
- Use descriptive filenames like `.talentsourcer-agent/video-editor-requirements.json` or `.talentsourcer-agent/campaign-flow-v1.json`.
- Do not use agent-specific folders like `.opencode/`, `.claude/`, or `.cursor/` unless the user explicitly asks.
- Do not put personal access tokens, credentials, secrets, or private environment values in these files.
- Treat generated payload files as temporary working artifacts unless the user asks to keep or commit them.
- If the current workspace is not writable, use an OS temp directory and tell the user where the file was written.
- After a successful command, mention the source file path briefly only when useful for auditability or reuse.

## CLI Setup

First check whether the CLI is installed:

```bash
talentsourcer --help
```

If the command is missing, install it:

```bash
npm install -g @talentsourcer/cli
```

Authenticate with a personal access token:

```bash
talentsourcer auth token set
```

Then verify authentication:

```bash
talentsourcer auth status --json
```

Upgrade the CLI after a new release:

```bash
talentsourcer upgrade
```

Use `talentsourcer upgrade --yes` for non-interactive environments. In interactive terminals, `talentsourcer upgrade` offers to update the local agent skills by running the standard installer:

```bash
npx skills add talentsourcer-ai/talentsourcer-ai-skill
```

The skills installer asks which agents to install for and whether to install globally or for the current project.

The user can create a personal access token in TalentSourcer AI settings under Personal Access Tokens. For project and briefing work, the token needs these scopes:

- `projects:read`
- `projects:write`
- `briefings:read`
- `briefings:write`

For search/candidate import work, the token also needs these scopes:

- `candidates:read`
- `candidates:write`
- `shortlists:read`
- `shortlists:write`
- `campaigns:read`
- `campaigns:write`
- `outreach_tasks:read`
- `outreach_tasks:write`

## Project Commands

List projects:

```bash
talentsourcer projects list --json
```

Get one project:

```bash
talentsourcer projects get <projectId> --json
```

Create a project:

```bash
talentsourcer projects create \
  --title "Senior Backend Engineer" \
  --company "Acme" \
  --location "Berlin, Germany" \
  --json
```

Update a project:

```bash
talentsourcer projects update <projectId> \
  --title "Senior Backend Engineer" \
  --company "Acme" \
  --description "Backend engineering role focused on product infrastructure." \
  --location "Berlin, Germany" \
  --salary-range "EUR 90k-120k" \
  --status Active \
  --json
```

Allowed project statuses:

- `Draft`
- `Active`
- `Paused`
- `Archived`

## Search And Candidate Commands

List searches for a project:

```bash
talentsourcer searches list --project <projectId> --json
```

Get one search:

```bash
talentsourcer searches get <searchId> --json
```

Add candidates to a search from JSON:

```bash
talentsourcer searches add-candidates <searchId> \
  --from-file candidates.json \
  --json
```

Adding candidates upserts global candidate profile fields, attaches candidates to the search, and stores organization-specific `notes` and `resumeData` for candidate checks. It does not automatically run candidate checks.

Update organization-scoped private candidate context for an existing candidate:

```bash
talentsourcer candidates update-context <candidateId> \
  --from-file candidate-context.json \
  --json
```

Use this when an agent has already imported or identified a candidate and later finds additional ATS/recruiter context. This does not automatically run candidate checks; users should start candidate checks in the TalentSourcer UI when they are ready to spend credits.

Private context guidance for agents:

- `notes`: short recruiter/ATS context, ideally 3-8 role-relevant bullets. Use for call notes, motivation, availability, compensation expectations, dealbreakers, agency observations, and other non-public context that helps evaluate this role.
- `resumeData`: concise factual CV/resume/ATS profile summary. Use for relevant work history, projects, skills, education, certifications, and ATS profile facts.
- Do not paste raw ATS exports, full transcripts, email threads, or unrelated personal data. Summarize relevant evidence and keep it compact.
- If an ATS MCP or private source is connected and the user asks to import or evaluate candidates, proactively include concise `notes` and/or `resumeData` when useful. If the private data is large or sensitive, ask whether to add a brief role-relevant summary first.
- This context may be used as private candidate-check context only for BYOK candidate checks after the user starts checks in the UI.

Candidate JSON contract:

```json
{
  "candidates": [
    {
      "fullName": "Alex Morgan",
      "title": "Senior Backend Engineer",
      "currentEmployer": "Example Cloud",
      "location": "Berlin, Germany",
      "linkedinUrl": "https://www.linkedin.com/in/alex-morgan-example",
      "email": "alex.morgan@example.com",
      "notes": "Agent-sourced note visible to this organization.",
      "resumeData": "Resume/profile summary to use during candidate checks.",
      "sourceProvider": "agent",
      "sourceCandidateId": "alex-morgan-example"
    }
  ]
}
```

Candidate context JSON contract:

```json
{
  "notes": "- Open to Berlin hybrid roles\n- Prefers backend/platform work\n- Available after 4 weeks notice",
  "resumeData": "Senior backend engineer with 7 years of Go and distributed systems experience. Built event-driven billing infrastructure at Example Cloud. BSc Computer Science."
}
```

Each candidate should include at least one stable identity signal: name, LinkedIn URL, email, or `sourceProvider` plus `sourceCandidateId`. Prefer LinkedIn URL or external source IDs to avoid duplicate candidates.

## Candidate Check Review

List candidate checks for a project with pagination:

```bash
talentsourcer candidate-checks list \
  --project <projectId> \
  --status Completed \
  --category A \
  --limit 50 \
  --json
```

Optional filters:

- `--briefing <briefingId>`
- `--status Pending|In Progress|Completed|Failed`
- `--category A|B|C|NO_FIT`
- `--bookmarked true|false`
- `--cursor <continueCursor>`
- `--limit <1-100>`

Use `continueCursor` from the JSON response to fetch the next page until `isDone` is true.

## Shortlist Commands

List shortlists for a project:

```bash
talentsourcer shortlists list --project <projectId> --json
```

Create a shortlist:

```bash
talentsourcer shortlists create \
  --project <projectId> \
  --name "Top candidates" \
  --description "Candidates selected by agent review" \
  --json
```

Update a shortlist:

```bash
talentsourcer shortlists update <shortlistId> \
  --name "Priority outreach" \
  --description "A and high-B candidates ready for outreach" \
  --json
```

Move candidate checks forward into a shortlist:

```bash
talentsourcer shortlists add-candidate-checks <shortlistId> \
  --from-file shortlist-candidate-checks.json \
  --json
```

Input JSON:

```json
{
  "candidateCheckIds": ["candidateCheckId1", "candidateCheckId2"],
  "notes": "Moved forward by agent after reviewing completed candidate checks."
}
```

When moving candidates forward, prefer completed checks with category `A` or strong `B` unless the user gives different selection criteria. Do not shortlist failed, pending, or in-progress checks unless the user explicitly requests that behavior.

## Campaign Commands

List campaigns for a project:

```bash
talentsourcer campaigns list --project <projectId> --json
```

Create a draft campaign with a flow:

```bash
talentsourcer campaigns create \
  --project <projectId> \
  --name "Outbound v1" \
  --from-file campaign-flow.json \
  --json
```

Update campaign metadata:

```bash
talentsourcer campaigns update <campaignId> \
  --description "Updated campaign notes" \
  --json
```

Replace the campaign flow:

```bash
talentsourcer campaigns update-flow <campaignId> \
  --from-file campaign-flow.json \
  --json
```

List filtered shortlist candidates before enrollment:

```bash
talentsourcer shortlists candidates <shortlistId> \
  --email-availability has_email \
  --contact-data-availability has_contact_data \
  --campaign-presence not_in_campaign \
  --limit 50 \
  --json
```

Useful shortlist candidate filters:

- `--email-availability all|has_email|missing_email`: email campaign readiness.
- `--contact-data-availability all|has_contact_data|no_contact_data`: enrichment/contact result. Use `no_contact_data` to find candidates where enrichment completed but no usable email or phone data was found.
- `--campaign-presence all|in_campaign|not_in_campaign`: current campaign membership.
- `--campaign-definition <campaignId>`: filter membership against a specific campaign.

Enroll one filtered page from a shortlist into a draft campaign:

```bash
talentsourcer campaigns enroll-shortlist <campaignId> \
  --shortlist <shortlistId> \
  --email-availability has_email \
  --contact-data-availability has_contact_data \
  --campaign-presence not_in_campaign \
  --limit 50 \
  --json
```

Use `continueCursor` from the response to continue enrolling additional pages. Enrollment creates draft campaign enrollments; do not assume messages are sent. Launching, pausing, and resuming campaigns remain campaign-control actions outside this basic agent workflow.

Get campaign analytics:

```bash
talentsourcer campaigns analytics <campaignId> \
  --range 30d \
  --json
```

Allowed analytics ranges are `7d`, `30d`, and `90d`. Use analytics to report enrollment status counts, reply rates, interested rates, connection acceptance rates, and daily trend data.

## Outreach Task Commands

List LinkedIn outreach tasks:

```bash
talentsourcer outreach-tasks list \
  --scope my \
  --limit 50 \
  --json
```

Useful filters:

- `--project <projectId>`
- `--campaign <campaignId>`
- `--scope my|all|unassigned|completed`
- `--action-type linkedin_connection|linkedin_message|linkedin_inmail`
- `--due-before <timestamp-or-ISO-date>`
- `--cursor <continueCursor>`
- `--limit <1-100>`

Snooze a task:

```bash
talentsourcer outreach-tasks snooze <taskId> \
  --until 2026-05-05T09:00:00Z \
  --json
```

Skip a task:

```bash
talentsourcer outreach-tasks skip <taskId> \
  --reason "Not relevant" \
  --json
```

Mark a task sent only after the LinkedIn action was completed outside TalentSourcer:

```bash
talentsourcer outreach-tasks mark-sent <taskId> --json
```

Outreach task commands do not automate LinkedIn in the backend. They expose task copy, due dates, candidate/profile context, and safe state transitions. Only use `mark-sent` after the local/user-controlled LinkedIn action actually happened.

Campaign flow JSON:

```json
{
  "steps": [
    {
      "stepOrder": 1,
      "channel": "email",
      "actionType": "email",
      "fallbackSubjectTemplate": "Quick question, {{candidateFirstName|there}}",
      "fallbackBodyTemplate": "Hi {{candidateFirstName|there}}, would you be open to a short conversation?",
      "personalizationMode": "overrides_allowed",
      "delayValue": 0,
      "delayUnit": "hours",
      "delayFrom": "previous_step_sent"
    }
  ]
}
```

Campaign step rules:

- `stepOrder` must be consecutive starting at `1`.
- Every non-empty outreach step needs fallback copy.
- LinkedIn connection notes must be at most 300 characters.
- LinkedIn InMail subjects must be at most 200 characters.
- A LinkedIn message step requires an earlier LinkedIn connection step.

Supported template variables:

- `{{candidateFirstName|there}}`
- `{{candidateLastName|your team}}`
- `{{candidateFullName|there}}`
- `{{candidateCurrentEmployer|your company}}`
- `{{candidateLocation|your area}}`
- `{{roleTitle}}`
- `{{roleCompany}}`
- `{{roleLocation}}`
- `{{personaFirstName}}`
- `{{personaLastName}}`
- `{{personaFullName}}`
- `{{personaSignature}}`

Candidate variable fallbacks are required. Use the `{{variableName|fallback text}}` form rather than `{{variableName}}` for candidate variables so the message can still render when candidate data is missing. Keep fallbacks neutral and natural in context.

Role variables come from the TalentSourcer project. Outreach profile variables come from the assigned outreach persona/profile. Role and persona variables do not require inline fallbacks, but you may still provide one if the sentence would read better when data is missing.

Good fallback examples:

- `Hi {{candidateFirstName|there}},`
- `I noticed your work at {{candidateCurrentEmployer|your company}}.`
- `Given your background in {{candidateLocation|your area}}, ...`
- `I thought the {{roleTitle|role}} opportunity at {{roleCompany|our company}} could be relevant.`
- `{{personaSignature|Best,}}`

Avoid fallback placeholders like `Fallback`, `N/A`, or text that exposes missing data to the candidate. Do not invent personal details when data is missing.

## Briefing Commands

List briefings for a project:

```bash
talentsourcer briefings list --project <projectId> --json
```

Get one briefing:

```bash
talentsourcer briefings get <briefingId> --json
```

Create a briefing:

```bash
talentsourcer briefings create \
  --project <projectId> \
  --name "Version 1" \
  --description "Initial briefing created by agent." \
  --json
```

Update briefing metadata:

```bash
talentsourcer briefings update <briefingId> \
  --name "Version 2" \
  --description "Updated briefing notes." \
  --json
```

Replace structured requirements from JSON:

```bash
talentsourcer briefings update-requirements <briefingId> \
  --from-file requirements.json \
  --json
```

Generate talent pool insights for a briefing after candidate checks exist:

```bash
talentsourcer briefings insights generate <briefingId> --json
```

Fetch latest talent pool insights:

```bash
talentsourcer briefings insights get <briefingId> --json
```

## Requirement Design Principles

Briefing requirements are used to evaluate and rank candidates. Write them so another agent or evaluator can apply them consistently from a LinkedIn profile, CV, ATS record, or other candidate data.

### Must-Haves

Must-haves are deal-breakers.

A requirement belongs in `mustHaves` if the candidate should likely be rejected or heavily deprioritized when evidence is missing or clearly false.

Good must-haves are:

- Specific enough to evaluate.
- Based on observable evidence.
- Important to success in the role.
- Not merely a preference.

Examples:

- `Has 5+ years of backend engineering experience`
- `Has managed B2B SaaS sales cycles from prospecting to close`
- `Is based in Germany or has clear right to work in Germany`

Avoid vague must-haves like:

- `Great communicator`
- `Strong cultural fit`
- `Smart and motivated`

If a subjective trait is important, convert it into observable evidence in the description.

### Nice-To-Haves

Nice-to-haves are ranking signals, not rejection criteria.

A requirement belongs in `niceToHaves` if it makes a candidate stronger but should not disqualify them when absent.

Examples:

- `Has worked in fintech`
- `Has experience with RevOps tooling`
- `Has contributed to open-source TypeScript projects`

### Requirement Descriptions

The `description` is the judging rubric. It should explain how to decide whether the requirement is satisfied.

A strong description should answer:

- What evidence should the evaluator look for?
- Which profile sections or data sources are relevant?
- What counts as strong evidence?
- What counts as weak or insufficient evidence?
- How should ambiguity be handled?

Example:

```json
{
  "requirement": "Has 5+ years of backend engineering experience",
  "description": "Check job titles, role descriptions, and tenure across backend-focused engineering roles. Count roles involving APIs, databases, infrastructure, distributed systems, or server-side product development. Do not count purely frontend, QA, support, or IT administration roles unless backend ownership is explicit.",
  "category": "workExperience",
  "weight": 1
}
```

### Subjective Requirements

Subjective requirements are allowed only when the description turns them into observable signals.

Weak:

```json
{
  "requirement": "Is a job-hopper",
  "description": "Check if they change jobs too often.",
  "category": "workExperience"
}
```

Better:

```json
{
  "requirement": "Does not show a repeated short-tenure pattern",
  "description": "Review the last 5-7 years of experience. Treat multiple full-time roles under 12 months as a risk signal unless there is a clear explanation such as internships, contract roles, layoffs, company closure, fixed-term consulting, or career progression. Do not penalize one short tenure if the broader pattern shows stability.",
  "category": "workExperience",
  "weight": 1
}
```

Use subjective requirements carefully. Prefer neutral phrasing and avoid protected-class, discriminatory, or personal-life inferences.

## Requirement JSON Contract

The JSON file must contain `mustHaves` and `niceToHaves` arrays.

```json
{
  "mustHaves": [
    {
      "requirement": "Has 5+ years of backend engineering experience",
      "description": "Check job titles, role descriptions, and tenure across backend-focused engineering roles.",
      "category": "workExperience",
      "weight": 1
    }
  ],
  "niceToHaves": [
    {
      "requirement": "Has worked on AI-related products",
      "description": "Look for AI, ML, LLM, MLOps, or model integration experience in product or platform teams.",
      "category": "industryExperience",
      "weight": 1
    }
  ]
}
```

Supported categories:

- `workExperience`
- `technical`
- `soft`
- `location`
- `seniority`
- `industryExperience`
- `education`
- `certifications`
- `languages`
- `currentWorkStatus`
- `other`

Optional fields:

- `weight`: numeric importance, default is `1`.
- `fulfilled`: usually omit; backend defaults to `false`.
- `reasoning`: usually omit; backend defaults to empty string.
- `confidence`: usually omit; backend defaults to `0`.

## Recommended Agent Workflow

When a user asks you to create or update a TalentSourcer project and briefing:

1. Verify the CLI is installed with `talentsourcer --help`.
2. Verify auth with `talentsourcer auth status --json`.
3. Create or identify the project.
4. Create or identify the briefing.
5. Draft a requirements JSON file locally.
6. Use `talentsourcer briefings update-requirements <briefingId> --from-file <file> --json`.
7. Return the updated briefing URL to the user.

When drafting requirements, ask a concise clarifying question if role-critical details are missing, such as location, seniority, industry, compensation, or must-have technology stack.

Do not invent deal-breakers that the user did not state or strongly imply.

## Talent Pool Insights

Talent pool insights help the user understand how briefing requirements affect candidate pool size.

Use insights after candidate checks have been run for a briefing. If no candidate checks exist, explain that TalentSourcer needs completed candidate checks before it can calculate sensitivity analysis.

Insights compare two kinds of changes:

- Expanding the pool by moving a must-have to a nice-to-have.
- Narrowing the pool by moving a nice-to-have to a must-have.

When interpreting insights:

- Do not automatically recommend loosening requirements only because the pool gets larger.
- First preserve true deal-breakers that are role-critical.
- Consider loosening must-haves with high pool impact if they are preferences disguised as requirements.
- Consider tightening nice-to-haves only if they strongly predict success and the remaining pool is still large enough.
- Use the current pool counts and deltas to explain tradeoffs in plain language.

Good agent response pattern:

1. Summarize the current pool size.
2. Identify requirements that most expand or narrow the pool.
3. Explain whether each change is strategically sensible.
4. Ask the user before changing requirements.

Do not update requirements based on insights without user confirmation unless the user explicitly asked you to optimize or revise the briefing autonomously.
