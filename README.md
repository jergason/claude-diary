# Claude Diary   

A long-term memory plugin for Claude Code that learns from your activity over time and continuously improves Claude's understanding of your preferences, patterns, and workflows.

- **Diary**: Generate a diary entry for any Claude Code session 
- **Reflection**: Performs reflection across multiple diary entries
- **Memory**: Reflection updates your `CLAUDE.md` with observed patterns and preferences

<img width="951" height="667" alt="cd" src="https://github.com/user-attachments/assets/d0b1deef-b027-4014-b761-67e36c221edc" />

## Quickstart 

1. **Clone this repository**:
```bash
git clone https://github.com/rlancemartin/claude-diary claude-diary
cd claude-diary
```

2. **Install commands**:
```bash
cp commands/*.md ~/.claude/commands/
```

3. **Install PreCompact hook** (for auto-diary generation):
```bash
mkdir -p ~/.claude/hooks
cp hooks/pre-compact.sh ~/.claude/hooks/pre-compact.sh
chmod +x ~/.claude/hooks/pre-compact.sh
```

4. **Use the system**:
   - Run `/diary` after important sessions (or let the PreCompact hook auto-generate)
   - After any collection of diary entries, run `/reflect` to analyze patterns
   - `CLAUDE.md` automatically updated with learnings

## Overview

`CLAUDE.md` files serve as [long-term memory](https://code.claude.com/docs/en/memory) for Claude Code. But, they are typically updated manually (e.g., via the `#` shortcut or `/memory` command). This plugin aims to automates the process of updating `CLAUDE.md` with the learnings from your activity over time. It is inspired by [an interview between Dan Shipper and Cat Wu / Boris Cherny from the Claude Code team](https://www.youtube.com/watch?v=IDSAMqip6ms&t=352s), which mentions that some Anthropic employees have started generating diary entries for Claude sessions and performing reflection over them. The approach follows that general flow, and also borrows ideas from the [Generative Agents paper](https://arxiv.org/pdf/2304.03442), which introduces a memory architecture with three key components:

1. **Observations**: Raw experiences recorded in natural language (like our diary entries)
2. **Reflection**: Higher-level insights and patterns (like our `/reflect` command)
3. **Retrieval**: Accessing memories to inform behavior (like reading CLAUDE.md into each session)

<img width="510" height="408" alt="Screenshot 2025-11-22 at 8 54 30 PM" src="https://github.com/user-attachments/assets/36c88072-9958-462e-b020-ad344445912a" />

### Diary Entries

The `/diary` command creates diary entries from the current session. It uses a context-first approach, reflecting on the conversation already in Claude Code's context, but can also parse JSONL transcripts, which are logged for each session to the project directory. See [diary.md](commands/diary.md) for details. The `diary` command can be run in two ways:

1. **Automatic**: PreCompact hook runs before conversation compacts (long sessions, 200+ messages). This could be customized to run at different points using different [Claude Code hooks](https://code.claude.com/docs/en/hooks-guide).
2. **Manual**: Run `/diary` anytime to capture important context in the current session

#### How the Diary Command Works

The command instructs Claude to create diary entries by reflecting on the conversation history in context (user messages, tool invocations, files modified, errors, solutions). It then instructs Claude to write the diary entry to a file in the `~/.claude/memory/diary/` directory as a markdown file with sections for task summary, work done, design decisions, user preferences, code review feedback, challenges, solutions, and code patterns. It instructs Claude to be factual and specific, capture the "why" behind decisions, document ALL user preferences (especially commits/PRs/linting/testing), include failures as learning. The output location is `~/.claude/memory/diary/YYYY-MM-DD-session-N.md`.

### Reflection

The `/reflect` command analyzes diary entries to identify patterns and automatically updates `CLAUDE.md`. See [reflect.md](commands/reflect.md) for details. The `reflect` command can be run in manually in any session; run `/reflect` anytime you have accumulated diary entries and want to update `CLAUDE.md` with new learnings.

#### How the Reflect Command Works

The reflect command instructs Claude to perform multi-diary pattern analysis. First, it has Claude check `processed.log` to skip already-analyzed diary entries (avoids duplicate analysis). Then, it has Claude recognize patterns that appear repeatedly (2+ occurrences = pattern, 3+ = strong pattern). In particular, it asks Claude to scan for violations of existing CLAUDE.md rules (highest priority for strengthening) and synthesize insights across 6 categories: PR review feedback, persistent preferences, design decisions that worked, anti-patterns to avoid, efficiency lessons, project-specific patterns. Finally, it has Claude auto-update CLAUDE.md with new rules that must be one-line bullets in imperative tone (no verbose explanations).

#### Reflect Command Options

```bash
# Analyze last 20 unprocessed entries
/reflect last 20 entries

# Analyze specific date range
/reflect from 2025-01-01 to 2025-01-31

# Filter by project
/reflect for project /Users/username/Code/my-app

# Filter by topic/keyword
/reflect related to testing

# Combine filters
/reflect last 15 entries for project /Users/username/Code/my-app related to React

# Re-analyze including already-processed entries
/reflect include all entries

# Re-analyze specific entry
/reflect reprocess 2025-11-07-session-1.md
```

## Directory Structure

This plugin follows the [Claude Code plugin structure](https://code.claude.com/docs/en/plugins):

```
cc-memory/
├── .claude-plugin/
│   └── plugin.json              # Plugin metadata
├── commands/
│   ├── diary.md                 # /diary command prompt
│   └── reflect.md               # /reflect command prompt
├── hooks/
│   ├── pre-compact.sh           # Auto-diary generation hook
│   └── README.md                # Hook installation guide
├── examples/
│   ├── sample-diary-entry.md    # Example diary output
│   └── sample-reflection.md     # Example reflection output
├── INSTALL.md                   # Installation instructions
├── LICENSE                      # MIT License
├── README.md                    # Main documentation
```

To modify or extend:
1. Edit command files in `commands/`
2. Test locally using the local marketplace approach
3. Share your improvements

## Future work

**Opportunities for improvement**:

1. **Project-level memory** - Support Claude Code's multiple memory tiers to enable context-appropriate rules (React patterns for React projects, Python conventions for Python projects)

2. **Session metadata API** - An official API for session introspection would eliminate JSONL parsing workarounds

3. **Adaptive reflection** - Optionally trigger reflection when N unprocessed entries accumulate, reducing manual overhead while preserving user control

4. **Memory retrieval** - Proactively surface relevant past learnings based on current context ("Last time you worked on authentication, you preferred JWT")

5. **Cross-project learning** - Detect when the same problem appears across different projects, suggesting generalizable solutions

## License

MIT License - feel free to use, modify, and share.
