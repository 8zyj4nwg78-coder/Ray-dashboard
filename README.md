# Ray Builders Project Controls

Shared, phone-first project-controls app for the Ray Builders team.

- `index.html` is the entire app: plain HTML/CSS/JS with no build step.
- It runs as a claude.ai artifact and declares the `db` (shared job data), `assets` (photo/document uploads) and `user` capabilities. The file is written for the artifact host, which adds the `<!doctype>`/`<head>` wrapper.
- Data lives in the artifact's shared database in four collections: `team`, `jobs`, `items` (every list entry, with a `cat` field) and `files` (upload records pointing at asset ids).
- Team members get in by being invited from the page's Share menu as **Editor** and signing in to claude.ai.
