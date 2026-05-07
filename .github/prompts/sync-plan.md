---
name: "sync-plan"
description: "Two-way sync plan with conflict detection and human approval"
---

# Generate Sync Plan (Two-Way Sync)

## Objective

Create a detailed sync plan that:
1. Detects changes in both local markdown AND remote (Jira/Confluence)
2. Identifies conflicts requiring human decision
3. Proposes merge strategy
4. Waits for approval before execution

## Instructions

### 1. Discover All Files with Sync Info

**Find files with:**
- `sync.jira` section (Jira-synced files)
- `sync.confluence` section (Confluence-synced files)

### 2. For Each File, Check Both Directions

**Local State:**
- Read file content
- Check last modified timestamp
- Compare to `synced_at` in frontmatter

**Remote State:**
- Fetch current content from Jira/Confluence
- Check remote last updated timestamp
- Compare to `synced_at` in frontmatter

### 3. Determine Sync Direction

**Scenario A: Local Changed, Remote Unchanged**
- Direction: Local → Remote (UPDATE)
- Conflict: No
- Action: Push local changes to remote

**Scenario B: Remote Changed, Local Unchanged**
- Direction: Remote → Local (PULL)
- Conflict: No
- Action: Pull remote changes to local

**Scenario C: Both Changed**
- Direction: Both (CONFLICT)
- Conflict: YES
- Action: Show diff, ask user to resolve

**Scenario D: Neither Changed**
- Direction: None (IN SYNC)
- Conflict: No
- Action: Skip

### 4. Generate Sync Plan

**For each file, show:**

```markdown
## File: specs/stories/STORY-001.md

**Sync Target:** Jira issue MCPW-1
**Last Synced:** 2025-12-01T10:00:00Z

**Local Status:**
- Modified: 2025-12-03T14:00:00Z (2 days after sync)
- Changes:
  * Added AC4: Remember Me Functionality
  * Updated technical notes

**Remote Status:**
- Modified: 2025-12-02T12:00:00Z (1 day after sync)
- Changes:
  * Status changed: To Do → In Progress
  * Assignee set: John Doe

**Conflict Detected:** YES (both changed)

**Proposed Action:**
1. Pull remote status field → local frontmatter (`status: in-progress`)
2. Pull remote assignee → local frontmatter (`assignee: "John Doe"`)
3. Push local AC4 addition → remote description
4. Push local technical notes → remote description

**User Decision Required:**
- Accept proposed merge? (yes/no)
- Or: Choose version (local/remote)
- Or: Skip this file
```

### 5. Handle Conflicts

**For conflicting files:**

**Option 1: Three-way merge**
- Identify common ancestor (last synced version)
- Apply non-conflicting changes from both sides
- Flag truly conflicting sections for user

**Option 2: Manual choice**
- Show local version
- Show remote version
- Ask user: "Which version do you want to keep?"

**Option 3: Skip**
- Leave file unchanged
- Mark as requiring manual resolution

### 6. Preview Full Plan

**Summary:**
```
Sync Plan Summary
─────────────────────────────────────────────
Files Analyzed: 5

IN SYNC (no action needed):
- specs/epics/EPIC-001.md
Total: 1

LOCAL → REMOTE (push changes):
- specs/stories/STORY-002.md
  Action: Push new AC5 to Jira
Total: 1

REMOTE → LOCAL (pull changes):
- specs/prds/PRD-001.md
  Action: Pull status update from Confluence
Total: 1

CONFLICTS (both changed):
- specs/stories/STORY-001.md
  Action: Merge (see details above)
Total: 1

SKIPPED (manual resolution required):
- specs/stories/STORY-003.md
  Reason: Complex merge conflict
Total: 1
─────────────────────────────────────────────

Next Steps:
1. Review conflict resolutions above
2. Confirm plan (type 'approve' to proceed)
3. I'll execute the sync actions
```

### 7. Wait for Approval

**Ask user:**
```
Do you approve this sync plan?
- Type 'approve' to execute
- Type 'skip [filename]' to exclude specific files
- Type 'cancel' to abort
```

### 8. Execute Approved Plan

**For each approved action:**
1. Execute sync (push/pull/merge)
2. Update frontmatter with new `synced_at`
3. Report success or failure

### 9. Final Report

```
Sync Plan Execution Complete
─────────────────────────────────────────────
✅ Successfully synced: 3 files
❌ Failed: 0 files
⏭️ Skipped: 1 file (STORY-003.md - manual resolution)

Updated Files:
- specs/stories/STORY-001.md (merged)
- specs/stories/STORY-002.md (pushed)
- specs/prds/PRD-001.md (pulled)

Skipped Files (require manual review):
- specs/stories/STORY-003.md
  Reason: Complex conflict in acceptance criteria
  Next: Manually merge and re-sync
─────────────────────────────────────────────
```

## Edge Cases

### Remote Deleted Since Last Sync
- If `sync.jira.issue_key` exists but issue deleted
- Action: Warn user, ask if should remove sync info or recreate

### Local Deleted Since Last Sync
- If file exists in Jira/Confluence but not locally
- Action: Ask if should pull back or delete remote

### Frontmatter-Only Changes
- If only `status`, `assignee` fields changed (not description)
- Action: Auto-merge (low risk)

### Description Conflicts
- If both local and remote description changed
- Action: Show diff, require user decision

## Configuration

**Conflict Strategy:** Three-way merge (default)
**Auto-Merge:** Frontmatter fields only
**Require Approval:** Always (safety gate)
