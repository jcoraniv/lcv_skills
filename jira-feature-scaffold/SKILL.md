---
name: jira-feature-scaffold
description: >
  Scaffold a feature from a Jira ticket. Use this skill whenever the user pastes or describes a Jira ticket (e.g. "TF-1036", "aquí está mi ticket de Jira", "genera el feature de este caso", "crea el md para este ticket", "scaffold this ticket"). 
  The skill generates two markdown files inside a `feature/` folder at the project root:
    - `{TICKET_ID}.md` — context file with the full ticket information
    - `{TICKET_ID}_TODO.md` — step-by-step plan where each sub-task has a commit-message-style title and a brief description in parentheses, formatted as checkboxes
  Both files are designed to be read at the start of every Claude Code session to provide context and track progress, enabling seamless handoffs between developers or sessions.
---

# Jira Feature Scaffold

This skill turns a raw Jira ticket into two structured markdown files that serve as the **persistent context and progress tracker** for a feature across all Claude Code sessions.

## Why this matters

Claude Code has no memory between sessions. These two files solve that:
- `{TICKET_ID}.md` — tells any Claude Code instance (or new developer) *what* the feature is and *why* it exists
- `{TICKET_ID}_TODO.md` — tells them *where* things stand and *what's next*

Each sub-task in the TODO is written so its title doubles as a git commit message. The parenthetical description gives the next session enough context to pick up without needing to re-read everything.

---

## Step 1 — Parse the Jira ticket

Extract from the user-provided ticket text:

| Field | How to find it |
|-------|---------------|
| `TICKET_ID` | The ticket key, e.g. `TF-1036` |
| `TICKET_TITLE` | Short summary / title of the ticket |
| `DESCRIPTION` | Full description body |
| `ACCEPTANCE_CRITERIA` | Any explicit AC, DoD, or "definition of done" section |
| `PRIORITY` | If stated (High / Medium / Low) |
| `LABELS` / `COMPONENTS` | If stated |
| `REPORTER` / `ASSIGNEE` | If stated |

If the user provides only a partial ticket (e.g. just the title and description), infer what you can and leave fields as `_Not specified_`.

---

## Step 2 — Plan the sub-tasks

Before writing any file, **think through the implementation plan** for this feature. Consider:

- What needs to be built end-to-end (backend, frontend, integration, tests)?
- What are the natural commit-sized units of work?
- What order minimizes blockers and makes the most sense for a developer reading top-to-bottom?

Each sub-task must follow this format:

```
- [ ] <commit-message-title> (<brief description of what this involves and why>)
```

**Commit message title rules:**
- Use Conventional Commits format: `type(scope): short description`
- Types: `feat`, `fix`, `refactor`, `test`, `chore`, `docs`, `style`, `build`
- Keep it under 72 characters
- Use imperative mood: "add", "implement", "create" — not "added" or "adding"

**Parenthetical description rules:**
- 1–2 sentences maximum
- Explain *what* the sub-task does and *why* it matters or any non-obvious detail
- Written for a developer (or Claude instance) who hasn't seen the conversation — they should be able to pick this up cold and know exactly what to do
- Include the file, module, or layer being touched if relevant

**Example sub-task:**
```
- [ ] feat(api): add POST /documents/upload endpoint (Creates the route handler and validates the multipart form; connects to S3 upload service defined in storage.service.ts)
- [ ] feat(ui): implement FileUploadButton component (Drag-and-drop zone using react-dropzone; emits onFileSelect event consumed by DocumentForm)
- [ ] test(api): add integration tests for upload endpoint (Covers happy path, max-size rejection, and invalid MIME type; uses existing test fixtures in /tests/fixtures/)
```

Aim for **6–15 sub-tasks** depending on feature complexity. Too few is vague; too many is noise.

---

## Step 3 — Create the files

### Folder

Create the `feature/` folder at the project root if it doesn't exist.

```bash
mkdir -p feature
```

---

### File 1: `feature/{TICKET_ID}.md`

This is the **context file**. Every Claude Code session should read it first to understand the feature.

```markdown
# {TICKET_ID}: {TICKET_TITLE}

> **Status:** In Progress  
> **Priority:** {PRIORITY}  
> **Labels:** {LABELS}

## Description

{DESCRIPTION}

## Acceptance Criteria

{ACCEPTANCE_CRITERIA}

## Technical Notes

{Any technical constraints, architecture decisions, or relevant context extracted from the ticket or implied by the domain. If none, write: "_No additional notes._"}

## Links & References

- Jira: `{TICKET_ID}`
- {Any URLs, related tickets, or design docs mentioned in the ticket}
```

---

### File 2: `feature/{TICKET_ID}_TODO.md`

This is the **progress tracker**. Each session checks items off as they are completed. The file evolves as the feature is built.

```markdown
# {TICKET_ID}: {TICKET_TITLE} — TODO

> Last updated: {current date YYYY-MM-DD}  
> Branch: `feature/{TICKET_ID}-{kebab-case-title}`

## Progress

- [ ] {sub-task 1 commit title} ({sub-task 1 description})
- [ ] {sub-task 2 commit title} ({sub-task 2 description})
- [ ] {sub-task 3 commit title} ({sub-task 3 description})
...

## Notes from sessions

<!-- Add session notes here as the feature progresses. Format:
### {date} — {brief what was done}
{Any blockers, decisions made, or context the next session needs to know}
-->
```

---

## Step 4 — Tell the user how to use it

After creating the files, explain briefly:

1. **At the start of every session**: Ask Claude Code to read both files before starting work. Suggested prompt:
   ```
   Lee feature/{TICKET_ID}.md y feature/{TICKET_ID}_TODO.md para entender el contexto y el estado del feature antes de continuar.
   ```

2. **Checking off tasks**: As each sub-task is completed, update the checkbox:
   ```
   - [x] feat(api): add POST /documents/upload endpoint (...)
   ```
   The user can say: *"marca como completo el paso de upload endpoint"* and Claude Code will update the file.

3. **Adding session notes**: After each session, add a brief note under `## Notes from sessions` with the date and what was done or decided. This is the handoff note for the next developer.

4. **Commit messages**: The title of each checked-off sub-task is ready to use as the git commit message — no reformatting needed.

---

## Edge cases

- **No ticket ID found**: Ask the user to confirm the ticket ID before creating files.
- **Files already exist**: Read the existing files first, then offer to update (append new sub-tasks or refresh the description) rather than overwrite.
- **Very large ticket**: If the description is very long, summarize into the key points in the context file and note that the full text is in Jira.
- **Sub-tasks already partially described in ticket**: Use the ticket's own breakdown as the starting point, then refine to match the commit-message format.
