# Thyma 🧠

**Total recall for your AI agents.**

> Hyperthymesia is the rare condition where nothing is ever forgotten — every experience, perfectly preserved, instantly retrievable. Thyma gives your agent the same gift.

Your OpenClaw agent is smart. But every time you start a new conversation, it forgets everything — your preferences, what worked last time, the quirks it discovered about the tools it uses. You end up repeating yourself. It ends up relearning the same lessons.

Thyma fixes that.

-----

## What Thyma Does

Thyma gives your agent a memory it can read before it starts work and write to when it discovers something worth keeping. The next time it starts up, it picks up where it left off.

Three things it remembers:

- **What happened** — “Stripe returned an error on large batches last Tuesday”
- **What’s true** — “You prefer formal tone in all external emails”
- **What works** — “When the API fails, retry with half the data and wait two seconds”

That’s it. No complex setup. No database to manage. Just a memory your agent actually uses.

-----

## Installation

You need [Node.js](https://nodejs.org) installed. If you’re running OpenClaw, you almost certainly have it already.

```bash
npm install -g thyma
```

Verify it worked:

```bash
thyma --version
```

That’s the entire installation. Thyma stores everything as plain text files on your machine — nothing in the cloud, nothing on anyone else’s servers.

-----

## Getting Started in 5 Minutes

### Step 1 — Add the Thyma skill to OpenClaw

Download the Thyma skill file and place it in your OpenClaw skills folder:

```bash
# On Mac / Linux
mkdir -p ~/.openclaw/skills/thyma
curl -o ~/.openclaw/skills/thyma/SKILL.md \
  https://raw.githubusercontent.com/chienda/thyma/main/skills/openclaw/SKILL.md
```

That’s the file that teaches your agent when and how to use its memory. Without it, the binary is installed but your agent doesn’t know it exists.

### Step 2 — Restart OpenClaw

Close and reopen your OpenClaw session, or run `/reload skills` if your version supports it. The Thyma skill will load automatically.

### Step 3 — Start a conversation

That’s it. Your agent now checks its memory before starting tasks and writes to it when it learns something useful. You don’t need to manage any of this — the agent handles it on its own.

-----

## What This Looks Like in Practice

**First conversation — agent starts fresh:**

> You: Process the invoices in my downloads folder.
> 
> Agent: *(no memory yet — works through the task, hits a rate limit error, figures out a retry approach that works)*
> 
> Agent: Done. Processed 847 invoices. Had to retry with smaller batches due to a rate limit — I’ve noted that for next time.

**Second conversation — agent remembers:**

> You: Process the new invoices.
> 
> Agent: *(checks memory, finds the retry approach from last time, uses it immediately)*
> 
> Agent: Done. Used the batch-splitting approach from last time — no errors.

No trial and error. No explaining the same thing twice. The agent got smarter between sessions.

-----

## Picking Up From a Previous Conversation

When you start a new conversation, Thyma can offer to load memory from a recent one:

```
┌─────────────────────────────────────────────────────┐
│ Continue from a recent conversation?                │
│                                                     │
│  1. Invoice processing — Stripe setup   (2hrs ago)  │
│  2. Email campaign drafts               (yesterday) │
│  3. AWS cost report                     (3 days ago)│
│  4. Start fresh                                     │
│                                                     │
│  Enter 1-4 (default: 4):                            │
└─────────────────────────────────────────────────────┘
```

Pick a number and your agent starts the new conversation already knowing what was established in the one you chose — the decisions made, the preferences noted, the approaches that worked. Choose “Start fresh” and it begins with a clean slate.

-----

## Checking What Your Agent Remembers

At any point you can look at what’s stored:

```bash
# See a summary — how many memories, how much space
thyma stats my-agent

# Search for something specific
thyma recall my-agent "stripe"

# See everything that would be injected before a task
thyma context my-agent "process invoices"
```

The output is plain text. You can read it, edit it, or delete entries you don’t want kept.

-----

## Correcting a Memory

If your agent remembered something wrong, fix it directly:

```bash
# Remove a specific memory
thyma forget my-agent "stripe rate limit is 100"

# Remove everything and start clean
thyma wipe my-agent
```

Or just tell your agent in conversation: “That’s not right — forget that and remember this instead.” It will update its memory accordingly.

-----

## Keeping Memory Clean

Over time, memory accumulates. Thyma will let you know when it’s getting large:

```
⚠ Episodic log has 523 entries. Run `thyma prune my-agent --older-than 90d`
  to clean up old entries. Run `thyma --help compact` for more options.
```

When you see this, run the suggested command or ask your agent to handle it:

```bash
# Remove episodes older than 90 days (safe, recommended regularly)
thyma prune my-agent --older-than 90d
```

For deeper cleanup — merging duplicate facts, removing outdated information — ask your agent directly:

> “Review and compact your Thyma memory. Export it, remove anything outdated or duplicated, and write back a clean version.”

Your agent knows how to do this. It reads its own memory, reasons about what to keep, and rewrites it cleanly.

-----

## Your Agent’s Memory Files

Everything is stored at:

```
~/.thyma/agents/your-agent-name/
```

Three files:

|File             |What it contains                              |
|-----------------|----------------------------------------------|
|`episodic.jsonl` |Things that happened — events and observations|
|`semantic.json`  |Things that are true — facts and preferences  |
|`procedural.json`|Things that work — when/then rules            |

You can open these in any text editor. They’re plain text. Back them up with Time Machine, copy them to a new machine, or delete them if you want to start over. No special tools required.

-----

## Multiple Agents

If you run multiple OpenClaw agents, each gets its own memory:

```bash
# See all agents with stored memory
thyma agents

# Check a specific agent
thyma stats billing-agent
thyma stats research-agent
```

Agents don’t share memory by default. Each one builds its own.

If you want a new agent to start with what an existing one knows:

```bash
thyma bootstrap new-agent --from existing-agent
```

The new agent gets a copy of the existing agent’s facts and rules. Its own experiences start fresh from there. The original agent is unchanged.

-----

## Command Reference

```
thyma context   <agent> "<task>"                     Check memory before a task
thyma recall    <agent> "<query>"                    Search for specific memories
thyma stats     <agent>                              See memory summary
thyma agents                                         List all agents

thyma observe   <agent> "<what happened>"            Write an episode
thyma learn     <agent> "<what is true>"             Write a fact
thyma practice  <agent> --when "..." --then "..."    Write a rule

thyma export    <agent> --type [episodic|semantic|procedural]
thyma forget    <agent> "<query>"                    Remove matching memories
thyma prune     <agent> --older-than 90d             Clean old episodes
thyma wipe      <agent>                              Remove all memory
thyma close     <agent> --title "..."                Close a conversation session
thyma bootstrap <new>   --from <existing>            Copy memory to new agent

thyma --help                                         Full documentation
thyma --help compact                                 How to compact memory
```

-----

## Frequently Asked Questions

**Is my data sent anywhere?**

No. Everything stays on your machine in `~/.thyma/`. Thyma has no servers, no accounts, no cloud sync. Your memory files are yours.

**Does this work with Claude Code too?**

Yes. The Thyma skill format is compatible with Claude Code’s skill system. See the [Claude Code integration guide](https://github.com/chienda/thyma/docs/claude-code.md) for setup instructions.

**What if I’m not technical — can I still use this?**

Yes. Once the skill is installed, you don’t need to run any commands yourself. Your agent manages its own memory. The commands in this guide are for when you want to inspect, correct, or clean up what’s stored — not for day-to-day use.

**Will this slow my agent down?**

No meaningfully. Memory reads and writes are file operations — they complete in milliseconds. You won’t notice it.

**What’s the difference between this and OpenClaw’s built-in memory?**

OpenClaw’s built-in memory stores conversation history. Thyma stores distilled knowledge — the things worth keeping across many conversations, extracted from the noise of individual sessions. They complement each other.

-----

## Getting Help

- **Documentation:** [github.com/chienda/thyma](https://github.com/chienda/thyma)
- **Issues:** [github.com/chienda/thyma/issues](https://github.com/chienda/thyma/issues)
- **Author:** [chienda.com](https://chienda.com)

-----

*Thyma is open source and MIT licensed. Built for the agent-native era.*

> *From the Greek root of hyperthymesia — the rare condition of perfect autobiographical memory.*
> *Your agent deserves the same.*
