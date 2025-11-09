# Claude Code Long-Term Memory System

## Overview
This document outlines a system for building long-term memory and learning patterns for Claude Code through diary entries and periodic reflection. The goal is to enable Claude to learn from past interactions and progressively refine its understanding of user preferences, project patterns, and effective problem-solving approaches.

**Key Insight from Cat Wu** (Anthropic): Some users create diary entries for every task documenting "what did it do, what did it try, why didn't it work," then have agents synthesize observations from past memory. Synthesizing memory from multiple logs helps identify true patterns vs. one-off requests. This is an emerging pattern that works well and could be productized.

## Goal
Create a memory system that:
- Captures what Claude learned from each interaction session
- Identifies patterns across multiple sessions through periodic reflection
- Automatically updates CLAUDE.md with actionable, generalizable insights
- Distinguishes between one-off preferences ("make this button pink") vs. persistent patterns ("always use TypeScript strict mode")

## How It Might Work

### 1. Diary Entry System

**Transcript Access**: Claude Code stores full conversation transcripts locally at:
- **Location**: `~/.claude/projects/[project-path]/[session-uuid].jsonl`
- **Format**: JSON Lines (JSONL) with message types: `summary`, `user`, `assistant`, `file-history-snapshot`, etc.
- **Access Method**: Read directly from these files or use `--resume` to interactively select past sessions
- **Note**: Hooks receive event-specific JSON data via stdin but don't have direct access to full transcripts

After each Claude Code session, create a structured diary entry that captures:
- **Date/Time**: When the interaction occurred
- **Task Summary**: What was the user trying to accomplish?
- **Actions Taken**: What did Claude do? What tools were used?
- **Challenges**: What didn't work? What errors occurred?
- **Solutions**: How were problems resolved?
- **User Preferences Observed**: Any explicit or implicit preferences noted
- **Code Patterns**: Architectural decisions, coding style, frameworks used
- **Context**: Project type, files modified, technologies involved

**Storage**: `~/.claude/memory/diary/YYYY-MM-DD-session-N.md` ✓ Directory confirmed writable

### 2. Reflection System
Periodically (weekly/monthly or after N sessions), run a reflection process that:
- Reads multiple diary entries from a time period
- Identifies patterns that appear repeatedly
- Distinguishes signal from noise (recurring patterns vs. one-off requests)
- Synthesizes insights at the right level of abstraction
- Generates candidate updates for CLAUDE.md

**Storage**: `~/.claude/memory/reflections/YYYY-MM-reflection.md`

### 3. CLAUDE.md Update Workflow
The reflection system proposes updates to CLAUDE.md:
- **Actionable Instructions**: Clear, concrete guidelines that Claude can follow
- **Project-Specific Patterns**: "In this codebase, always..."
- **General Preferences**: "User prefers X over Y when..."
- **Anti-Patterns to Avoid**: "Don't do X because..."

User reviews and approves/edits these proposed updates before they're added to CLAUDE.md.

### 4. Implementation Approach

#### Phase 1: Manual Diary Entries
- Custom slash command: `/diary` - reads the current session transcript from `~/.claude/projects/[project-path]/[session-uuid].jsonl` and creates a structured diary entry
- User manually runs this after interesting sessions
- Entries stored in structured markdown format at `~/.claude/memory/diary/`

#### Phase 2: Reflection Command
- Custom slash command: `/reflect` - analyzes diary entries and proposes CLAUDE.md updates
- Takes a date range or "last N entries" parameter
- Generates a reflection document with proposed changes
- User reviews and manually updates CLAUDE.md

#### Phase 3: Automated Retrieval (Future)
- Integrate with Claude Code's context system
- Automatically surface relevant past diary entries when working on similar tasks
- Vector similarity search over diary entries for "show me how I solved X before"

## Key Benefits
1. **Persistent Learning**: Claude learns your patterns over time
2. **Reduced Repetition**: Stop re-explaining the same preferences
3. **Better Context**: Past solutions inform current problems
4. **Transparent Memory**: You control what goes into CLAUDE.md
5. **Pattern Recognition**: Identify implicit preferences you didn't realize you had

## Implementation Strategy
Start simple (manual diary entries via slash command), then progressively automate. Focus on making diary entries easy to write and structured enough to be useful for later reflection.

---

## Claude Code Directory Structure

### Overview: `~/.claude/`
The Claude Code configuration and data directory contains all session data, transcripts, settings, and user customizations.

### Key Directories

#### **`projects/`** - Conversation Transcripts by Project
- **Location**: `~/.claude/projects/[encoded-project-path]/`
- **Purpose**: Stores full conversation transcripts for each session
- **File Format**: JSONL (JSON Lines) - one JSON object per line
- **Naming**:
  - Session files: `[session-uuid].jsonl` (e.g., `a6c7742e-24f5-4075-b59d-b315ef66173d.jsonl`)
  - Agent files: `agent-[short-id].jsonl` (e.g., `agent-f7473b50.jsonl`)

**Transcript Structure** (JSONL format):
```jsonl
{"type": "summary", "leafUuid": "...", "summary": "..."}
{"type": "file-history-snapshot", ...}
{"type": "user", "message": {...}, "uuid": "...", "timestamp": ..., "sessionId": "..."}
{"type": "assistant", "message": {...}, "uuid": "...", "timestamp": ..., "toolUseResult": {...}}
```

**Message Fields**:
- `type`: Message type (`summary`, `user`, `assistant`, `file-history-snapshot`)
- `message`: The actual message content (for user/assistant messages)
- `uuid`: Unique identifier for this message
- `parentUuid`: UUID of the parent message (conversation threading)
- `sessionId`: Session UUID this message belongs to
- `timestamp`: Unix timestamp (milliseconds)
- `cwd`: Current working directory
- `gitBranch`: Active git branch at time of message
- `toolUseResult`: Results from tool invocations (for assistant messages)
- `version`: Schema version
- `userType`: Type of user message
- `isSidechain`: Whether this is a subagent conversation

#### **`file-history/[session-uuid]/`** - File Version History
- **Purpose**: Stores versions of files edited during each session
- **Format**: `[file-hash]@v[version-number]` (e.g., `235c5b0f83d15d8d@v1`, `@v2`, `@v3`)
- **Use Case**: Enables undo/redo, version comparison, and session replay

#### **`history.jsonl`** - User Prompt History
- **Purpose**: Records all user prompts across sessions
- **Format**: JSONL with fields:
  ```json
  {
    "display": "user's prompt text",
    "pastedContents": {},
    "timestamp": 1762488120792,
    "project": "/Users/rlm/Desktop/Code/notes",
    "sessionId": "a6c7742e-24f5-4075-b59d-b315ef66173d"
  }
  ```
- **Use Case**: Command history, prompt reuse, session lookup

#### **`commands/`** - Custom Slash Commands
- **Purpose**: User-defined slash commands (markdown files)
- **Format**: `[command-name].md` files containing command prompts
- **Example**: `/todos` → `~/.claude/commands/todos.md`

#### **`memory/`** - Long-term Memory Storage (Custom)
- **Purpose**: User-created directory for diary entries and reflections
- **Subdirectories**:
  - `diary/`: Session diary entries
  - `reflections/`: Synthesized insights from diary entries

#### **`skills/`** - Custom Skills
- **Purpose**: User and project-specific skill definitions
- **Example**: `langgraph-docs/` skill for LangGraph documentation access

#### **`debug/`** - Debug Logs
- **Purpose**: Detailed debug logs for each session
- **Format**: `[session-uuid].txt` files with timestamped debug output
- **Content**: LSP events, plugin loading, file operations, system events

#### **`session-env/[session-uuid]/`** - Session Environments
- **Purpose**: Stores session-specific environment state
- **Use Case**: Environment variables, shell state per session

#### **`shell-snapshots/`** - Shell Configuration Snapshots
- **Purpose**: Captured shell environment configurations
- **Use Case**: Reproducing shell state for command execution

#### **`todos/`** - Session Todo Lists
- **Purpose**: Stores todo list state for active sessions
- **Format**: `[session-uuid]-agent-[agent-id].json`

#### **Configuration Files**
- **`CLAUDE.md`**: Global instructions for Claude Code (user preferences, patterns)
- **`settings.json`**: User settings and permissions
- **`statusline-script.sh`**: Custom status line script

### Access Patterns for Memory System

**Reading Current Session Transcript**:
```bash
# Get current session ID from environment or most recent file
SESSION_ID=$(ls -t ~/.claude/projects/[project-path]/*.jsonl | head -1 | xargs basename .jsonl)

# Read full transcript
cat ~/.claude/projects/[project-path]/$SESSION_ID.jsonl
```

**Finding Session by Date**:
```bash
# List sessions by modification time
ls -lt ~/.claude/projects/[project-path]/*.jsonl

# Sessions modified today
find ~/.claude/projects/[project-path]/ -name "*.jsonl" -mtime -1
```

**Parsing Transcript Messages**:
```bash
# Extract all user messages
jq -r 'select(.type == "user") | .message' session.jsonl

# Extract assistant tool uses
jq -r 'select(.type == "assistant" and .toolUseResult) | .toolUseResult' session.jsonl
```

---

## Additional Context

### Cat's Implementation Suggestions
- Add a custom slash command to tell Claude Code to write down its learnings from the transcript in a folder
- After accumulating several diary entries, ask Claude Code to summarize them
- Point Claude Code to the raw transcripts which are stored locally

### Source
Raw notes from podcast with Cat Wu and Boris Cherny (https://www.youtube.com/watch?v=IDSAMqip6ms&t=352s): 

a similar feature in the past, cloud code can use those APIs like query GitHub directly and find how people
27:19
implemented a similar feature in the past and read that code and um copy the
27:24
relevant parts. Yeah. Is there um have you found any use for like log files of okay you know
27:30
here's here's the full history of like how I implemented it and like is that important to give to claude and and and
27:37
how are you how are you um implementing that or making it useful for it? Some people swear by it. Uh there are
27:44
some people at anthropic where for every task they do, they tell cloud code to write a diary entry in a specific format
27:50
that just documents like what did it do, what did it try, why didn't it work, and then they even have these agents that
27:56
like look over the past memory and synthesize it into observations. I think this is like the starting
28:02
budding like there's like something interesting here that we could productize.
28:08
Um but it's a new emerging pattern that we're seeing that works well. I think the hard thing about like oneshotting
28:14
memory from just one transcript is that it's hard to know how relevant a
28:19
specific instruction is to all future tasks. Like our canonical example is if
28:24
I say make the button pink, I don't want you to remember to make all buttons pink in the future. And so I think um
28:31
synthesizing memory from a lot of logs is a is a way to um find these patterns
28:36
more um consistently. It seems like you probably need like there's some things where you're going to know um you'll be
28:44
able to summar like synthesize or summarize in this sort of like top down way like this this will be useful later
28:50
and and you'll you'll know the right level of abstraction at which it might be useful but then there's also a lot of
28:55
stuff where it's like you actually you know any given like commit log like make the button pink it could be useful for
29:03
kind of an infinite number of different reasons um that you're not going to know beforehand. So you also need the the
29:09
model to be able to look up all similar past, you know, commits and surface that
29:15
at the right time. Is that something that you're also thinking about? Yeah, I think I think there could there could be
29:20
something like that. And maybe I think one way to see it is this kind of like traditional memory storage work like
29:26
like mex like kind of stuff where you just want to like put all the information into the system and then
29:31
it's kind of a retrieval problem problem after that. Um,
29:36
yeah. I think as the model also gets smarter, it naturally I've seen it start to naturally do this also with Sonnet 45
29:43
where if it's stuck on something, it'll just naturally start looking like we talked about before like using bash
29:48
spontaneously. So just like look through git history and be like, "Oh, okay. Yeah, this is kind of an interesting way to do it."
29:54
Yeah. One of the things that like we were talking before we started recording, one of the um things that
29:59
we're doing inside of every like I feel like it has really um change the way that we do engineering because everyone
30:05
