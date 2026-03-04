---
name: Commit Spelunker
description: Deep-dive analysis tool for exploring git repository history, uncovering contributor patterns, code evolution trends, and hidden insights through advanced git querying and visualization.
version: 1.2.0
author: SMOUJBOT
tags:
  - git
  - history
  - research
  - patterns
  - analytics
  - forensics
dependencies:
  - git >= 2.30
  - bash >= 4.0
  - coreutils
  - grep
  - sed
  - awk
  - dateutils (optional)
  - gnuplot (optional, for graphs)
  - gitstats (optional, for reports)
env_vars:
  - COMMIT_SPELUNKER_MAX_COMMITS: Limit history analysis (default: 10000)
  - COMMIT_SPELUNKER_DATE_FORMAT: Output date format (default: %Y-%m-%d)
  - COMMIT_SPELUNKER_LOGGING: Enable debug logging (0/1, default: 0)
---

# Commit Spelunker

A specialized skill for exhaustive exploration of git repository history. Extracts meaningful patterns from commit data, analyzes contributor behavior, tracks file evolution, and generates actionable insights about codebase health and development practices.

## Purpose

Real-world use cases:

- **Onboarding research**: Understand codebase evolution before making architectural changes
- **Bug hunt**: Identify when and where specific code patterns were introduced
- **Team analysis**: Measure contributions across teams/individuals, identify knowledge silos
- **Technical debt tracking**: Find "quick fix" commits (single-line changes, "hotfix" messages)
- **Release forensics**: Reconstruct what changed between specific releases
- **Architecture decay detection**: Find files with highest churn rate (frequently changed)
- **Contributor mapping**: Identify who owns which parts of codebase through commit frequency
- **Merge conflict prediction**: Identify files with high concurrent modification rates
- **Compliance audit**: Track when specific licenses, copyright headers, or security patterns were added
- **Dead code detection**: Find files/features that haven't been touched in years

## Scope

Supported operations:

```bash
# Pattern search in commit messages
commits-by-message --pattern "bugfix|hotfix|quick" --since "6 months ago"
commits-by-message --pattern "refactor" --author "john" --exclude "docs"

# Contributor analysis
contributor-stats --top 10 --metric commits
contributor-stats --active-days --per-file
contributor-heatmap --since "1 year ago"

# File evolution tracking
file-history <path/to/file> --detailed
file-churn --top 20 --min-commits 5
file-ownership <path> --show-blame-summary

# Time-based analysis
commit-rhythm --interval weekly --heatmap
commit-punch-card --last 6 months
dead-code-finder --stale-years 2

# Code relation analysis
coupling-finder <path> --co-modified-with
feature-birth-tracking --pattern "feature:.*login"

# Release archaeology
diff-between-tags v1.2.0 v1.3.0 --stats-only
changes-under-tag <tag> --group-by-author

# Merge analysis
merge-forensics --complex-merges --show-conflicts
branch-lifespan-analysis --merged-only
```

## Detailed Work Process

### Phase 1: Repository Assessment
```bash
# Quick health check
git rev-parse --is-inside-work-tree
git remote -v
git branch -a | wc -l
git log --oneline | wc -l
```

### Phase 2: Contributor Landscape
```bash
# Top contributors by commit count
git shortlog -sne --all | head -20

# Contribution recency (last 90 days)
git log --since="90 days ago" --pretty="%an" | sort | uniq -c | sort -rn | head -15

# Commit frequency by weekday/hour
git log --pretty="%ad" --date=local | awk '{print $4}' | cut -d: -f1 | sort | uniq -c
```

### Phase 3: File Deep Dive
```bash
# Most changed files (high churn)
git log --pretty=format: --name-only | sort | uniq -c | sort -rg | head -20

# Files with most distinct authors
git log --pretty="%an" --name-only | awk 'NF==1{file=$0;next}{print file":"$0}' | \
  sort | uniq | awk -F: '{count[$1]++} END {for (f in count) print count[f], f}' | sort -rn | head -15

# Largest commits (by file count)
git log --oneline --numstat | awk 'NF==3 {add+=$1+$2; files++} END {print files, add}'
```

### Phase 4: Pattern Detection
```bash
# Find "work in progress" or incomplete commits
git log --grep="WIP\|TODO\|FIXME\|XXX" --oneline

# Identify quick fixes (single-line changes)
git log --oneline --numstat | awk 'NF==3 && ($1+$2)==1 {print $0}'

# Track feature development lifecycle
git log --grep="^feat(.*)" --oneline | wc -l
```

### Phase 5: Temporal Analysis
```bash
# Monthly commit pattern (for capacity planning)
git log --since="2 years ago" --pretty="%s%ad" --date=format:%Y-%m | sort | uniq -c

# Time between commits (developer velocity)
git log --pretty="%at" | awk 'NR>1 {print $1-prev} {prev=$1}' | \
  awk '{sum+=$1; count++} END {print "Avg seconds between commits:", sum/count}'
```

### Phase 6: Relationship Mapping
```bash
# Files often changed together (coupling)
git log --pretty=format: --name-only | awk 'NF>0 {file=$0; getline; while (getline && NF>0) print file","$0}' | \
  tr ',' '\n' | sort | uniq -c | sort -rn | head -20

# Branch merge patterns
git log --merges --oneline --pretty="%s" | cut -d'(' -f2 | cut -d')' -f1 | sort | uniq -c
```

### Phase 7: Reporting & Export
```bash
# Export to CSV for spreadsheet analysis
git log --pretty="%H,%an,%ae,%ad,%s" --date=iso > commits_export.csv

# Generate visualization (if gitstats available)
gitstats /path/to/repo /output/directory
```

## Golden Rules

1. **Performance awareness**: Always use `--since`, `--until`, `--max-count` on large repos. Default limit: `COMMIT_SPELUNKER_MAX_COMMITS=10000`
2. **Respect .gitattributes**: Honor rename detection settings (`-M` flag toggles)
3. **Preserve context**: Never include raw file diffs in logs; use `--stat` instead of full diff
4. **Author attribution**: Use canonical email (`git shortlog -sne` shows authoritative mapping)
5. **Time zones**: Normalize to UTC for cross-region analysis (`--date=iso-strict`)
6. **Binary files**: Exclude from churn analysis (they show as "0" lines changed)
7. **Merge commits**: Decide upfront whether to include (`--no-merges` vs `--merges`)
8. **Security**: Never output full commit hashes in public reports; truncate to 8 chars
9. **Reproducibility**: All analysis scripts must log exact git command used
10. **Non-destructive**: All operations are read-only; never checkout or reset during spelunking

## Examples

### Example 1: Find the most controversial files (most authors)
```bash
# Input
commit-spelunker file-ownership --path src/ --min-authors 3

# Output
   12 src/utils/helpers.js
    8 src/components/Button.tsx
    7 src/services/api.ts
    5 src/hooks/useAuth.ts
```

### Example 2: Detect "quick fix" commits (likely technical debt)
```bash
# Input
commit-spelunker pattern-search --pattern "quick|hotfix|temporary" --exclude "docs|test" --last 30

# Output
a1b2c3d4 (2025-03-01) alice: Quick fix for login crash
e5f6g7h8 (2025-02-28) bob: Hotfix: race condition in checkout
i9j0k1l2 (2025-02-27) charlie: Temporary workaround for Safari
```

### Example 3: Contributor punch card (WHO codes WHEN)
```bash
# Input
commit-spelunker punch-card --last 12 weeks --output matrix.tsv

# Output (TSV):
Mon Tue Wed Thu Fri Sat Sun
  15  12  18  14  11   3   1  # Week 1
  14  16  13  17  12   2   0  # Week 2
...
```

### Example 4: Feature birth tracking
```bash
# Input
commit-spelunker feature-tracking --regex "^feat:.*payment" --show-evolution

# Output
Feature: payment-processing
  Born:  2024-08-15 (commit 7a8b9c0) - "feat: add Stripe integration"
  Matured: 2024-10-22 (commit d1e2f3a) - "feat(payment): webhook handling"
  Stabilized: 2025-01-05 (commit 4g5h6i7) - "fix: idempotency keys"
  Current owners: alice (68%), bob (22%), charlie (10%)
```

### Example 5: Churn heat map for refactoring prioritization
```bash
# Input
commit-spelunker churn-heatmap --top 15 --format json > churn.json

# Output (JSON excerpt):
{
  "files": [
    {"path": "src/store/cart.ts", "commits": 142, "authors": 8, "lines_changed": 3847},
    {"path": "src/components/ProductGrid.js", "commits": 128, "authors": 6, "lines_changed": 2912}
  ]
}
```

## Rollback Commands

Commit Spelunker is read-only by design. Rollback actions only apply to temporary files:

```bash
# Remove temporary analysis outputs
rm -f /tmp/commit-spelunker-*.{csv,tsv,json,txt}

# Clear git reflog entries created during spelunking (none by default)
# Note: Skill never creates reflog entries

# Reset any file permissions changes (if running with elevated perms)
chmod -R u+rwX,go+rX /tmp/commit-spelunker-*/

# If using gitstats and want to remove generated reports
rm -rf /var/tmp/gitstats-output/

# Environment cleanup
unset COMMIT_SPELUNKER_*
```

## Verification Steps

After running any Commits Spelunker operation:

```bash
# 1. Check exit code
echo $?
# Should be 0 for success

# 2. Verify output file exists (if --output used)
test -f analysis.csv && echo "Output created" || echo "No output"

# 3. Validate CSV format (if applicable)
head -1 analysis.csv | grep -q "," && echo "CSV valid" || echo "Invalid CSV"

# 4. Ensure no secrets leaked in output
grep -iE "(password|token|secret|key)" analysis.* && echo "WARNING: Secrets detected!" || echo "Clean"

# 5. Confirm git repo integrity preserved
git status --short | grep -q '^M\|^D\|^A' && echo "Repo modified!" || echo "Repo clean"
```

## Troubleshooting

**Symptom**: "fatal: ambiguous argument 'HEAD'"
- **Cause**: Not in git repository
- **Fix**: `cd /path/to/repo` before running skill

**Symptom**: "Argument list too long" on large repos
- **Cause**: Exceeding shell arg limit
- **Fix**: Set `COMMIT_SPELUNKER_MAX_COMMITS=5000` or use `--batch-size` flag

**Symptom**: Merge commits skewing author statistics
- **Cause**: Merge commits counted as regular commits
- **Fix**: Add `--no-merges` flag or set `COMMIT_SPELUNKER_EXCLUDE_MERGES=1`

**Symptom**: Rename detection not working
- **Cause**: Git rename detection disabled globally
- **Fix**: Use `--detect-renames` flag manually or check `.gitattributes`

**Symptom**: Slow performance on monorepo
- **Cause**: Analyzing entire history (500k+ commits)
- **Fix**: Use `--since "1 year ago"` or `--max-count 10000`

**Symptom**: Binary files showing in churn analysis
- **Cause**: Binary files counted as "1 line changed"
- **Fix**: Pipe through `--exclude "*.png,*.jpg,*.pdf"` or use filter script

**Symptom**: Dates in wrong timezone
- **Cause**: Local git config vs UTC requirement
- **Fix**: Set `export GIT_AUTHOR_DATE=UTC` or use `--date=iso-strict`
```