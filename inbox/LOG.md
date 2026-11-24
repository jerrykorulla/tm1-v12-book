# Inbox log

Each processed inbox file gets an entry here, newest first, in this format:

```text
## YYYY-MM-DD: <file name>

- **Pages changed:** docs/..., docs/...
- **Pages created:** docs/...
- **Left out:** what was skipped and why
- **Open questions:** anything that needs checking
```

## 2026-11-24: sandbox_writeup.md

- **Pages created:** docs/rest-api/sandboxes.md
- **Pages changed:** docs/rest-api/index.md (replaced "Work in progress" with an overview), docs/concepts/database-objects.md, docs/concepts/glossary.md (added Base, Cellset, Change set, Publish, Sandbox dimension; updated Sandbox), mkdocs.yml
- **Left out:** server implementation details and internal values; only behaviour visible to users is included.
- **Open questions:** Check the `ServerSandboxesDelete` filter syntax (spaces after commas). Where is `EnableSandboxDimension` set in v12? The book says "commit" and the REST API says "Publish"; both terms are now used.

