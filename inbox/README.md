# Inbox

Drop source material for the book here: notes, PDFs, Word documents, slides, screenshots, code, exported rules or TurboIntegrator processes.

Then ask an AI agent to process it:

- **Claude Code:** run `/ingest`. To process only some files, name them: `/ingest feeders-notes.md`.
- **Any other agent:** ask it to "process the inbox following AGENTS.md".

The agent reads each file, works its content into the right pages under `docs/`, checks the site still builds, moves the file to `inbox/processed/`, and records what it did in [LOG.md](LOG.md). Review the changes before you commit them.

## Files here are not committed

Everything in this folder except this README and `LOG.md` is ignored by git, so your source files are never pushed to the public repo. Keep your own copies if you need them.

Only the rewritten content in `docs/` is published. Agents are told to rewrite in their own words and to leave out client names, real data and credentials, but check before you commit.
