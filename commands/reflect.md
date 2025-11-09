---
description: Analyze diary entries to identify patterns and propose CLAUDE.md updates
---

# Reflect on Diary Entries and Synthesize Insights

You are going to analyze multiple diary entries to identify recurring patterns, synthesize insights, and propose updates to the user's CLAUDE.md file.

## Parameters

The user can provide:
- **Date range**: "from YYYY-MM-DD to YYYY-MM-DD" or "last N days"
- **Entry count**: "last N entries" (e.g., "last 10 entries")
- **Project filter**: "for project [project-path]" (optional - filter to specific project)
- **Pattern filter**: "related to [keyword]" (e.g., "related to testing" or "related to React")

If no parameters are provided, default to analyzing the **last 10 diary entries**.

## Steps to Follow

1. **Check CLAUDE.md for processed entries**:
   - Read `~/.claude/CLAUDE.md` to find the `# Memory System - Processed Entries` section
   - Extract list of already-processed diary entry filenames
   - If section doesn't exist yet, all entries are unprocessed
   - Format in CLAUDE.md:
     ```markdown
     # Memory System - Processed Entries
     - 2025-11-07-session-1.md (reflected: 2025-11-08)
     - 2025-11-07-session-2.md (reflected: 2025-11-08)
     ```

2. **Locate diary entries**:
   - Directory: `~/.claude/memory/diary/`
   - Entries are named: `YYYY-MM-DD-session-N.md`
   - List all entries, sorted by date (newest first)
   - **Exclude already-processed entries** (unless user explicitly requests re-analysis)

3. **Filter entries based on parameters**:
   - If date range specified: only include entries within that range
   - If entry count specified: take the N most recent entries
   - If project filter specified: only include entries where the Project field matches
   - If pattern filter specified: only include entries that mention the keyword in any section

3. **Read and parse filtered diary entries**:
   - Read each diary entry file
   - Extract information from all sections
   - Pay special attention to:
     - User Preferences Observed
     - Code Patterns and Decisions
     - Solutions Applied (what works well)
     - Challenges Encountered (what to avoid)

4. **Create the reflections directory** (if it doesn't exist):
   - Directory: `~/.claude/memory/reflections/`
   - Use `mkdir -p` to create it automatically

5. **Analyze entries for patterns**:
   - **Frequency analysis**: What preferences/patterns appear in multiple entries?
   - **Consistency check**: Are preferences consistent or contradictory?
   - **Context awareness**: Do patterns apply globally or to specific project types?
   - **Abstraction level**: Can specific instances be generalized into rules?
   - **Signal vs. noise**: Distinguish between:
     - **One-off requests**: "Make this button pink" (appears once)
     - **Recurring patterns**: "Always use TypeScript strict mode" (appears 3+ times)

6. **Synthesize insights** organized by category:

   **CRITICAL**: Focus on extracting concise, actionable rules suitable for CLAUDE.md (which is read into every session).

   **A. PR Review Feedback Patterns** (from code reviews):
   - Common feedback themes from reviewers
   - Code quality issues flagged repeatedly
   - What reviewers appreciate vs. criticize
   - Patterns in "looks AI-generated" feedback

   **B. Persistent Preferences** (appear 2+ times):
   - Commit and PR style requirements
   - Code organization and structure
   - Testing and linting workflows
   - Tool and framework choices
   - Communication style

   **C. Design Decisions That Worked** (successful approaches):
   - Architecture choices that solved problems well
   - Technology selections with clear rationale
   - Patterns that led to clean, maintainable code
   - Decision-making frameworks that helped

   **D. Anti-Patterns to Avoid** (caused problems 2+ times):
   - Approaches that failed or needed rework
   - Common mistakes that waste time
   - What NOT to do and why
   - Alternatives that work better

   **E. Efficiency Lessons** (save time in future):
   - What workflows worked smoothly
   - What caused delays or friction
   - Tools/commands that proved useful
   - Processes to streamline

   **F. Project-Specific Patterns** (context-dependent):
   - Patterns specific to certain project types (React, CLI, Python, etc.)
   - Technology-specific preferences
   - Framework-specific conventions

7. **Generate a reflection document** with this structure:

```markdown
# Reflection: [Date Range or "Last N Entries"]

**Generated**: [YYYY-MM-DD HH:MM:SS]
**Entries Analyzed**: [count]
**Date Range**: [first-date] to [last-date]
**Projects**: [list of projects if filtered, or "All projects"]

## Summary
[2-3 paragraph overview of key insights discovered across these entries]

## Patterns Identified

### A. PR Review Feedback Patterns
[What reviewers commonly flag or appreciate]

1. **[Feedback Pattern]** (appeared in X/Y entries)
   - **Observation**: [What reviewers said/flagged]
   - **Examples**: [Specific feedback quotes]
   - **Lesson**: [What to do/avoid]
   - **CLAUDE.md rule**: `- [succinct actionable rule]`

### B. Persistent Preferences (2+ occurrences)
[Recurring user preferences across sessions]

1. **[Preference Name]** (appeared in X/Y entries)
   - **Observation**: [What was consistently preferred]
   - **Evidence**: [Which sessions, what happened]
   - **Confidence**: High/Medium/Low
   - **CLAUDE.md rule**: `- [succinct actionable rule]`

### C. Design Decisions That Worked
[Successful technical decisions and approaches]

1. **[Decision Name]**
   - **What worked**: [Brief description]
   - **Why it worked**: [Key reason]
   - **When to use**: [Context/applicability]
   - **CLAUDE.md rule** (if generalizable): `- [succinct rule]`

### D. Anti-Patterns to Avoid
[Things that failed or caused problems 2+ times]

1. **[Anti-pattern Name]** (appeared in X/Y entries)
   - **What didn't work**: [Brief description]
   - **Why it failed**: [Key reason]
   - **What to do instead**: [Alternative]
   - **CLAUDE.md rule**: `- [avoid X, use Y instead]`

### E. Efficiency Lessons
[Workflows, tools, and processes that save time]

1. **[Efficiency Pattern]**
   - **What worked well**: [Description]
   - **Time/effort saved**: [Impact]
   - **When to apply**: [Context]
   - **CLAUDE.md rule** (if applicable): `- [succinct rule]`

### F. Project-Specific Patterns
[Patterns that apply to specific project types or technologies]

1. **[Pattern Name]** (for [project type/technology])
   - **Observation**: [What was observed]
   - **Context**: [When this applies]
   - **CLAUDE.md rule**: `- [context]: [action]`

## Notable Mistakes and Learnings
[Key mistakes that taught valuable lessons]
- **Mistake**: [What went wrong]
  - **Why**: [Root cause]
  - **Learning**: [What was learned]
  - **Prevention**: [How to avoid in future]

## One-Off Observations
[Preferences that appeared only once - not patterns yet, but worth noting]
- [Observation from single session]

## Proposed CLAUDE.md Updates

**CRITICAL FORMAT REQUIREMENTS**:
- CLAUDE.md is read into EVERY session, so keep updates **succinct and non-verbose**
- Use bullet points (-, +, or numbered lists)
- Be direct and actionable (imperative tone)
- Avoid explanations - just state the rule
- Group related rules together
- Use context markers when needed (e.g., "for Python projects:", "when reviewing PRs:")

**Good Example** (succinct, actionable):
```markdown
- git commits: use conventional format (feat:, fix:, refactor:, docs:, test:)
- PR descriptions: no Claude Code attribution or AI tool mentions
- testing: always run tests before committing, ensure they pass
```

**Bad Example** (too verbose):
```markdown
- When you are creating git commits, it's important to follow the conventional
  commit format which includes prefixes like feat: for features, fix: for bug
  fixes, refactor: for code refactoring, docs: for documentation changes, and
  test: for test changes. This helps maintain consistency across the codebase.
```

Below are proposed additions to your `~/.claude/CLAUDE.md` file. **Review these carefully before adding them.**

### Section: [General Preferences / Project-Specific / Code Quality / Git Workflow / etc.]

```markdown
[Proposed succinct bullet points for CLAUDE.md]
- [Actionable rule 1]
- [Actionable rule 2]
- [Context-specific rule]: [when X, do Y]
```

## Metadata
- **Diary entries analyzed**: [list of filenames]
- **Total user messages**: [count across all entries]
- **Total actions taken**: [count across all entries]
- **Challenges documented**: [count]
- **Projects covered**: [list of unique projects]
```

8. **Save the reflection document**:
   - Filename format: `YYYY-MM-reflection-N.md` (increment N if multiple reflections in same month)
   - Save to: `~/.claude/memory/reflections/[filename]`

9. **Update CLAUDE.md processed entries tracking**:
   - Add or update the `# Memory System - Processed Entries` section in `~/.claude/CLAUDE.md`
   - List all diary entries just analyzed with reflection date
   - Format: `- [diary-filename] (reflected: YYYY-MM-DD)`
   - If section doesn't exist, add it at the bottom of CLAUDE.md
   - If section exists, append new entries
   - **IMPORTANT**: Only update this tracking section, do NOT add the proposed rules yet
   - The user will review and approve rules separately in next step

10. **Present the reflection to the user**:
   - Display the proposed CLAUDE.md updates prominently
   - Ask the user to review and confirm which updates they want to apply
   - Offer to help apply approved updates to their CLAUDE.md file
   - Explain that user review is critical to maintain quality

## Important Guidelines

### Pattern Recognition Principles

1. **Frequency matters**: Require 2+ occurrences before calling something a "pattern"
   - **Strong patterns**: 3+ occurrences with consistency
   - **Emerging patterns**: 2 occurrences worth noting
   - **One-off**: Single occurrence, document but don't add to CLAUDE.md yet

2. **Context matters**: Note whether patterns are:
   - Universal (apply everywhere)
   - Project-specific (only for certain types of projects)
   - Tool-specific (only when using certain technologies)

3. **Consistency matters**: Flag contradictory preferences for user review

4. **Actionability matters**: Only propose rules that Claude can actually follow

5. **Abstraction matters**: Find the right level:
   - Too specific: "The login button should be blue" ❌
   - Too broad: "Users like colors" ❌
   - Just right: "Use the design system's primary color for CTAs" ✅

6. **Succinctness matters for CLAUDE.md**:
   - Each rule should be ONE line (or short bullet)
   - Use imperative tone: "do X", "use Y", "avoid Z"
   - Add context prefix when needed: "for Python:", "when testing:"
   - NO explanations or rationale in CLAUDE.md - just the rule

### Distinguishing Signal from Noise

**SIGNAL** (add to CLAUDE.md):
- "Always use TypeScript strict mode" (appears in 5 sessions across 3 projects)
- "Prefer functional components in React" (appears in 4 React projects)
- "Run tests before committing" (appears in 6 sessions)

**NOISE** (document but don't add to CLAUDE.md):
- "Make this button pink" (appears once, specific task)
- "Use dark mode for this demo" (appears once, context-specific)
- "Skip tests this time" (contradicted by usual pattern)

### Quality Checks

Before proposing a CLAUDE.md update, verify:
- ✅ Does this apply to future sessions? (not just the past)
- ✅ Is this actionable? (Claude can actually do it)
- ✅ Is this generalizable? (not too specific to one case)
- ✅ Is this consistent? (doesn't contradict other patterns)
- ✅ Is this valuable? (will it improve future interactions)

## Error Handling

- If no diary entries exist, inform the user and suggest running `/diary` first
- If all diary entries have been processed and no new entries are found, inform the user
- If fewer than 3 entries are found, proceed but note that pattern confidence is low
- If diary entries are malformed, skip them and document which ones had issues
- If the reflections directory cannot be created, report the error
- If CLAUDE.md cannot be read or written, report the error but continue with reflection

## Handling Already-Processed Entries

**Default behavior**: Skip entries already listed in `# Memory System - Processed Entries`

**User can override** with these flags:
- "include all entries" - re-analyze everything including processed entries
- "reprocess [filename]" - re-analyze specific entry
- "last N entries including processed" - analyze N most recent, even if processed

**When to suggest re-processing**:
- User significantly changed their workflow
- User wants to validate previous patterns
- User wants to extract different insights from same sessions

## Example Usage

```
# Analyze last 10 unprocessed entries (default)
/reflect

# Analyze last 20 unprocessed entries
/reflect last 20 entries

# Analyze entries from a date range
/reflect from 2025-01-01 to 2025-01-31

# Analyze entries for specific project
/reflect for project /Users/rlm/Desktop/Code/my-app

# Analyze entries related to testing
/reflect related to testing

# Combine filters
/reflect last 15 entries for project /Users/rlm/Desktop/Code/my-app related to React

# Re-analyze including already-processed entries
/reflect include all entries

# Re-analyze specific entry
/reflect reprocess 2025-11-07-session-1.md
```
