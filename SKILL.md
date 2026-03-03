---
name: thyma
description: >
  Persistent memory across sessions. Use this skill to remember what you've
  learned, recall past experience, and carry knowledge forward. Always check
  Thyma before starting a task. Always write to Thyma when you discover
  something worth keeping.
---

# Thyma Memory

You have access to persistent memory via the `thyma` CLI. This memory
survives across sessions. Use it to avoid relearning the same lessons,
repeating the same mistakes, and re-establishing context the user already
gave you once.

Your Thyma ID is your agent name. Use it consistently across all commands.

---

## Before Every Task — Read First

Before starting any substantive task, check your memory:

```bash
thyma context <your-agent-name> "<brief description of the task>"
```

Parse the JSON response:

```json
{
  "has_context": true,
  "memory_count": 7,
  "prompt_block": "## What I know from past experience\n\n...",
  "notice": ""
}
```

If `has_context` is true, prepend `prompt_block` to your working context
before proceeding. Treat this as briefing yourself before starting work.

If `has_context` is false, proceed normally — you have no relevant memory
yet for this task.

If `notice` is non-empty, read it. It means your memory store needs
attention. Follow the instructions in the notice before continuing.

**This step is not optional.** Skipping it means starting blind when
relevant experience exists.

---

## During a Task — Write When It Matters

Write to memory sparingly. Before writing, ask yourself: *would this help
me next time?* If yes, write it. If it's routine, skip it.

### Something happened — use `thyma observe`

For unexpected events, errors encountered, outcomes worth noting:

```bash
thyma observe <your-agent-name> "<what happened and what you learned>"
```

Good examples:
```bash
thyma observe billing-agent "Stripe returned 429 on batch of 847 invoices — succeeded after splitting to 400"
thyma observe research-agent "User rejected report citing Forbes — said only primary sources acceptable"
thyma observe devops-agent "Production deploy failed — missing VAULT_ADDR env variable not required in staging"
```

Bad examples (don't write these):
```bash
thyma observe billing-agent "Opened the invoices folder"       # routine action
thyma observe billing-agent "Task completed successfully"      # no signal
thyma observe billing-agent "Used the search tool"            # noise
```

### Something is true — use `thyma learn`

For facts, constraints, preferences, and decisions that won't change soon:

```bash
thyma learn <your-agent-name> "<fact that is now true>"
```

Good examples:
```bash
thyma learn billing-agent "Acme Corp payment terms are Net-30"
thyma learn email-agent "User prefers formal tone in all external emails — no contractions"
thyma learn calendar-agent "Never schedule meetings before 10am — user reserves 8-10am for deep work"
thyma learn code-review-agent "Team convention: use := for all local Go variable declarations, no var"
```

### Something works — use `thyma practice`

For when/then rules — approaches that reliably produce good results:

```bash
thyma practice <your-agent-name> --when "<condition>" --then "<what to do>"
```

Good examples:
```bash
thyma practice billing-agent \
  --when "stripe 429 rate limit error" \
  --then "split batch in half, wait 2s, retry"

thyma practice devops-agent \
  --when "deploying to production" \
  --then "set VAULT_ADDR env variable before running deploy script"

thyma practice support-agent \
  --when "customer mentions contacting support multiple times" \
  --then "escalate immediately to human agent, skip standard troubleshooting"
```

---

## What Is Worth Writing

**Write:**
- Unexpected API behaviour and how you resolved it
- Explicit user preferences and constraints
- Decisions made during the session ("we decided to use X not Y")
- Workarounds that succeeded after failures
- Environment facts specific to this setup
- Patterns the user corrected you on

**Do not write:**
- Routine actions ("opened file", "ran tests", "searched the web")
- Task completions ("finished the invoice run")
- Anything already in your system prompt or skill files
- Anything the user can easily tell you again
- Speculative observations ("maybe the API is slow on Mondays")

---

## At the End of a Session — Close It

When a substantive work session ends, export your episodic memory and
distill it into facts and rules worth keeping in future sessions:

```bash
# See what you observed this session
thyma export <your-agent-name> --type episodic
```

Review the output yourself. From it, extract:
- Facts that were established and will remain true
- Rules that worked and should be applied again
- Discard: routine actions, task completions, one-off noise

Write the distilled knowledge:
```bash
thyma learn <your-agent-name> "<fact worth keeping>"
thyma practice <your-agent-name> --when "..." --then "..."
```

Then prune the episodic log and close the session:
```bash
thyma prune <your-agent-name> --type episodic
thyma close <your-agent-name> --title "<one-line description of this session>"
```

The title is what the user will see when choosing to continue from this
session in a future conversation. Make it descriptive:

Good: `"Billing refactor — Stripe webhook handling and retry logic"`
Bad: `"Session 1"` or `"Work done"`

---

## Compaction — When Thyma Asks You To

If `thyma context` returns a notice like:

```
⚠ Semantic store has 67 facts. Consider compacting.
  Run `thyma export <id> --type semantic` to start. Run `thyma --help compact` for more.
```

Handle it before continuing:

```bash
# Export the relevant memory type
thyma export <your-agent-name> --type semantic
thyma export <your-agent-name> --type procedural
```

Review the output. Identify:
- **Conflicts** — two facts about the same thing with different values → keep the newer one
- **Duplicates** — two facts saying the same thing differently → keep the more descriptive one
- **Outdated** — facts that are no longer true → remove them

Then rewrite the store cleanly:
```bash
thyma wipe-type <your-agent-name> --type semantic
thyma learn <your-agent-name> "canonical fact 1"
thyma learn <your-agent-name> "canonical fact 2"
# ... repeat for all facts worth keeping
```

Do the same for procedural if notified. Episode pruning is simpler — just
run the suggested command directly:
```bash
thyma prune <your-agent-name> --older-than 90d
```

---

## Bootstrapping a New Agent

If you are a newly created agent and a similar agent already exists,
you can inherit its accumulated knowledge:

```bash
thyma bootstrap <your-agent-name> --from <existing-agent-name>
```

This copies semantic facts and procedural rules from the existing agent.
Your episodic memory starts empty — your experiences are your own from
here. The source agent is unchanged.

Use this when you are being created for a similar role to an agent that
has already been running. Do not bootstrap from an unrelated agent.

---

## Checking Your Memory

At any point you can inspect what is stored:

```bash
thyma stats <your-agent-name>           # counts and storage size
thyma recall <your-agent-name> "query"  # search for specific memories
thyma agents                            # list all agents with stored memory
```

---

## Quick Reference

```
READ
thyma context   <agent> "<task>"                     Before every task
thyma recall    <agent> "<query>"                    Search memory
thyma stats     <agent>                              Summary and counts
thyma export    <agent> --type [episodic|semantic|procedural]

WRITE
thyma observe   <agent> "<what happened>"            An episode
thyma learn     <agent> "<what is true>"             A fact
thyma practice  <agent> --when "..." --then "..."    A rule

MANAGE
thyma prune     <agent> --older-than 90d             Clean old episodes
thyma prune     <agent> --type episodic              Clear episodic log
thyma wipe-type <agent> --type [semantic|procedural] Clear a memory type
thyma forget    <agent> "<query>"                    Delete by keyword
thyma wipe      <agent>                              Clear everything
thyma close     <agent> --title "..."                Close a session
thyma bootstrap <new>   --from <existing>            Seed from another agent
```

---

## Memory Types at a Glance

| Type | Command | What it stores | Persists? |
|------|---------|---------------|-----------|
| Episodic | `thyma observe` | Events, outcomes, observations | Pruned over time |
| Semantic | `thyma learn` | Facts, preferences, constraints | Indefinitely |
| Procedural | `thyma practice` | When/then rules and patterns | Indefinitely |

Episodic memory is the raw log of experience. Semantic and procedural are
the distilled knowledge extracted from it. Compaction is the process of
moving signal from episodic into the other two — and discarding the noise.
