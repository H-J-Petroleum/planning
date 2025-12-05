# Label System Standard

> 30-label taxonomy for AI-first project management, based on Kubernetes and "Sane GitHub Labels" standards.

## Overview

This label system provides a standardized way to categorize and filter issues across all H&J Petroleum repositories. It's optimized for:

- ✅ **AI agent filtering** (find work suitable for automation)
- ✅ **Multi-repo coordination** (track scope across 11 repos)
- ✅ **Priority management** (clear what to work on first)
- ✅ **Effort estimation** (Fibonacci sizing for planning)

## Label Categories

### Priority Labels (4 labels)

Indicate urgency and importance.

```yaml
priority/critical
  badge: ![Priority: Critical](https://img.shields.io/badge/priority%2Fcritical-d93f0b)
  color: "d93f0b"  # Red
  description: "Blocking production or security issue. Address immediately."

priority/high
  badge: ![Priority: High](https://img.shields.io/badge/priority%2Fhigh-ff9800)
  color: "ff9800"  # Orange
  description: "Important but not blocking. Address this cycle."

priority/medium
  badge: ![Priority: Medium](https://img.shields.io/badge/priority%2Fmedium-ffc107)
  color: "ffc107"  # Yellow
  description: "Normal priority. Address in next 2-4 weeks."

priority/low
  badge: ![Priority: Low](https://img.shields.io/badge/priority%2Flow-8bc34a)
  color: "8bc34a"  # Green
  description: "Nice to have. Address when time permits."
```

**Usage:**

- Every issue gets ONE priority label
- Default: `priority/medium`
- Change as priorities shift

### Type Labels (6 labels)

Categorize the kind of work.

```yaml
type/bug
  badge: ![Type: Bug](https://img.shields.io/badge/type%2Fbug-d73a4a)
  color: "d73a4a"  # Red
  description: "Something isn't working correctly"

type/feature
  badge: ![Type: Feature](https://img.shields.io/badge/type%2Ffeature-0e8a16)
  color: "0e8a16"  # Green
  description: "New feature or enhancement request"

type/documentation
  badge: ![Type: Docs](https://img.shields.io/badge/type%2Fdocumentation-0075ca)
  color: "0075ca"  # Blue
  description: "Improvements or additions to documentation"

type/security
  badge: ![Type: Security](https://img.shields.io/badge/type%2Fsecurity-d93f0b)
  color: "d93f0b"  # Dark Red
  description: "Security vulnerability or security-related improvement"

type/automation
  badge: ![Type: Automation](https://img.shields.io/badge/type%2Fautomation-1d76db)
  color: "1d76db"  # Steel Blue
  description: "CI/CD, workflows, or automation improvements"

type/cleanup
  badge: ![Type: Cleanup](https://img.shields.io/badge/type%2Fcleanup-d4c5f9)
  color: "d4c5f9"  # Light Purple
  description: "Code cleanup, refactoring, or technical debt"
```

**Usage:**

- Every issue gets ONE type label
- Choose the primary type if multiple apply

### Scope Labels (4 labels)

**UNIQUE TO H&J PETROLEUM** - Indicates how many repos are affected.

```yaml
scope/all-repos
  badge: ![Scope: All Repos](https://img.shields.io/badge/scope%2Fall--repos-5319e7)
  color: "5319e7"  # Purple
  description: "Affects all 11 repositories"

scope/multi-repo
  badge: ![Scope: Multi Repo](https://img.shields.io/badge/scope%2Fmulti--repo-7057ff)
  color: "7057ff"  # Light Purple
  description: "Affects 2-5 repositories"

scope/single-repo
  badge: ![Scope: Single Repo](https://img.shields.io/badge/scope%2Fsingle--repo-0075ca)
  color: "0075ca"  # Blue
  description: "Affects only one repository"

scope/org-level
  badge: ![Scope: Org Level](https://img.shields.io/badge/scope%2Forg--level-fbca04)
  color: "fbca04"  # Gold
  description: "Organization-level change (settings, policies)"
```

**Usage:**

- Essential for multi-repo initiatives
- Helps prioritize organization-wide work
- Use with filters to see cross-repo impact

### Effort Labels (6 labels)

Fibonacci-based effort estimation.

```yaml
effort/1
  badge: ![Effort: 1](https://img.shields.io/badge/effort%2F1-c2e0c6)
  color: "c2e0c6"  # Very Light Green
  description: "< 30 minutes (quick win)"

effort/2
  badge: ![Effort: 2](https://img.shields.io/badge/effort%2F2-bfe5bf)
  color: "bfe5bf"  # Light Green
  description: "~1 hour"

effort/3
  badge: ![Effort: 3](https://img.shields.io/badge/effort%2F3-a3d9a5)
  color: "a3d9a5"  # Medium Green
  description: "2-3 hours (half day)"

effort/5
  badge: ![Effort: 5](https://img.shields.io/badge/effort%2F5-7bc96f)
  color: "7bc96f"  # Green
  description: "Half day to full day"

effort/8
  badge: ![Effort: 8](https://img.shields.io/badge/effort%2F8-51af32)
  color: "51af32"  # Dark Green
  description: "2-3 days"

effort/13
  badge: ![Effort: 13](https://img.shields.io/badge/effort%2F13-2a7515)
  color: "2a7515"  # Very Dark Green
  description: "1 week+ (epic)"
```

**Usage:**

- Use during shaping phase
- Don't overthink - rough estimate is fine
- If unsure, estimate high
- Fibonacci enforces "t-shirt sizing" thinking

### Status Labels (5 labels)

Track state of work (optional, Projects handle most of this).

```yaml
status/ready
  badge: ![Status: Ready](https://img.shields.io/badge/status%2Fready-0e8a16)
  color: "0e8a16"  # Green
  description: "Ready to be worked on, all info available"

status/in-progress
  badge: ![Status: In Progress](https://img.shields.io/badge/status%2Fin--progress-1d76db)
  color: "1d76db"  # Blue
  description: "Actively being worked on"

status/blocked
  badge: ![Status: Blocked](https://img.shields.io/badge/status%2Fblocked-d93f0b)
  color: "d93f0b"  # Red
  description: "Blocked by external factor or dependency"

status/in-review
  badge: ![Status: In Review](https://img.shields.io/badge/status%2Fin--review-0052cc)
  color: "0052cc"  # Dark Blue
  description: "Under review, awaiting feedback"

status/on-hold
  badge: ![Status: On Hold](https://img.shields.io/badge/status%2Fon--hold-ff9800)
  color: "ff9800"  # Orange
  description: "Work paused, may resume later"
```

**Usage:**

- Optional if using GitHub Projects (Projects show status)
- Useful for repos without Projects enabled
- Apply one status label maximum

### AI Labels (5 labels)

**CRITICAL FOR AI TEAMS** - Guide AI agents to appropriate work.

```yaml
ai/ready
  badge: ![AI: Ready](https://img.shields.io/badge/ai%2Fready-7057ff)
  color: "7057ff"  # Purple
  description: "Fully specified, AI can start immediately"

ai/needs-human
  badge: ![AI: Needs Human](https://img.shields.io/badge/ai%2Fneeds--human-ff9800)
  color: "ff9800"  # Orange
  description: "Human decision or review required"

ai/in-progress
  badge: ![AI: In Progress](https://img.shields.io/badge/ai%2Fin--progress-9c27b0)
  color: "9c27b0"  # Purple
  description: "AI agent currently working on this"

ai/pair
  badge: ![AI: Pair](https://img.shields.io/badge/ai%2Fpair-d1bcf5)
  color: "d1bcf5"  # Light Purple
  description: "Human+AI pair programming needed"

ai/blocked
  badge: ![AI: Blocked](https://img.shields.io/badge/ai%2Fblocked-d93f0b)
  color: "d93f0b"  # Red
  description: "AI stuck, needs human help"
```

**Usage:**

- Add `ai/ready` when issue has complete context
- Use `ai/pair` for complex work needing collaboration
- `ai/blocked` triggers human review within 24h

## Complete Label Set

**Total: 30 labels**

| Category | Count | Purpose             |
| -------- | ----- | ------------------- |
| Priority | 4     | Urgency/importance  |
| Type     | 6     | Work category       |
| Scope    | 4     | Multi-repo tracking |
| Effort   | 6     | Size estimation     |
| Status   | 5     | Work state          |
| AI       | 5     | AI agent guidance   |

## Labeling Guidelines

### Every Issue Should Have

**Required (4 labels minimum):**

1. ONE `priority/*` label
2. ONE `type/*` label
3. ONE `scope/*` label
4. ONE `effort/*` label

**Optional:**

- ZERO or ONE `status/*` label
- ZERO or MORE `ai/*` labels

### Example Combinations

**Example 1: Quick Win**

```
priority/high
type/automation
scope/all-repos
effort/1
status/ready
ai/ready
```

**Example 2: Complex Feature**

```
priority/medium
type/feature
scope/single-repo
effort/8
status/in-progress
ai/pair
```

**Example 3: Urgent Security Fix**

```
priority/critical
type/security
scope/multi-repo
effort/3
status/blocked
ai/needs-human
```

## Installation

### Manual Setup

```bash
# Create each label
gh label create "priority/critical" \
  --repo H-J-Petroleum/hjps-toolkit \
  --color "d93f0b" \
  --description "Blocking production or security issue. Address immediately."

# Repeat for all 30 labels...
```

### Automated Setup (Recommended)

```bash
# Use provided script
cd toolkit-standards/scripts
./sync-labels.sh H-J-Petroleum/hjps-toolkit

# Sync to all repos
./sync-labels-all-repos.sh
```

### Using labels.yml

```bash
# Install github-label-sync
npm install -g github-label-sync

# Sync from YAML file
github-label-sync \
  --access-token $GITHUB_TOKEN \
  --labels toolkit-standards/templates/.github/labels.yml \
  H-J-Petroleum/hjps-toolkit
```

## Using Labels

### In GitHub Web UI

1. Open issue
2. Click "Labels" in right sidebar
3. Search for label (e.g., type "priority/")
4. Select appropriate labels
5. Save

### With GitHub CLI

```bash
# Add single label
gh issue edit 123 --add-label "priority/high"

# Add multiple labels
gh issue edit 123 \
  --add-label "priority/high" \
  --add-label "type/automation" \
  --add-label "scope/all-repos" \
  --add-label "effort/2"

# Remove label
gh issue edit 123 --remove-label "status/blocked"
```

### With GitHub API

```bash
# Add labels via API
curl -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/H-J-Petroleum/hjps-toolkit/issues/123/labels \
  -d '{"labels":["priority/high","type/automation"]}'
```

## Label Queries

### Useful Filters

**In GitHub Issues:**

```
# All critical work
is:issue is:open label:priority/critical

# AI-ready quick wins
is:issue is:open label:ai/ready label:effort/1

# Multi-repo high priority
is:issue is:open label:scope/all-repos label:priority/high

# Blocked items needing attention
is:issue is:open label:status/blocked

# AI work in progress
is:issue is:open label:ai/in-progress
```

**In GitHub Projects:**

- Create filtered views using these queries
- Save common filters for quick access
- Use in automation rules

## Migration from Old Labels

### Mapping Old → New

```yaml
# If you have existing labels:
bug → type/bug
enhancement → type/feature
documentation → type/documentation
high-priority → priority/high
quick-win → effort/1
needs-review → status/in-review
```

### Migration Script

```bash
# Rename existing labels
gh label edit "bug" \
  --repo H-J-Petroleum/hjps-toolkit \
  --name "type/bug"

# Or delete and recreate
gh label delete "bug" --repo H-J-Petroleum/hjps-toolkit
# Then create with new system
```

## Maintenance

### Regular Reviews

**Monthly:**

- Audit label usage (are all labels being used?)
- Check for unlabeled issues
- Verify label consistency across repos

**Quarterly:**

- Review if new labels are needed
- Consider retiring unused labels
- Update documentation

### Label Hygiene

**Do's:**
✅ Apply labels when creating issues
✅ Update labels as work progresses
✅ Use standard labels (don't create ad-hoc labels)
✅ Keep labels in sync across repos

**Don'ts:**
❌ Create duplicate labels with different colors
❌ Use too many labels (causes confusion)
❌ Leave issues unlabeled
❌ Change label meanings without documentation

## Advanced Usage

### Label Automation

**GitHub Actions:**

```yaml
# .github/workflows/label-issues.yml
name: Auto-label Issues
on:
  issues:
    types: [opened]
jobs:
  label:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/labeler@v4
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}
```

**Probot:**
Use GitHub Apps like Probot for advanced label automation.

### Label-Based Workflows

**Example: Auto-assign to AI**

```yaml
# If labeled ai/ready, assign to AI agent
on:
  issues:
    types: [labeled]
jobs:
  auto-assign:
    if: github.event.label.name == 'ai/ready'
    runs-on: ubuntu-latest
    steps:
      - run: |
          gh issue edit ${{ github.event.issue.number }} \
            --add-assignee ai-claude
```

## Troubleshooting

### Labels not syncing across repos

Use `sync-labels-all-repos.sh` script to apply consistently.

### Too many labels to manage

Focus on the 4 required categories first. Add others gradually.

### Team not using labels consistently

- Provide training/documentation
- Add labels as PR requirement
- Use automation to enforce

### Labels don't match GitHub Projects fields

Projects can read labels as filters. Map label values to field values.

## Resources

- [labels.yml Template](../../templates/.github/labels.yml)
- [Sync Script](../../scripts/sync-labels.sh)
- [GitHub Labels Documentation](https://docs.github.com/en/issues/using-labels-and-milestones-to-track-work/managing-labels)
- [Kubernetes Label Taxonomy](https://github.com/kubernetes/test-infra/blob/master/label_sync/labels.md)

---

**Version**: 1.0.0
**Last Updated**: 2025-10-13
**Standards**: Based on Kubernetes + Sane GitHub Labels + H&J custom
