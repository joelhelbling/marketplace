---
name: designing-agent-native-clis
description: Use when designing, building, extending, or reviewing a command-line tool that AI agents will invoke — new CLI design docs, adding commands or flags to an existing CLI, API wrappers, or auditing a CLI for agent-friendliness (hangs on prompts, unparseable output, duplicate creates, token-heavy responses).
---

# Designing Agent-Native CLIs

Ten principles for CLIs whose users include AI agents, from Trevin Chow's
["10 Principles for Agent-Native CLIs"](https://trevinsays.com/p/10-principles-for-agent-native-clis).
Agents differ from humans: they hang on prompts, retry without noticing
duplicates, pay per token of output, and generalize vocabulary across every
CLI they have ever seen. Design for that.

Apply ALL ten. Experience shows designers reliably nail 1–2 and 5 but skip
`--dry-run`, drift on vocabulary, forget enumerating errors, and omit
introspection, profiles, and feedback entirely. Walk the checklist.

## Tier 1: Table stakes

**1. Non-interactive by default.** Never prompt — a prompt hangs an agent.
Detect TTY; treat non-TTY as headless. Take all input via flags, files, or
stdin. Destructive operations require an explicit `--force` flag (invocation
alone is NOT consent — a retried or mis-generated command must not delete
things silently). Standardize on `--force`; ban ad-hoc variants like
`--skip-confirmations` or "no flag needed."

**2. Structured, parseable output.** `--json` (that exact flag — not
`--format=json`, not `--output json`) on every data-returning command.
Results → stdout; diagnostics, progress, truncation hints → stderr, so JSON
stdout stays a clean document. Suppress ANSI color when stdout isn't a TTY.
Stable exit-code taxonomy: 0 success, distinct non-zero codes per failure
class.

**3. Errors that teach, and enumerate.** Every rejection names the problem
AND the valid options, so the agent self-corrects in one retry:
`invalid status: "done" — must be one of: considering, todo, in_flight, completed`.
Validate early; give a corrected example, never a stack trace.

**4. Safe retries and explicit mutation boundaries.** Agents retry.
Idempotent creates return the existing resource instead of duplicating.
Every `create`/`update` accepts `--dry-run` (validate + show what would be
sent, send nothing). Every mutation response returns the resource id so the
agent can chain without re-searching. Async operations keep durable job
state across retries.

**5. Bounded responses, at every layer.** Default limits on all list
output (`--limit`, sane default like 20–50); cursor pagination; filter
flags. When truncating, the stderr hint should teach a narrower query
("showing 50 of 1,204 — filter with --tag or raise --limit"). Keep help
text and tool descriptions terse; budget them deliberately.

## Tier 2: Compounding

**6. Cross-CLI vocabulary consistency.** Agents transfer patterns from
every CLI they know — match the ecosystem. Use `create`/`get`/`list`/
`update`/`delete` — never `add`, `info`, `ls`, `edit`, `rm`, `show`.
Use `--force`, `--json`, `--limit`. Enforce mechanically (schema/codegen
or a lint on the command table), not by code review — "manually enforcing
consistency through reviews is Swiss cheese."

**7. Three-layer introspection.** (1) `--help` for humans; (2) an
`agent-context` command emitting versioned JSON describing the FULL command
surface — commands, flags, enum values, env vars, exit codes; (3) a skill
manifest / doc showing how to compose commands into workflows. Keep all
three synchronized with the implementation via automated validation, not
memory.

**8. Async-aware execution.** For commands wrapping async work: `--wait`
blocks until done (poll with exponential backoff + jitter), collapsing many
agent turns into one. Persist a job ledger (e.g. `~/.<cli>/jobs.jsonl`)
that survives disconnects; expose `jobs list`, `jobs get <id>`,
`jobs prune`.

**9. Persistent identity through profiles.** Named config bundles:
`profile save/use/list/show/delete`, plus `--profile <name>` as a root
flag. Precedence: explicit flag > env var > profile > default. List
available profiles in `agent-context` output so agents discover identities
instead of guessing.

**10. Two-way I/O.** `--deliver <dest>` routes artifacts atomically to
stdout, a file path, or a webhook URL; unknown schemes get a structured
refusal enumerating supported ones. A `feedback` command records agent
friction locally (JSONL), optionally POSTing upstream; advertise the
channel in `agent-context` so agents know whether feedback reaches
maintainers.

## Review checklist

- [ ] Zero prompts in any code path; `--force` gates destruction
- [ ] `--json` everywhere; stdout=data, stderr=diagnostics; exit codes stable
- [ ] Every error enumerates valid values / shows corrected usage
- [ ] `--dry-run` on every mutation; creates idempotent; ids in responses
- [ ] `--limit` + pagination; truncation hints teach better queries
- [ ] Verbs are `create/get/list/update/delete`; flags match ecosystem norms
- [ ] `--help` + `agent-context` JSON + workflow doc, all auto-validated
- [ ] `--wait` + durable jobs ledger for anything async
- [ ] Profile system with flag > env > profile > default precedence
- [ ] `--deliver` for artifacts; `feedback` command for friction

## Common mistakes

| Mistake | Fix |
|---|---|
| "Invocation is consent" for `rm`/`delete` | Retries make that dangerous — require `--force` |
| `--format=json` or JSON-only-when-piped | The literal flag `--json`, everywhere, always honored |
| Error says what's wrong but not what's valid | Enumerate the valid set in the message |
| Skipping `--dry-run` as YAGNI | It's how agents verify a mutation before committing |
| Inventing verbs (`add`, `snip`, `stash`) | Boring, ecosystem-standard verbs transfer for free |
| `--help` as the only introspection | Add machine-readable `agent-context` + workflow docs |
