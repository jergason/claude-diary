# Claude Code Hooks for Memory System

This directory contains hooks that automate diary entry creation.

## SessionEnd Hook

**File**: `session-end.sh`

**Purpose**: Automatically generates a diary entry when a Claude Code session ends.

### Installation

1. Copy the hook to your Claude Code hooks directory:

```bash
mkdir -p ~/.claude/hooks
cp hooks/session-end.sh ~/.claude/hooks/session-end.sh
chmod +x ~/.claude/hooks/session-end.sh
```

2. The hook will now run automatically at the end of every Claude Code session.

### How It Works

When a session ends, the hook:
1. Prints a message: "📝 Auto-generating diary entry for session..."
2. Runs the `/diary` command
3. Creates a diary entry at `~/.claude/memory/diary/YYYY-MM-DD-session-N.md`

### Manual Override

If you don't want automatic diary generation:
- Remove or rename the hook file
- Or add logic to skip certain sessions based on criteria (project, duration, etc.)

### Customization

You can customize the hook to:
- Only generate diary for sessions longer than X minutes
- Skip diary for specific projects
- Add notifications when diary is created
- Run additional cleanup tasks

Example with session length check:

```bash
#!/bin/bash
# Only create diary if session was longer than 10 minutes
SESSION_START=$(date -r ~/.claude/session_start +%s 2>/dev/null || echo 0)
SESSION_END=$(date +%s)
DURATION=$((SESSION_END - SESSION_START))

if [ $DURATION -gt 600 ]; then
  echo "📝 Auto-generating diary entry for session (${DURATION}s)..."
  echo "/diary"
else
  echo "⏭️ Skipping diary (session too short: ${DURATION}s)"
fi
```

## Other Potential Hooks

### Stop Hook (Alternative)

You could also use the `Stop` hook which triggers when Claude finishes responding:

```bash
# ~/.claude/hooks/stop.sh
#!/bin/bash
# Runs after every Claude response completes
# Less ideal for diary since it runs frequently, not just at session end
```

**Recommendation**: Use `SessionEnd` for diary generation, not `Stop`, since you want one diary per session, not per response.

## Troubleshooting

**Hook not running?**
- Ensure file is executable: `chmod +x ~/.claude/hooks/session-end.sh`
- Check file location: `~/.claude/hooks/session-end.sh` (not `.claude-config/`)
- Verify hook name matches exactly: `session-end.sh`

**Diary not being created?**
- Check that `/diary` command is available (plugin installed)
- Verify memory directory exists: `~/.claude/memory/diary/`
- Check Claude Code logs for errors

## Documentation

For more on Claude Code hooks, see: https://code.claude.com/docs/en/hooks-guide
