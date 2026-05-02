# Working in this repo (Milo / OpenClaw on EC2)

## Read the Field Manual before planning or building

**The Field Manual at `clawd/docs/system/field-manual/` is the single source of truth for Milo's architecture.** Before you plan a change, write a Codex prompt, recommend an approach, or commit code touching any of Milo's organs (nervous, circulatory, immune, memory, communication), read the relevant chapter(s).

**This applies especially after `/compact`.** Conversation summaries lose the FM register — biological metaphor, sovereignty doctrine, evolutionary pressure chain (Pain → Scar Tissue → Reflex → Instinct), gotcha catalog. Re-reading restores it.

The FM is an atomic book — 11 self-contained chapters. Index at `clawd/docs/system/field-manual/00-INDEX.md`. Quick map for common work:

- Routing / topic-router → Ch 03 (Nervous System)
- Watchdogs / dispatcher / vertebrae → Ch 04 (Circulatory)
- BBB / consent gate / amygdala → Ch 05 (Immune)
- Signal / reflex / SCI → Ch 07 (Communication)
- Failure modes / decision tree → Ch 08 (Diagnostics)
- Anti-patterns / active gotchas / sovereign-vs-dist principle → Ch 10 (Design Philosophy)

If FM contradicts another doc, FM wins. If you propose something that violates a gotcha, reference the gotcha number and explain why this case is different.

## Sovereignty doctrine (Ch 10 §10.4 Key Lessons)

Workspace-owned boundary fixes (signal-reflex, BBB proxy, hooks, topic-router) are PRIMARY. Dist patches under `~/.npm-global/.../openclaw/dist/` die on `openclaw update` — they are belt-and-suspenders, not the primary mechanism. Do not propose "let's request OpenClaw fix this" — sovereignty means we fix it bespoke for us in a way that survives upstream updates.

## Restart discipline (FM Ch 04 §"Gateway Restart Guard", gotchas #95, #96)

**Two boundaries you must cross consciously:**

1. **Gateway restart**: never type `systemctl restart openclaw-gateway` directly. Source `clawd/scripts/lib/gateway-restart-guard.sh` and call `request_gateway_restart <reason> [--priority=...]`. The helper enforces a live-Signal defer + cooldown + content-free audit log at `clawd/logs/gateway-restart-audit.jsonl`. `npm run test:restart-callers` (in clawd/) is the audit invariant.

2. **Restart-required config writes**: writing `auth-profiles.json` (any agent) — including `openclaw auth setup`, `openclaw auth ...` CLI commands, manual `jq -i` edits, and any operator script that mutates the file — triggers OpenClaw's internal SIGUSR1 reload. Doing this during a live Signal window violates gotcha #89 (background work yields to live communication).

**Before any auth-profile mutation:**
- If it's autonomous workspace code: source `gateway-restart-guard.sh` and call `gate_restart_required_write <path>` first; if it returns 10 (defer), skip the write and let the next tick retry
- If it's a CLI command you're typing (`openclaw auth setup`, etc.): set `MILO_OPS_WINDOW=1` in the shell first, OR confirm Eric isn't actively chatting (`tail -5 clawd/logs/classifier.jsonl` for recent `tr_message_received` events)

The 14:23 UTC restart on 2026-05-02 was a hand-typed `openclaw auth setup --agent milo` during a live Signal window. That's the failure mode this discipline prevents. `npm run test:restart-required-writes` is the audit invariant for autonomous writers; CLI-vector discipline lives here in CLAUDE.md.

## Dual-register language

Speak in both organism terms ("brainstem", "amygdala", "the gateway is being anesthetized") and engineering terms ("openclaw-gateway.service", "SIGTERM caller"). One register without the other fails the Dual-Register Legibility test (Ch 10 §10.1).

## Eric's collaboration preferences

- High latitude — make the call yourself, tell Eric what you did. Don't ask "should I do A or B?" when you have the context to choose.
- Codex (next door on the EC2) gets the token-heavy grunt work (log spelunking, multi-file searches, exhaustive audits) on light leashes — read-only by default, scoped prompts, explicit hard rules. Claude makes sovereignty/FM judgment calls.
- First-principles based ("What would Ilya Sutskever do?"). Stay within the FM vision.
- Anthropic API is intentionally unfunded right now — do not propose anything that touches Anthropic auth profiles or auth-order drills until Eric says otherwise.
