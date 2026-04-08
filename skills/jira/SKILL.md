---
name: jira
description: Operate Atlassian Jira via command line using jira. Use when the user wants to list, create, edit, view, assign, move, delete, clone, or comment on Jira issues; manage epics, sprints, or releases; search issues with JQL; or configure Jira connection. Triggers on any mention of "jira", "Jira", "issue", "ticket", "sprint", "epic", "JQL", or tasks like "list my issues", "create a ticket", "assign issue", "move to done", "show sprint issues", "create epic", or Jira-related workflows. Also use when user mentions "jira" or wants CLI-based Jira operations.
---

# jira Skill

A skill for operating Atlassian Jira via the `jira` CLI tool.

## Prerequisites

1. **Install jira** (if not already installed):
   ```bash
   # Download from https://github.com/ankitpokhrel/jira/releases
   curl -L "https://github.com/ankitpokhrel/jira/releases/download/v1.7.0/jira_1.7.0_linux_x86_64.tar.gz" -o /tmp/jira.tar.gz
   tar -xzf /tmp/jira.tar.gz -C /tmp/
   sudo cp /tmp/jira_1.7.0_linux_x86_64/bin/jira /usr/local/bin/
   ```

2. **Configure jira** (if not already configured):
   - Get API token: https://id.atlassian.com/manage-profile/security/api-tokens
   - Export: `export JIRA_API_TOKEN="your-token-here"`
   - Run: `jira init` and follow prompts

3. **Verify installation**:
   ```bash
   jira version
   jira me  # should show your user
   ```

## Usage Patterns

### Global Flags
- `-c, --config string` - Config file path
- `-p, --project string` - Jira project key (e.g., PROJ, MYPROJECT)
- `--plain` - Output in plain text format (useful for scripting)
- `--debug` - Turn on debug output

### Quick Reference

| Task | Command |
|------|---------|
| List my issues | `jira issue list -a$(jira me)` |
| List issues in project | `jira issue list -pPROJECT` |
| Create issue | `jira issue create` |
| View issue | `jira issue view ISSUE-123` |
| Edit issue | `jira issue edit ISSUE-123` |
| Assign issue | `jira issue assign ISSUE-123 "User Name"` |
| Transition issue | `jira issue move ISSUE-123 "Done"` |
| Add comment | `jira issue comment add ISSUE-123 "comment text"` |
| Clone issue | `jira issue clone ISSUE-123` |
| Delete issue | `jira issue delete ISSUE-123` |
| List epics | `jira epic list` |
| List sprints | `jira sprint list` |
| List projects | `jira project list` |

## Issue Operations

### List Issues
```bash
# List recent issues
jira issue list

# List issues assigned to me
jira issue list -a$(jira me)

# List issues by status
jira issue list -s"To Do"
jira issue list -s"In Progress"
jira issue list -s"Done"

# List issues by priority
jira issue list -yHigh
jira issue list -yMedium
jira issue list -yLow

# List issues by type
jira issue list -tBug
jira issue list -tTask
jira issue list -tStory

# List issues created in last N days
jira issue list --created -7d

# List issues with labels
jira issue list -lbackend

# Use JQL directly
jira issue list -q "summary ~ bug"

# Output in plain/JSON/CSV format
jira issue list --plain
jira issue list --raw
jira issue list --csv
```

### Create Issue
```bash
# Interactive create
jira issue create

# Create with flags (--no-input skips prompts)
jira issue create -tBug -s"New Bug" -yHigh -lbug -b"Bug description" --no-input

# Create issue attached to epic
jira issue create -tStory -s"Story summary" -PEPIC-123
```

### View Issue
```bash
# View issue details
jira issue view ISSUE-123

# View with N recent comments
jira issue view ISSUE-123 --comments 5
```

### Edit Issue
```bash
# Edit issue (interactive)
jira issue edit ISSUE-123

# Edit with flags
jira issue edit ISSUE-123 -s"New summary" -yHigh -lbug -b"New description" --no-input

# Remove label/component (use minus prefix)
jira issue edit ISSUE-123 --label -p2 --component -FE
```

### Assign Issue
```bash
# Assign (interactive)
jira issue assign ISSUE-123

# Assign to user
jira issue assign ISSUE-123 "John Doe"

# Assign to self
jira issue assign ISSUE-123 $(jira me)

# Unassign
jira issue assign ISSUE-123 x
```

### Move/Transition Issue
```bash
# Transition (interactive)
jira issue move ISSUE-123

# Transition to specific status
jira issue move ISSUE-123 "In Progress"
jira issue move ISSUE-123 Done

# Transition with comment
jira issue move ISSUE-123 "Done" --comment "Work completed"

# Transition with resolution
jira issue move ISSUE-123 Done -RFixed
```

### Add Comment
```bash
# Add comment (interactive)
jira issue comment add ISSUE-123

# Add comment with text
jira issue comment add ISSUE-123 "Fixed the issue"

# Add internal comment
jira issue comment add ISSUE-123 "Internal note" --internal

# Use template
jira issue comment add ISSUE-123 --template /path/to/template.tmpl
```

### Worklog
```bash
# Add worklog (interactive)
jira issue worklog add ISSUE-123

# Add worklog with time
jira issue worklog add ISSUE-123 "2d 3h 30m" --no-input

# Add worklog with comment
jira issue worklog add ISSUE-123 "1h" --comment "Working on fix" --no-input
```

### Clone Issue
```bash
# Clone issue (interactive)
jira issue clone ISSUE-123

# Clone with modifications
jira issue clone ISSUE-123 -s"Cloned summary" -yHigh -a$(jira me)

# Clone with text replacement
jira issue clone ISSUE-123 -H"old-text:new-text"
```

### Delete Issue
```bash
# Delete (interactive, with confirmation)
jira issue delete ISSUE-123

# Delete with cascade (deletes subtasks too)
jira issue delete ISSUE-123 --cascade
```

### Link Issues
```bash
# Link two issues
jira issue link ISSUE-123 ISSUE-456 Blocks

# Add remote link
jira issue link remote ISSUE-123 https://example.com "Example"

# Unlink issues
jira issue unlink ISSUE-123 ISSUE-456
```

## Epic Operations

```bash
# List epics
jira epic list

# List epics in table view
jira epic list --table

# List issues in an epic
jira epic list EPIC-123

# Create epic
jira epic create -n"Epic Name" -s"To Do" -yHigh -b"Description"

# Add issues to epic
jira epic add EPIC-123 ISSUE-1 ISSUE-2

# Remove issues from epic
jira epic remove EPIC-123 ISSUE-1
```

## Sprint Operations

```bash
# List sprints
jira sprint list

# List sprints in table
jira sprint list --table

# List current sprint issues
jira sprint list --current

# List previous sprint
jira sprint list --prev

# List issues in sprint
jira sprint list SPRINT_ID

# Add issues to sprint
jira sprint add SPRINT_ID ISSUE-1 ISSUE-2
```

## Release Operations

```bash
# List releases
jira release list

# List releases for specific project
jira release list --project PROJECT_KEY
```

## Project & Board Operations

```bash
# List projects
jira project list

# List boards in project
jira board list -pPROJECT

# Open in browser
jira open
jira open ISSUE-123
```

## Server Info

```bash
# Get Jira server information
jira serverinfo
```

## Common Workflows

### Find My High Priority Open Issues
```bash
jira issue list -a$(jira me) -yHigh -s~Done
```

### Find Issues Created This Week
```bash
jira issue list -r$(jira me) --created week
```

### Move Issue to Done with Comment
```bash
jira issue move ISSUE-123 Done --comment "Completed as per ticket"
```

### Create Bug with All Details
```bash
jira issue create -tBug -s"Bug Summary" -yHigh -lbug -lurgent -b"Steps to reproduce..." --no-input
```

### Assign Multiple Issues to Me
```bash
# List unassigned and assign
jira issue list -ax | while read line; do jira issue assign "$line" $(jira me); done
```

## Output Formats

- **Interactive UI** (default) - Navigate with arrow keys, j/k/h/l, g/G
- **Plain** (`--plain`) - Simple text table
- **JSON** (`--raw`) - Raw JSON output
- **CSV** (`--csv`) - CSV format

### Interactive UI Controls
- Arrow keys or `j, k, h, l` - Navigate
- `g` / `G` - Go to top/bottom
- `v` - View issue details
- `m` - Transition issue
- `CTRL+r` / `F5` - Refresh
- `ENTER` - Open in browser
- `c` - Copy URL to clipboard (Linux: requires xclip/xsel)
- `CTRL+k` - Copy issue key
- `?` - Help
- `q` / `ESC` / `CTRL+c` - Quit

## Configuration

- Default config: `~/.config/.jira/.config.yml`
- Override with: `JIRA_CONFIG_FILE` env var or `-c` flag
- API token: Set via `JIRA_API_TOKEN` environment variable
- Auth type: Set via `JIRA_AUTH_TYPE` (basic, bearer, or mtls)