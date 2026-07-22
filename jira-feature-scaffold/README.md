# jira-feature-scaffold

Turns a raw Jira ticket into two markdown files inside the project's `feature/` folder, designed to persist context and track progress across Claude Code sessions and developers.

---

## What it generates

```
feature/
  TF-1036.md        ← full ticket context
  TF-1036_TODO.md   ← step-by-step execution plan
```

### `TF-1036.md`

Ticket description, acceptance criteria, and technical notes. Every Claude Code session reads this first to understand the feature.

### `TF-1036_TODO.md`

Checklist where each item has a commit-message-style title and a brief parenthetical description for quick context. Items are checked off as work progresses.

```
- [x] feat(api): add POST /upload endpoint (Creates route handler and validates multipart form; connects to S3 service in storage.service.ts)
- [ ] feat(ui): implement FileUploadButton component (Drag-and-drop zone using react-dropzone; emits onFileSelect consumed by DocumentForm)
- [ ] test(api): add integration tests for upload endpoint (Covers happy path and invalid MIME type rejection; uses fixtures in /tests/fixtures/)
```

> Each item title is ready to use as a git commit message — no reformatting needed.

---

## Usage

### Start the scaffold

Paste the Jira ticket text into Claude Code:

```
Here is my Jira ticket:

TF-1036: [title]
[ticket text]
```

### Start each session

```
Read feature/TF-1036.md and feature/TF-1036_TODO.md before continuing
```

### Check off a completed step

```
Mark the upload endpoint step as complete
```

### Add a session note

After finishing work, ask Claude Code to add a brief note under the `## Notes from sessions` section in the TODO file. This is the handoff note for the next developer or session.
