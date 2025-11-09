---
description: Create a structured diary entry from the current session transcript
---

# Create Diary Entry from Current Session

You are going to create a structured diary entry that documents what happened in the current Claude Code session. This entry will be used later for reflection and pattern identification.

## Steps to Follow

1. **Locate the current session transcript**:
   - The current working directory is: `{{ cwd }}`
   - Session transcripts are stored at: `~/.claude/projects/[encoded-project-path]/[session-uuid].jsonl`
   - Find the most recent session file for this project
   - The file is in JSONL format (JSON Lines - one JSON object per line)

   **Quick Overview Commands**:
   ```bash
   # Check file size and line count
   ls -lh ~/.claude/projects/[project-path]/[session-uuid].jsonl
   wc -l ~/.claude/projects/[project-path]/[session-uuid].jsonl

   # Get first few lines to understand structure
   head -5 [session-file].jsonl
   ```

2. **Parse the transcript** using jq (JSON processor) and grep:

   **Important**: Session transcript files can be large (1-2MB). Use strategic sampling and filtering.

   **jq** is a command-line JSON processor - like `sed`/`awk` for JSON. Use it to parse JSONL files.

   ### Extract User Messages (What was requested):
   ```bash
   # All user messages
   jq -r 'select(.type == "user") | .message.content' [session-file].jsonl

   # First 10 user messages (for sampling)
   jq -r 'select(.type == "user") | .message.content' [session-file].jsonl | head -10

   # Or use grep for faster search on large files
   grep '"type":"user"' [session-file].jsonl | head -10
   ```

   ### Extract Session Metadata:
   ```bash
   # Get session summaries (if they exist)
   jq -r 'select(.type == "summary") | .summary' [session-file].jsonl

   # Get git branch
   jq -r '.gitBranch' [session-file].jsonl | head -1

   # Get timestamp from first message
   jq -r '.timestamp' [session-file].jsonl | head -1
   ```

   ### Extract Tool Usage (What Claude did):
   ```bash
   # Count tools used
   jq -r 'select(.message.content[]?.type == "tool_use") | .message.content[].name' [session-file].jsonl | sort | uniq -c

   # Find specific tool uses (e.g., Edit operations)
   jq 'select(.message.content[]?.name == "Edit")' [session-file].jsonl
   ```

   ### Extract File Operations:
   ```bash
   # Files that were edited (from toolUseResult)
   jq -r 'select(.toolUseResult.filePath) | .toolUseResult.filePath' [session-file].jsonl | sort -u

   # Files that were read
   jq -r 'select(.message.content[]?.name == "Read") | .message.content[].input.file_path' [session-file].jsonl

   # Or use grep for file paths
   grep -o '"filePath":"[^"]*"' [session-file].jsonl | sort -u
   ```

   ### Extract Errors and Challenges:
   ```bash
   # Find stderr output (errors)
   jq 'select(.toolUseResult.stderr != "")' [session-file].jsonl

   # Find failed commands/tools
   jq 'select(.message.content[]?.is_error == true)' [session-file].jsonl

   # Find error exit codes
   grep '"Exit code' [session-file].jsonl
   ```

   ### Extract Git Operations:
   ```bash
   # Find git commands
   grep -E '(git commit|git push|git merge|git checkout)' [session-file].jsonl

   # Extract commit messages
   jq -r 'select(.message.content[]?.input.command | contains("git commit")) | .message.content[].input.command' [session-file].jsonl
   ```

   ### Extract PR/Code Review Comments:
   ```bash
   # Look for pr-comments or gh commands
   grep -i 'pr-comment\|gh pr\|gh api' [session-file].jsonl

   # Look for review feedback mentions
   grep -i 'review\|feedback\|comment' [session-file].jsonl
   ```

   **Key Information to Extract**:
   - All user messages (lines with `"type": "user"`)
   - All assistant messages and tool uses (lines with `"type": "assistant"`)
   - Tool use results (check `toolUseResult` field)
   - Any errors or challenges encountered
   - Files that were created, edited, or read
   - Git operations (commits, PRs, branches)
   - Code review feedback or PR comments mentioned
   - Linting, testing, or build operations
   - User corrections or feedback on Claude's work

   **Strategy for Large Files**:
   - Use `head` and `tail` to sample from beginning and end
   - Use `grep` for keyword searches (faster than jq on large files)
   - Use jq with `head` to limit output: `jq ... | head -20`
   - For files >1MB, use Task tool with general-purpose agent to analyze

3. **Create the diary entry directory** (if it doesn't exist):
   - Directory: `~/.claude/memory/diary/`
   - Use `mkdir -p` to create it automatically

4. **Generate a structured diary entry** in markdown format with these sections:

```markdown
# Session Diary Entry

**Date**: [YYYY-MM-DD]
**Time**: [HH:MM:SS]
**Session ID**: [session-uuid]
**Project**: [project-path]
**Git Branch**: [branch-name if available]

## Task Summary
[What was the user trying to accomplish? Summarize the main goal(s) in 2-3 sentences]

## Work Summary
[High-level summary of what was accomplished:]
- Features implemented
- Bugs fixed
- Refactoring completed
- Documentation added
- Tests written
- Build/deployment changes

## Design Decisions Made
[IMPORTANT: Document key technical decisions and their rationale:]
- Architectural choices (e.g., "Used React Context instead of Redux because...")
- Technology selections (e.g., "Chose PostgreSQL over MongoDB because...")
- API design decisions (e.g., "Made endpoint RESTful instead of GraphQL because...")
- Data structure choices (e.g., "Used array instead of Set because...")
- Pattern selections (e.g., "Applied Factory pattern for...")
- Trade-offs considered and why specific choices were made

## Actions Taken
[List the key actions Claude took, including:]
- Files read, created, or edited (with file paths)
- Commands executed (bash, npm, git, etc.)
- Tools used (Read, Edit, Write, Grep, etc.)
- Code changes made (be specific about what changed)
- Tests run (pass/fail status)
- Linting or formatting applied
- Git operations (commits, branches, PRs)

## Code Review & PR Feedback
[IMPORTANT: Capture any code review comments or PR-related feedback:]
- PR comments mentioned (e.g., "looks like AI wrote this - make it more natural")
- Code quality feedback (e.g., "add more tests", "improve variable naming")
- Linting issues found and fixed
- Review requests or changes requested
- Any discussion about commit messages or PR descriptions
- Feedback about code style or patterns

## Challenges Encountered
[Document what didn't work on the first try:]
- Errors encountered (with error messages)
- Failed approaches (what was tried and why it failed)
- Debugging steps taken
- Tool failures or limitations
- Tests that failed initially
- Build or compilation issues

## Solutions Applied
[How were problems resolved?]
- What worked and why
- How errors were fixed
- Alternative approaches considered
- What was learned from the failures

## User Preferences Observed
[CRITICAL: Document ALL user preferences, especially workflow preferences:]

### Commit & PR Preferences:
- Commit message style (e.g., "don't add 'Generated by Claude Code' to commits")
- PR description preferences (e.g., "no Claude Code links in PR descriptions")
- When to commit (e.g., "always run tests before committing")
- Branch naming conventions

### Code Quality Preferences:
- Linting requirements (e.g., "always run lint before committing")
- Testing requirements (e.g., "ensure tests pass", "add tests for new features")
- Code review preferences (e.g., "make code look human-written")
- Documentation requirements

### Technical Preferences:
- Libraries and frameworks preferred
- Code patterns and styles
- Language features to use/avoid
- File organization preferences

### Communication Preferences:
- How user likes to receive information
- Level of detail preferred
- Explanation style

## Code Patterns and Decisions
[Technical patterns and architectural decisions:]
- Design patterns used (Factory, Observer, Strategy, etc.)
- Code organization structure chosen
- Testing approach and framework
- Error handling strategy
- State management approach
- API design patterns

## Context and Technologies
[Project context:]
- Project type (web app, CLI tool, library, etc.)
- Primary languages used
- Frameworks and libraries involved
- Key files or modules worked on

## Notes
[Any other observations or context that might be useful for future reference]
```

5. **Save the diary entry**:
   - Filename format: `YYYY-MM-DD-session-N.md` where N is incremented if multiple sessions occur on the same day
   - Save to: `~/.claude/memory/diary/[filename]`
   - Check if files with the same date prefix exist and increment N appropriately

6. **Confirm completion**:
   - Display the path where the diary entry was saved
   - Show a brief summary of what was captured (number of user messages, assistant actions, challenges documented)

## Important Guidelines

- **Be factual and specific**: Include concrete details (file names, line numbers, error messages)
- **Capture the 'why'**: Don't just list actions, explain the reasoning behind decisions
- **Document design decisions**: Always explain WHY a particular approach was chosen over alternatives
- **Capture PR/review feedback**: This is CRITICAL - any feedback about code quality, style, or appearing AI-generated must be documented
- **Note ALL preferences**: Especially around commits, PRs, linting, testing - these are high-value patterns
- **Distinguish one-off from patterns**: Mark whether a preference seems specific to this task or generally applicable
- **Include failures**: Document what didn't work - this is valuable learning
- **Keep it structured**: Follow the template consistently for easier analysis later
- **Pay attention to corrections**: When the user corrects Claude's work, that's a strong signal about preferences

## Error Handling

- If you cannot find the session transcript, explain where you looked and ask the user for help
- If the `~/.claude/memory/diary/` directory cannot be created, report the error
- If any part of the transcript is malformed or unreadable, document what you could parse and note the issue

## Advanced Tips: Analyzing Large Transcripts

### When Session Files are >500KB:

**Option 1: Use Task tool with general-purpose agent**
- For files >1MB, delegate analysis to Task tool
- Provide agent with specific extraction instructions
- Agent can read file in chunks and synthesize findings

**Option 2: Strategic Sampling**
```bash
# Sample from beginning (setup phase)
head -100 [session-file].jsonl | jq 'select(.type == "user")'

# Sample from end (completion phase)
tail -100 [session-file].jsonl | jq 'select(.type == "user")'

# Sample from middle
sed -n '200,300p' [session-file].jsonl | jq 'select(.type == "user")'
```

**Option 3: Targeted grep searches**
```bash
# Count occurrences of key activities
grep -c '"name":"Edit"' [session-file].jsonl    # How many edits?
grep -c '"name":"Write"' [session-file].jsonl   # How many new files?
grep -c '"name":"Read"' [session-file].jsonl    # How many files read?
grep -c 'git commit' [session-file].jsonl       # How many commits?

# Extract unique file paths worked on
grep -o '"filePath":"[^"]*"' [session-file].jsonl | sort -u

# Find all error messages
grep -i '"error"\|"stderr"\|"failed"' [session-file].jsonl
```

### jq Tips and Tricks:

**Basic Syntax**:
- `jq '.'` - Pretty-print entire JSON
- `jq '.field'` - Extract a field
- `jq -r` - Raw output (no quotes)
- `jq 'select(condition)'` - Filter objects

**Useful Patterns**:
```bash
# Combine multiple filters
jq 'select(.type == "user" and .gitBranch == "main")' file.jsonl

# Check if field exists
jq 'select(.toolUseResult.filePath)' file.jsonl

# Access nested arrays
jq '.message.content[0].text' file.jsonl

# Optional chaining (won't error if field missing)
jq '.message.content[]?.name' file.jsonl

# Count matches
jq -s 'map(select(.type == "user")) | length' file.jsonl
```

### Common Issues and Solutions:

**Issue**: `jq: parse error`
- **Cause**: JSONL has one malformed line
- **Solution**: Use `grep` to find and skip malformed lines

**Issue**: `jq` too slow on large file
- **Solution**: Use `grep` for initial filtering, then pipe to `jq`
```bash
grep '"type":"user"' file.jsonl | jq '.message.content'
```

**Issue**: Complex nested structure
- **Solution**: Explore structure first with `head -1 file.jsonl | jq '.'`

**Issue**: Want to see array contents
- **Solution**: Use `[]?` for optional array access
```bash
jq '.message.content[]?.name' file.jsonl
```

### Efficiency Recommendations:

1. **Check file size first**: `ls -lh` to decide strategy
2. **Use grep for presence checks**: Faster than jq for "does X exist?"
3. **Use jq for extraction**: Better for structured data extraction
4. **Combine tools**: `grep | jq` for best of both worlds
5. **Sample strategically**: Beginning + end + errors usually tell the story
6. **Count before extracting**: Use `wc -l` or `grep -c` to understand volume
