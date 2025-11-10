# Claude Code Memory System

A long-term memory plugin for Claude Code that enables learning from past interactions through structured diary entries and periodic reflection.

## Overview

This plugin implements a two-phase memory system inspired by [Cat Wu's approach at Anthropic](https://www.youtube.com/watch?v=IDSAMqip6ms&t=352s):

1. **Diary Entries**: Document what Claude learned from each session
2. **Reflection**: Analyze patterns across multiple sessions and propose actionable updates to `CLAUDE.md`

The key insight: Synthesizing memory from multiple sessions helps identify true patterns versus one-off requests (e.g., "always use TypeScript strict mode" vs. "make this button pink").

## Installation

### Option 1: Install as Plugin

1. Clone or download this repository
2. Create a local marketplace file (or add to existing one):

```bash
mkdir -p ~/.claude/marketplaces/local
```

Create `~/.claude/marketplaces/local/marketplace.json`:

```json
{
  "name": "local-plugins",
  "owner": {
    "name": "Local"
  },
  "plugins": [
    {
      "name": "claude-memory",
      "source": "/path/to/cc-memory",
      "description": "Long-term memory system for Claude Code"
    }
  ]
}
```

3. In Claude Code, install the plugin from your local marketplace

### Option 2: Manual Setup

Copy the commands to your global Claude Code commands directory:

```bash
cp commands/*.md ~/.claude/commands/
```

## Usage

### Creating Diary Entries

**Automatic (Recommended)**: Install the SessionEnd hook for automatic diary generation:

```bash
mkdir -p ~/.claude/hooks
cp hooks/session-end.sh ~/.claude/hooks/session-end.sh
chmod +x ~/.claude/hooks/session-end.sh
```

Now a diary entry will be automatically created at the end of every Claude Code session.

**Manual**: You can also run the diary command manually:

```bash
/diary
```

This will:
- Parse the current session transcript from `~/.claude/projects/...`
- Extract user messages, assistant actions, challenges, and solutions
- Generate a structured diary entry at `~/.claude/memory/diary/YYYY-MM-DD-session-N.md`

**Diary entries capture:**
- Task summary and goals
- Design decisions made (architectural choices, technology selections, trade-offs)
- Actions taken (files edited, commands run, tools used)
- Code review & PR feedback (avoiding "AI-looking" code, test coverage, style)
- Challenges encountered (errors, failed approaches)
- Solutions applied (what worked and why)
- User preferences observed (commit style, testing requirements, workflow preferences)
- Code patterns and architectural decisions
- Project context and technologies

### Reflecting on Patterns

After accumulating several diary entries (recommended: 5-10+), run:

```bash
/reflect
```

**Filter options:**

```bash
# Analyze last 20 unprocessed entries (default: skips already-processed entries)
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

This will:
- Check `~/.claude/memory/reflections/processed.log` for already-analyzed entries
- Read and analyze unprocessed diary entries
- Identify patterns that appear 2+ times (persistent preferences)
- Extract PR review feedback patterns (avoiding "AI-looking" code)
- Distinguish project-specific patterns from universal ones
- Identify effective approaches and anti-patterns to avoid
- Generate a reflection document at `~/.claude/memory/reflections/YYYY-MM-reflection-N.md`
- **Automatically append actionable rules to your `~/.claude/CLAUDE.md` file**
- Update `processed.log` with analyzed entries

**Processed Entries Tracking**: The system maintains `~/.claude/memory/reflections/processed.log` to track which diary entries have been analyzed. This prevents duplicate analysis and allows you to run `/reflect` ad hoc as new diary entries accumulate.

## Directory Structure

The plugin creates and uses these directories:

```
~/.claude/
├── hooks/
│   └── session-end.sh           # Auto-generates diary at session end
├── memory/
│   ├── diary/                   # Session diary entries
│   │   ├── 2025-01-15-session-1.md
│   │   ├── 2025-01-15-session-2.md
│   │   └── 2025-01-16-session-1.md
│   └── reflections/             # Synthesized insights & tracking
│       ├── 2025-01-reflection-1.md
│       ├── 2025-02-reflection-1.md
│       └── processed.log        # Tracks which entries have been analyzed
└── CLAUDE.md                    # Rules automatically updated by /reflect
```

These directories are automatically created on first use.

**processed.log format**: `[diary-entry] | [reflection-date] | [reflection-file]`

## How It Works

### Session Transcripts

Claude Code stores full conversation transcripts at:
```
~/.claude/projects/[encoded-project-path]/[session-uuid].jsonl
```

Each transcript is in JSONL format (JSON Lines) containing:
- User messages
- Assistant responses
- Tool invocations and results
- File operations
- Error messages
- Git operations

The `/diary` command parses these transcripts using `jq` (JSON processor) to create structured memory entries. See the diary command documentation for detailed jq usage examples and parsing strategies.

### Pattern Recognition

The `/reflect` command uses several principles to distinguish signal from noise:

**Signal** (becomes a rule):
- Appears 2+ times across multiple sessions (strong patterns: 3+)
- Consistent (not contradicted by other preferences)
- Generalizable (not too specific to one case)
- Actionable (Claude can follow the instruction)
- Succinct (can be expressed as one-line bullet point)

**Noise** (documented but not added to CLAUDE.md):
- One-off requests specific to a single task
- Contradictory to established patterns
- Too context-specific to be useful generally

**Processed entries tracking**: The reflect command maintains `~/.claude/memory/reflections/processed.log` to track which diary entries have been analyzed, preventing duplicate analysis and allowing ad hoc incremental reflection.

### Example Pattern Evolution

```
Session 1: User asks to use TypeScript strict mode
Session 3: User corrects code to follow strict mode
Session 5: User mentions strict mode again
Session 8: User emphasizes importance of strict types

→ Reflection identifies pattern: "Always use TypeScript strict mode"
→ Proposes CLAUDE.md update: "For TypeScript projects, always enable strict mode in tsconfig.json"
```

## Best Practices

### When to Create Diary Entries

Run `/diary` after sessions that involve:
- ✅ Solving a challenging problem
- ✅ Making architectural decisions
- ✅ Learning user preferences
- ✅ Trying multiple approaches (especially if some failed)
- ✅ Working with new technologies or patterns

Skip `/diary` for:
- ❌ Trivial one-line changes
- ❌ Simple questions with quick answers
- ❌ Sessions with no meaningful learning

### When to Reflect

Run `/reflect` when:
- ✅ You have 5-10+ diary entries accumulated
- ✅ You want to identify patterns in a specific area (e.g., testing, React)
- ✅ You're switching projects and want to capture learnings
- ✅ Monthly or weekly as a regular practice
- ✅ Your `CLAUDE.md` feels incomplete or outdated

### Reviewing Proposed Updates

When `/reflect` proposes CLAUDE.md updates:

1. **Check generalizability**: Will this apply to future sessions?
2. **Check consistency**: Does this contradict existing instructions?
3. **Check actionability**: Can Claude actually follow this instruction?
4. **Check specificity**: Is it concrete enough to be useful?
5. **Edit before applying**: Feel free to refine the proposed text

## Examples

### Example Diary Entry Structure

```markdown
# Session Diary Entry

**Date**: 2025-01-15
**Time**: 14:30:22
**Session ID**: a6c7742e-24f5-4075-b59d-b315ef66173d
**Project**: /Users/rlm/Desktop/Code/my-react-app
**Git Branch**: feature/user-auth

## Task Summary
User wanted to implement user authentication using JWT tokens with React context for state management.

## Actions Taken
- Created AuthContext.tsx with JWT token management
- Implemented login/logout methods
- Added protected route wrapper component
- Updated App.tsx to wrap routes with AuthProvider
- Created useAuth custom hook for consuming auth context

## Challenges Encountered
- Initial approach stored token in component state, which cleared on refresh
- JWT decode library had TypeScript type issues
- Protected route redirects caused navigation loop

## Solutions Applied
- Moved token storage to localStorage for persistence
- Used @types/jwt-decode package for TypeScript support
- Added redirect location tracking to prevent navigation loops

## User Preferences Observed
- Prefers functional components over class components
- Wants TypeScript strict mode enabled
- Likes custom hooks for reusable logic
- Prefers explicit error handling over silent failures

## Code Patterns and Decisions
- React Context pattern for global state
- Custom hooks for auth logic encapsulation
- Protected route HOC pattern
- JWT token storage in localStorage
- Redirect state tracking in location.state

## Context and Technologies
- React 18 with TypeScript
- React Router v6
- JWT for authentication
- Vite build tool
```

### Example Reflection Output

```markdown
# Reflection: Last 10 Entries

**Generated**: 2025-01-20 10:15:00
**Entries Analyzed**: 10
**Date Range**: 2025-01-10 to 2025-01-18
**Projects**: All projects

## Summary
Analyzed 10 diary entries spanning 8 days across 3 different projects. Strong patterns emerged around TypeScript preferences, React coding style, and testing practices. User consistently prefers functional patterns, strict typing, and comprehensive error handling.

## Patterns Identified

### Persistent Preferences (3+ occurrences)

1. **TypeScript Strict Mode** (appeared in 8/10 entries)
   - **Observation**: User consistently enables and enforces TypeScript strict mode
   - **Evidence**: Sessions 1, 2, 3, 5, 6, 7, 9, 10 all involved strict mode
   - **Confidence**: High
   - **Proposed CLAUDE.md rule**: "For all TypeScript projects, enable strict mode in tsconfig.json and avoid using 'any' types"

2. **Functional React Components** (appeared in 7/8 React sessions)
   - **Observation**: User always requests functional components, never class components
   - **Evidence**: All React sessions used functional components with hooks
   - **Confidence**: High
   - **Proposed CLAUDE.md rule**: "In React projects, always use functional components with hooks. Never suggest class components."

3. **Explicit Error Handling** (appeared in 6/10 entries)
   - **Observation**: User wants errors surfaced clearly, not silently caught
   - **Evidence**: User corrected code that swallowed errors in sessions 2, 4, 6, 7, 9, 10
   - **Confidence**: High
   - **Proposed CLAUDE.md rule**: "Always implement explicit error handling. Avoid empty catch blocks or silent failures. Log errors or surface them to the user."

## Proposed CLAUDE.md Updates

### Section: TypeScript Preferences

\`\`\`markdown
## TypeScript Configuration
- Always enable strict mode in tsconfig.json
- Avoid using 'any' types - use 'unknown' or proper types instead
- Prefer interfaces over type aliases for object shapes
- Enable all strict type checking options (strictNullChecks, strictFunctionTypes, etc.)
\`\`\`

### Section: React Coding Style

\`\`\`markdown
## React Best Practices
- Use functional components with hooks exclusively
- Never suggest class components
- Prefer custom hooks for reusable logic
- Use React Context for global state management
- Implement proper cleanup in useEffect hooks
\`\`\`
```

## Roadmap

### Phase 1: Core Memory System ✅ (Complete)
- `/diary` command for diary creation
- Automatic diary generation via SessionEnd hook
- `/reflect` command for pattern analysis
- Automatic CLAUDE.md updates with actionable rules
- processed.log tracking for incremental reflection
- Ad hoc reflection runs on accumulated entries

### Phase 2: Enhanced Analysis (Future)
- Vector similarity search over diary entries
- "Show me how I solved X before" retrieval
- Confidence scoring for patterns
- Cross-session pattern validation

### Phase 3: Integration (Future)
- Automatic context surfacing in new sessions
- Proactive pattern suggestions
- Cross-project learning
- Collaborative memory (team settings)

## Troubleshooting

### "Cannot find session transcript"

The `/diary` command looks for transcripts in `~/.claude/projects/`. If it can't find the current session:
1. Check that you're in a Claude Code session (not just a shell)
2. Verify the project path is correct
3. Try running `/diary` at the end of the session (not during)

### "No diary entries found"

Run `/diary` first to create some diary entries before running `/reflect`.

### "Cannot create memory directory"

Ensure you have write permissions to `~/.claude/memory/`:
```bash
mkdir -p ~/.claude/memory/diary
mkdir -p ~/.claude/memory/reflections
```

## Contributing

This plugin follows the [Claude Code plugin structure](https://code.claude.com/docs/en/plugins):

```
cc-memory/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   ├── diary.md
│   └── reflect.md
├── README.md
└── scope.md
```

To modify or extend:
1. Edit command files in `commands/`
2. Test locally using the local marketplace approach
3. Share your improvements

## Credits

- Inspired by [Cat Wu's diary entry pattern](https://www.youtube.com/watch?v=IDSAMqip6ms&t=352s) at Anthropic
- Built for the Claude Code community
- Author: rlm

## License

MIT License - feel free to use, modify, and share.

---

**Note**: This is Phase 1 of the memory system. It focuses on manual diary entries and reflection. Future phases will add automation, retrieval, and deeper integration with Claude Code's context system.
