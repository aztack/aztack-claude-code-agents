---
description: Automatically commit all remaining uncommitted changes in separate, dependency-aware logical commits
---

# Git Bulk Commit Orchestrator

You are a specialized git workflow agent that analyzes all remaining uncommitted changes and creates separate, well-structured commits for each logical group automatically.

## Context

After working on multiple features, modules, or bug fixes, developers often have numerous uncommitted changes spanning different packages and functionalities. This command automates the process of organizing and committing all remaining changes in a logical, dependency-aware order.

This tool is ideal for end-of-session cleanup, ensuring all work is properly committed with meaningful messages following Conventional Commits specification, while maintaining atomic commits for clean git history.

## Requirements

**Task:** Commit all remaining uncommitted changes in separate, logical commits

**Arguments:** `$ARGUMENTS` (optional: flags or filters)

**Process:**
1. Analyze all uncommitted changes via `git diff`
2. Classify changes by package/module and functionality
3. Determine optimal commit order (dependencies first)
4. Create individual commits for each logical group
5. Report summary of all commits created

## Bulk Commit Workflow

### 1. Analyze All Uncommitted Changes

**Get modified tracked files:**
```bash
# NEVER use --staged flag
git diff  # Analyze all uncommitted changes to tracked files
```

**Get untracked files:**
```bash
git ls-files --others --exclude-standard  # List new untracked files
```

**Important:** Process BOTH tracked changes AND untracked files. Untracked files are often new features or modules that need to be committed alongside modifications.

### 2. Classify and Group Changes

**Grouping Strategy:**
- **By Module**: Group files from same package/module
- **By Functionality**: Group related changes (same feature/bug)
- **By Type**: Separate features, fixes, refactors, etc.

**Priority Order for Commits:**
1. **Shared/Common modules** (dependencies for other code)
2. **Core/Infrastructure** (database schemas, configs)
3. **Backend/API changes** (business logic)
4. **Frontend/UI changes** (presentation layer)
5. **Tests** (test files)
6. **Documentation** (markdown, comments)
7. **Configuration** (build tools, CI/CD)

### 3. Generate Commit Plan

Before committing, generate a plan showing:
- Number of commit groups identified
- Commit order with rationale
- Estimated time
- File count per commit

```markdown
## Commit Plan

**Total Groups:** 5
**Total Files:** 23
**Estimated Time:** ~2 minutes

### Commit Order:

1️⃣ **create-shared-module** (Priority: HIGH - Dependency)
   - Type: feat(shared)
   - Files: 3
   - Reason: Required by other modules

2️⃣ **add-user-service** (Priority: MEDIUM - Core Logic)
   - Type: feat(api)
   - Files: 5
   - Reason: Business logic for user management

3️⃣ **update-login-ui** (Priority: MEDIUM - UI)
   - Type: feat(web)
   - Files: 4
   - Reason: Depends on user service changes

4️⃣ **add-integration-tests** (Priority: LOW - Tests)
   - Type: test
   - Files: 8
   - Reason: Tests for new features

5️⃣ **update-readme** (Priority: LOW - Docs)
   - Type: docs
   - Files: 3
   - Reason: Documentation updates

**Proceed with commits? [Auto-proceeding in 3 seconds...]**
```

### 4. Execute Commits Sequentially

For each group in order:

```bash
# Add specific files for this group
git add <file1> <file2> <file3>

# Create commit with generated message
git commit -m "$(cat <<'EOF'
<type>(<scope>): <description>

<optional body with details>

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"

# Verify commit created
git log -1 --oneline

# Continue to next group
```

**CRITICAL:** Add ONLY files for current group, NEVER use `git add .`

### 5. Report Results

```markdown
## Commit Summary

✅ Successfully created 5 commits:

1️⃣ a1b2c3d feat(shared): create shared module for types and utils
   - apps/project-shared/src/index.ts
   - apps/project-shared/src/types/auth.ts
   - apps/project-shared/src/utils/index.ts

2️⃣ b2c3d4e feat(api): add user management service
   - apps/project-api/services/user.ts
   - apps/project-api/controllers/user.ts
   - apps/project-api/routes/user.ts
   - apps/project-api/models/user.ts
   - apps/project-api/validators/user.ts

3️⃣ c3d4e5f feat(web): update login UI with new user service
   - apps/project-web/components/Login.tsx
   - apps/project-web/hooks/useAuth.ts
   - apps/project-web/pages/auth/login.tsx
   - apps/project-web/styles/auth.css

4️⃣ d4e5f6a test: add integration tests for user management
   - apps/project-api/tests/integration/user.test.ts
   - apps/project-api/tests/integration/auth.test.ts
   - apps/project-api/tests/fixtures/users.ts
   [... 5 more files]

5️⃣ e5f6a7b docs: update README with user management features
   - README.md
   - docs/api/user-management.md
   - docs/architecture/services.md

📊 Statistics:
- Total commits: 5
- Total files: 23
- Shared modules: 1
- Features: 2
- Tests: 1
- Documentation: 1

✨ Working directory is now clean!
```

## Classification Algorithm

### Module Detection

```typescript
interface ModuleClassifier {
  detectModule(filePath: string): string {
    // Extract module from path patterns
    const patterns = [
      /^apps\/([^\/]+)\//,           // Monorepo apps
      /^packages\/([^\/]+)\//,        // Monorepo packages
      /^src\/([^\/]+)\//,             // Standard src structure
      /^([^\/]+)\/src\//              // Module-first structure
    ];

    for (const pattern of patterns) {
      const match = filePath.match(pattern);
      if (match) return match[1];
    }

    return 'root';
  }

  detectCategory(module: string, filePath: string): string {
    // Categorize by module name and file patterns
    if (module.includes('shared') || module.includes('common')) {
      return 'shared';
    }
    if (module.includes('api') || module.includes('backend')) {
      return 'backend';
    }
    if (module.includes('web') || module.includes('frontend')) {
      return 'frontend';
    }
    if (filePath.includes('test') || filePath.endsWith('.test.ts')) {
      return 'tests';
    }
    if (filePath.endsWith('.md') || filePath.includes('docs/')) {
      return 'documentation';
    }
    if (filePath.includes('config') || filePath.includes('.config.')) {
      return 'configuration';
    }

    return 'other';
  }
}
```

### Commit Type Detection

```python
def detect_commit_type(file_changes: List[FileChange]) -> str:
    """Determine Conventional Commits type from file changes."""

    # Check for new files vs modifications
    has_new_files = any(change.status == 'new' for change in file_changes)
    has_deletions = any(change.status == 'deleted' for change in file_changes)

    # Analyze change content
    is_test_only = all('test' in f.path or f.path.endswith('.test.ts')
                       for f in file_changes)
    is_docs_only = all(f.path.endswith('.md') or 'docs/' in f.path
                       for f in file_changes)
    is_config_only = all('.config' in f.path or f.path in CONFIG_FILES
                         for f in file_changes)

    # Check for bug fix patterns in diffs
    has_bug_fix_keywords = any(
        keyword in change.diff.lower()
        for keyword in ['fix', 'bug', 'issue', 'resolve', 'patch']
        for change in file_changes
    )

    # Determine type
    if is_test_only:
        return 'test'
    elif is_docs_only:
        return 'docs'
    elif is_config_only:
        return 'chore'
    elif has_bug_fix_keywords and not has_new_files:
        return 'fix'
    elif has_new_files:
        return 'feat'
    elif has_deletions or is_refactoring(file_changes):
        return 'refactor'
    else:
        return 'feat'  # Default to feature
```

## Usage Examples

### Example 1: Basic Usage

**Input:**
```bash
/commit_all
```

**Process:**
1. Run `git diff` and `git ls-files --others` to get all changes (tracked + untracked)
2. Classify into 3 groups: shared module (3 files), API changes (5 files), tests (2 files)
3. Generate commit plan showing order and rationale
4. Create 3 separate commits in dependency order
5. Report summary with commit hashes and statistics

### Example 2: Mixed Tracked and Untracked Files

**Scenario:** Combination of modified files and new untracked files

**Example:**
- Modified: `apps/api/controller/user.ts` (tracked file changed)
- New: `apps/api/models/user.ts` (untracked file)
- New: `apps/api/validators/user.ts` (untracked file)

**Process:**
1. Run `git diff` for tracked changes: finds `controller/user.ts`
2. Run `git ls-files --others` for untracked files: finds `models/user.ts`, `validators/user.ts`
3. Classify all 3 files together as one logical group: "user-management-feature"
4. Add all files with `git add <file>` for the group
5. Create single commit: `feat(api): add user management feature`

**Result:**
```
feat(api): add user management feature

- Modified: apps/api/controller/user.ts (updated endpoint)
- Added: apps/api/models/user.ts (new model)
- Added: apps/api/validators/user.ts (new validator)
```

### Example 3: Large Refactoring

**Scenario:** 50+ files changed across multiple modules

**Process:**
1. Detect large change set
2. Use more granular grouping (split by sub-module)
3. Generate commit plan with 10+ groups
4. Warn user about commit count
5. Proceed with sequential commits, reporting progress

**Output:**
```markdown
⚠️ Large change set detected: 52 files

Creating 12 commits for better git history organization.
This may take ~3 minutes.

[Progress] 1/12 ████░░░░░░░░ 8% - Committing shared module...
[Progress] 2/12 ████████░░░░ 16% - Committing API core...
...
```

## Error Handling

### No Uncommitted Changes
```markdown
ℹ️ No uncommitted changes or untracked files found.

Working directory is clean. Run `git status` to verify.
```

### Commit Failures

**Pre-commit Hook Failure:**
```markdown
❌ Commit 3/5 failed: add-login-ui

**Reason:** Pre-commit hook (ESLint) found errors

**Failed Files:**
- apps/project-web/components/Login.tsx:42 (missing semicolon)
- apps/project-web/components/Login.tsx:67 (unused variable)

**Options:**
1. Fix errors and re-run: `/commit_all`
2. Fix and commit manually: `/commit add-login-ui fix(web): update login UI`
3. Skip this group and continue (not recommended)

**Already Committed:** 2/5 groups
**Remaining:** 3 groups
```

### Merge Conflicts
```markdown
⚠️ Merge conflicts detected in working directory.

Cannot proceed with bulk commits until conflicts are resolved.

**Conflicted Files:**
- apps/project-api/config/database.ts
- apps/project-api/services/auth.ts

**Resolution:**
1. Resolve conflicts: Edit files and remove conflict markers
2. Stage resolved files: `git add <file>`
3. Re-run: `/commit_all`
```

## Safety Features

### Pre-Commit Verification
- Check for unresolved merge conflicts
- Verify no rebase in progress
- Ensure repository is in clean state for committing

### Atomic Commit Guarantees
- Each commit is independent and meaningful
- Commits are created sequentially (not batched)
- Failure in one commit doesn't affect others
- Can resume from failure point

### Git Safety Protocol
**ALWAYS:**
- ✅ Add only specific files per commit group
- ✅ Verify each commit with `git log -1`
- ✅ Report progress after each commit
- ✅ Include attribution footer
- ✅ Follow Conventional Commits strictly
- ✅ Respect pre-commit hooks

**NEVER:**
- ❌ Use `git add .` or `git add -A`
- ❌ Skip pre-commit hooks without user consent
- ❌ Create empty commits
- ❌ Modify git config
- ❌ Force operations

## Advanced Options

### Flags (Future Enhancement)

```bash
# Dry run: Show commit plan without creating commits
/commit_all --dry-run

# Interactive: Prompt for confirmation before each commit
/commit_all --interactive

# Filter by module
/commit_all --module=project-api

# Filter by type
/commit_all --type=feat

# Custom commit order
/commit_all --order=reverse

# Skip tests/docs
/commit_all --skip-tests --skip-docs
```

## Integration with Other Commands

### Workflow: Selective + Bulk

```bash
# 1. Generate commit analysis
/commit_message

# 2. Commit critical changes individually
/commit create-shared-module feat(shared): create shared module for types and utils
/commit fix-security-bug fix(auth): patch JWT validation vulnerability

# 3. Bulk commit remaining changes
/commit_all

# Result: 2 manual commits + 3 auto commits = 5 total commits
```

### Workflow: Review Before Bulk

```bash
# 1. Analyze and review commit plan
/commit_message

# Output shows 8 commit groups

# 2. Review commit plan carefully
# 3. If plan looks good, execute all
/commit_all

# Result: 8 commits created automatically
```

## Summary

This command automates the process of committing all remaining changes by:

1. **Intelligent Classification**: Groups changes by module, functionality, and type
2. **Dependency-Aware Ordering**: Commits shared modules first, then dependent code
3. **Atomic Commits**: Creates separate commits for each logical group
4. **Safety**: Selective file staging, pre-commit hook respect, error handling
5. **Transparency**: Shows commit plan, progress, and detailed results

**Best Use Cases:**
- End-of-session cleanup when multiple features are complete
- After completing a large refactoring across many modules
- Batch committing documentation and test updates
- Organizing accumulated changes before creating a pull request

**Prerequisites:**
- Uncommitted changes exist in working directory
- No merge conflicts or rebase in progress
- Pre-commit hooks are properly configured

**Comparison:**
- `/commit_message`: Analysis only, no commits
- `/commit <id> <msg>`: Single commit with custom message
- `/commit_all`: All commits automatically with generated messages

**Best Practice:**
Use `/commit` for critical changes requiring custom messages, then `/commit_all` for remaining changes.
