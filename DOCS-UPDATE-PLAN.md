# Documentation Update Plan

**Date**: 2025-10-09
**Goal**: Simplify and remove outdated/confusing content based on lessons learned

## devops-as-apps-project Updates

### 1. README.md - MAJOR REWRITE
**Current**: 340 lines with extensive TraderX content
**Target**: ~200 lines focusing on the pattern

**Changes**:
- ✂️ Remove entire TraderX section (lines 78-126) - move to traderx repo
- ✂️ Remove multi-agent workflow section (lines 278-324) - move to traderx repo
- ✂️ Simplify "Live Examples" to just link to devops-examples and traderx
- ✅ Keep: Core concepts, prerequisites, getting started
- ✅ Keep: Links to other repos (devops-examples, devops-sdk)

### 2. PROPOSAL-INTEGRATING-EXAMPLES.md - DELETE
**Reason**: Draft from Jan 2025, not implemented, experimental
**Action**: Delete file entirely

### 3. CLAUDE.md - MINOR UPDATES
**Changes**:
- ✅ Update any remaining outdated paths (already done mostly)
- ✅ Verify all examples exist
- ✅ Remove references to deleted files

### 4. Other files - KEEP AS-IS
- CANONICAL-PATTERNS-SUMMARY.md - Good reference
- CONFIGHUB-ACTUAL-FEATURES.md - Good reference
- CONFIGHUB-DEPLOYMENT-PATTERN.md - Good reference

## traderx Updates

### 1. README.md - MAJOR SIMPLIFICATION
**Current**: 648 lines (too detailed)
**Target**: ~300 lines (high-level overview)

**Changes**:
- ✂️ Move detailed deployment steps to docs/DEPLOYMENT-GUIDE.md
- ✂️ Move troubleshooting to docs/TROUBLESHOOTING.md
- ✅ Keep: Quick start, architecture overview, key features
- ✅ Add: Link to migration-notes/ for implementation history

### 2. docs/ - REVIEW FOR REDUNDANCY
- ADVANCED-CONFIGHUB-PATTERNS.md (376 lines) - Review
- CHANWIT-LESSONS.md (384 lines) - Keep (historical)
- AUTOUPDATES-AND-GITOPS.md (472 lines) - Review
- LINKS-AND-DEPLOYMENT.md (559 lines) - Keep (reference)

### 3. Add Reference to migration-notes/
Update docs to point to migration-notes/ for implementation history

## Execution Order

1. devops-as-apps-project: Delete PROPOSAL
2. devops-as-apps-project: Rewrite README.md
3. devops-as-apps-project: Update CLAUDE.md references
4. traderx: Simplify README.md
5. traderx: Review docs/ for outdated content
6. Commit and push both repos

## Success Criteria

- ✅ devops-as-apps-project README < 250 lines
- ✅ traderx README < 350 lines
- ✅ No references to deleted files
- ✅ Clear separation: patterns in devops-as-apps, implementation in traderx
- ✅ All paths and links verified working
