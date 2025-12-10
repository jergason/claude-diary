# Claude Code Hooks for Memory System

This directory contains hooks that automate diary entry creation.

## PreCompact Hook (Recommended)

**Purpose**: Automatically generates a diary entry before Claude Code compacts the conversation.

### Why PreCompact?

- ✅ Runs **while Claude Code is still active** (can invoke `/diary` command)
- ✅ Triggers at **natural checkpoints** (when conversation context gets large)
- ✅ Automatic but **not too frequent** (only when compacting is needed)
- ✅ Captures session state at meaningful milestones

### Installation

**If installed as a plugin**: The hook works automatically! The `hooks/hooks.json` config registers the PreCompact hook when the plugin is installed.

**If using standalone** (without the plugin): Register the hook manually in `~/.claude/settings.json`:

1. Copy the hook script somewhere accessible:

```bash
mkdir -p ~/.claude/hooks
cp hooks/pre-compact.sh ~/.claude/hooks/pre-compact.sh
chmod +x ~/.claude/hooks/pre-compact.sh
```

2. Add to your settings file (`~/.claude/settings.json`):

```json
{
  "hooks": {
    "PreCompact": [
      {
        "matcher": "auto",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/pre-compact.sh"
          }
        ]
      },
      {
        "matcher": "manual",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/pre-compact.sh"
          }
        ]
      }
    ]
  }
}
```

**Note**: Hooks require explicit registration - they're not auto-discovered from directories. The `auto` matcher triggers when context fills up; `manual` triggers from the `/compact` command.

3. Restart Claude Code for hooks to take effect (hooks are loaded at startup).

### How It Works

When Claude Code is about to compact the conversation, the hook:

1. Prints a message: "📝 Auto-generating diary entry before compact..."
2. Invokes the `/diary` command
3. Creates a diary entry at `~/.claude/memory/diary/YYYY-MM-DD-session-N.md`
4. Compact operation proceeds normally

**Frequency**: Depends on session length and conversation size. Typically:

- Short sessions (< 50 messages): May not trigger at all
- Medium sessions (50-200 messages): 1-2 diary entries
- Long sessions (200+ messages): Multiple diary entries capturing different phases

### Manual Diary Creation

You can **always** run `/diary` manually at any time:

```bash
/diary
```

**When to use manual `/diary`:**

- ✅ At the end of important work sessions
- ✅ After making significant decisions or discoveries
- ✅ Before switching to a different project
- ✅ Anytime you want to capture the current session state

**Best practice**: Use both! PreCompact hook handles automatic periodic captures, and you run `/diary` manually for important milestones.

### Disabling Automatic Diary

If you don't want automatic diary generation:

- Remove or rename the hook file: `rm ~/.claude/hooks/pre-compact.sh`
- Or add conditional logic to skip certain sessions

### Customization Examples

**Skip diary for short sessions:**

```bash
#!/bin/bash
# Only create diary if session has significant activity

# Count messages in current session (example logic)
if [ -f ~/.claude/session_message_count ]; then
  MSG_COUNT=$(cat ~/.claude/session_message_count)
  if [ $MSG_COUNT -lt 20 ]; then
    echo "⏭️ Skipping diary (only $MSG_COUNT messages)"
    exit 0
  fi
fi

echo "📝 Auto-generating diary entry before compact..."
echo "/diary"
```

**Skip diary for specific projects:**

```bash
#!/bin/bash
# Skip diary for scratch/test projects

PROJECT_DIR=$(pwd)
if [[ "$PROJECT_DIR" == *"/scratch/"* ]] || [[ "$PROJECT_DIR" == *"/tmp/"* ]]; then
  echo "⏭️ Skipping diary for scratch project"
  exit 0
fi

echo "📝 Auto-generating diary entry before compact..."
echo "/diary"
```

## Troubleshooting

**Hook not running?**

- Ensure hook is registered in `~/.claude/settings.json` (not just placed in directory)
- Ensure file is executable: `chmod +x ~/.claude/hooks/pre-compact.sh`
- Verify the path in settings.json matches the actual script location
- Restart Claude Code (hooks are loaded at startup)

**Diary not being created?**

- Verify `/diary` command is available (commands installed)
- Check memory directory exists: `~/.claude/memory/diary/`
- Try running `/diary` manually to test
- Check Claude Code output for hook execution messages

**Too many diary entries?**

- This is actually fine! Reflection will synthesize across all entries
- Or add conditional logic to reduce frequency (see customization examples)

**No diary entries being created?**

- PreCompact only triggers on long sessions with many messages
- For short sessions, use manual `/diary` command instead
- Or use both automatic (PreCompact) and manual approaches

## Two-Approach Strategy (Recommended)

**Automatic (PreCompact hook)**: Captures long sessions automatically

- Install the hook and forget about it
- Handles complex, multi-hour sessions with many exchanges
- Creates checkpoints during extended work

**Manual (/diary command)**: Captures important moments

- Run `/diary` at end of significant work
- Capture key decisions, breakthroughs, or completions
- Works for sessions of any length

**Combined approach**: Best of both worlds - automatic capture for long sessions, manual control for important moments.

## Documentation

For more on Claude Code hooks, see: https://code.claude.com/docs/en/hooks-guide

### Available Hook Events

- `PreToolUse` - Before tool calls
- `PostToolUse` - After tool calls
- `UserPromptSubmit` - When user submits prompts
- `Notification` - When Claude Code sends notifications
- `Stop` - When Claude finishes responding
- `SubagentStop` - When subagent tasks complete
- **`PreCompact`** - Before compact operations ← **We use this!**
- `SessionStart` - When Claude Code starts/resumes
- `SessionEnd` - When session terminates
