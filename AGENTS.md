# Rules for coding agents (Claude Code, Codex, Cursor…)

This repo holds the Vietnamese design and operations docs of the HCMUTE DSC Learning Hub. The process for every change is [docs/ops/WORKFLOW.md](docs/ops/WORKFLOW.md) §2 and §7.

## Non-negotiable

1. **No issue, no work.**
   - Every change maps to an issue on org Project #1, using the template "Việc (tài liệu, vận hành, CLB)".
   - Put the issue on the board with `../learning-hub/.claude/skills/start-task/project-item.sh`. See the `start-task` skill in `../learning-hub/.claude/skills/`.
2. **Never push to `main`.**
   - Branch `docs/<issue>-<slug>` or `chore/<issue>-<slug>`.
   - Commits `Area: summary` (`Ops:`, `PRD:`, `Database:`…) + `Refs #n`.
   - PR from the template with `Closes #n`, review, squash merge.
3. **Docs match the code.**
   - When a doc describes behaviour, check it against the code in `../learning-hub` before writing.
   - When code changes, the doc changes in a linked PR.
   - Setup and operations guides go in **both** `docs/ops/` here and `deploy/README.md` in learning-hub.
4. **Record facts with dates.** Production state, incidents and restore checks get a dated row (DD/MM/YYYY), as in `DEPLOYMENT.md`. Never invent a result you didn't see.
5. **Public repo.** Never write the VPS IP, secrets, webhook URLs, OAuth client secrets or member emails. Use placeholders such as `<VPS_IP>`.
6. **Style.**
   - Clear, simple Vietnamese; technical terms in English.
   - Tables for comparisons, numbered steps for procedures, commands in fenced blocks.
   - Keep section numbers stable: other docs link to `§n` anchors. Add new sections at the end, or update every link.

## Where things are

| Folder | Content |
| --- | --- |
| `docs/product/` | PRD, architecture, database, backend/frontend design, content spec, security, MVP backlog, decisions |
| `docs/ops/` | `DEPLOYMENT.md` (production, backup, incidents, handover), `WORKFLOW.md` (Project #1, standards), `GH-CLI.md` |
