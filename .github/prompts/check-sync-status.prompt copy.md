---
name: "check-sync-status"
description: "Detect out-of-date files and sync conflicts"
---

# Check Sync Status

## Objective

Analyze all markdown files with sync frontmatter and detect:
- Files modified since last sync (out-of-date)
- Files synced but remote deleted (orphaned)
- Files never synced (new)

## Instructions

### 1. Discover All Spec Files

**Scan folders:**
- `specs/stories/` - Jira stories
- `specs/epics/` - Jira epics
- `specs/prds/` - Confluence PRDs
- `specs/adrs/` - Confluence ADRs (if exists)

### 2. Categorize Each File

**Category A: Never Synced (NEW)**
- Condition: No `sync` section in frontmatter
- Status: 🆕 NEW
- Action Needed: Sync to Jira/Confluence

**Category B: Out-of-Date (MODIFIED)**
- Condition: File modified after `synced_at` timestamp
- Status: ⚠️ OUT-OF-DATE
- Action Needed: Update remote (Jira/Confluence)

**Category C: Synced and Current**
- Condition: File not modified since `synced_at`
- Status: ✅ UP-TO-DATE
- Action Needed: None

**Category D: Orphaned (REMOTE DELETED)**
- Condition: Has `sync` info but remote issue/page deleted
- Status: ❌ ORPHANED
- Action Needed: Remove sync info or recreate remote

### 3. Check Remote Status

For files with sync info:

**Jira Issues:**
- Use `sync.jira.issue_key` to check if issue exists
- API: `@atlassian get issue [issue_key]`

**Confluence Pages:**
- Use `sync.confluence.page_id` to check if page exists
- API: `@atlassian get page [page_id]`

### 4. Detect Conflicts (Optional Deep Check)

**If remote was updated after local `synced_at`:**
- Status: ⚠️ CONFLICT
- Action Needed: Manual merge or choose version

### 5. Generate Report

**Summary Table:**

```
Sync Status Report
Generated: 2025-12-03T15:20:00Z

NEW (never synced):
- specs/stories/STORY-002.md
- specs/prds/PRD-002.md
Total: 2

OUT-OF-DATE (local changes not synced):
- specs/stories/STORY-001.md (modified 2 hours ago, last synced 1 day ago)
Total: 1

UP-TO-DATE:
- specs/epics/EPIC-001.md
- specs/prds/PRD-001.md
Total: 2

ORPHANED (remote deleted):
(none)
Total: 0

CONFLICTS (both local and remote updated):
(none - requires deep check)
Total: 0
```

**Recommended Actions:**
1. Sync 2 NEW files: Run `sync-to-jira.prompt.md` or `sync-to-confluence.prompt.md`
2. Update 1 OUT-OF-DATE file: Re-run sync prompt
3. No orphaned files detected

### 6. Action Prompts

**If NEW files found:**
```
To sync new files, run:
- For Jira: Use sync-to-jira.prompt.md
- For Confluence: Use sync-to-confluence.prompt.md
```

**If OUT-OF-DATE files found:**
```
To update out-of-date files, run:
- Re-run appropriate sync prompt (will auto-detect UPDATE action)
```

**If ORPHANED files found:**
```
To fix orphaned files:
1. Remove sync section from frontmatter, or
2. Re-sync to create new remote issue/page
```

## Edge Cases

### File Modified But Content Unchanged
- Check actual content diff, not just timestamp
- If only whitespace/formatting changed, mark as UP-TO-DATE

### Multiple Sync Targets
- If file synced to both Jira AND Confluence
- Check status for each target separately

### Permission Issues
- If can't access remote due to permissions
- Mark as UNKNOWN status, suggest permission check

## Configuration

**Folders to Scan:**
- specs/stories/ (Jira)
- specs/epics/ (Jira)
- specs/prds/ (Confluence)
- specs/adrs/ (Confluence, optional)

**Deep Conflict Check:** Optional (add `--deep` flag)