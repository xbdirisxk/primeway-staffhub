# Architecture

Produced in the architecture and data-design sessions (with the `architect` and `db-rls-reviewer` agents), then kept up to date.

| Doc | Contents | Status |
| --- | --- | --- |
| `overview.md` | System context, the web app, worker, Supabase and DigitalOcean pieces, request lifecycle, folder structure | to write |
| `data-model.md` | Entity diagram (Mermaid), every table with columns, relationships, enums, soft delete | to write |
| `rls-matrix.md` | Table × role × operation (select / insert / update), helper functions, sensitive-table rules | to write |
| `flows/` | One file per key flow: sign-in and 2FA, branch switching, applicant link, upload → scan → review, agreement, gate approval, transfer | to write |
| `environments.md` | Local, staging, production: what runs where, which keys each part holds | to write |
