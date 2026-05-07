---
name: "sync-to-jira"
description: "Batch sync all markdown stories to Jira with parent-child relationships"
---

# Sync Stories to Jira

## Objective

Sync all markdown story files in `specs/stories/` to Jira project **MCPW** (or specified project).

## Instructions

### 1. Discover Stories

- Find all `.md` files in `specs/stories/` folder
- Read frontmatter to determine sync status

### 2. Analyze Sync State

For each story file:

**If NO `sync.jira` in frontmatter:**
- Action: **CREATE** new Jira issue
- Type: Use `type` field from frontmatter (`story`, `epic`, `task`, `bug`)
- Summary: Use `title` field

**If HAS `sync.jira.issue_key`:**
- Action: **UPDATE** existing Jira issue
- Issue: Use `sync.jira.issue_key` value
- Check: Verify issue still exists in Jira

### 3. Handle Parent-Child Relationships

**If story references an epic:**
- Check frontmatter for `epic` field (e.g., `epic: "EPIC-001"`)
- Look up epic's Jira issue key from `specs/epics/EPIC-001.md` frontmatter
- Set parent link when creating/updating story

### 4. Preview Before Execution

**Show me:**
- List of files to sync
- Action for each (CREATE or UPDATE)
- Parent-child relationships detected
- Total count: X creates, Y updates

**Wait for my confirmation** before proceeding.

### 5. Execute Sync

For each file:
1. Sync to Jira using `@atlassian` MCP
2. Update frontmatter with sync info
3. Report success or failure

### 6. Summary Report

After all syncs complete, show:
- ✅ Successfully synced: [count]
- ❌ Failed: [count] (with reasons)
- 📝 Files updated with sync info: [list]

## Example Frontmatter

**Before sync (no sync info):**

```yaml
---
title: "User Login Form"
type: story
status: draft
epic: "EPIC-001"
---
```

**After sync (CREATE):**
```yaml
---
title: "User Login Form"
type: story
status: draft
epic: "EPIC-001"
sync:
  jira:
    url: "https://yourname-workshop.atlassian.net/browse/MCPW-1"
    issue_key: "MCPW-1"
    synced_at: "2025-12-03T15:00:00Z"
---
```

## Edge Cases

### Epic Not Yet Synced
- If story references epic but epic has no `sync.jira.issue_key`
- Action: Sync epic first, then sync story

### Story Updated in Markdown Only
- If `synced_at` is older than file modification time
- Action: UPDATE existing Jira issue

### Jira Issue Deleted
- If `sync.jira.issue_key` exists but issue not found in Jira
- Action: Warn user, ask if should CREATE new issue or skip

### Conflicting Changes
- If both markdown and Jira were updated since last sync
- Action: Show diff, ask which version to keep

## Configuration

**Default Project:** MCPW
**Story Folder:** specs/stories/
**Epic Folder:** specs/epics/

To change project, specify: "Sync to Jira project [PROJECT_KEY]"
