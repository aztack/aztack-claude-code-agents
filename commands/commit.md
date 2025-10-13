---
description: Commit specific file groups identified by commit message generator with custom commit messages
argument-hint: "<unique-identifier> <type>(<scope>): <description>"
---

# Git Commit Executor for Specific File Groups

You are a specialized git workflow agent that commits specific file groups identified by the commit message generator, ensuring precise and atomic commits.

## Context

After analyzing git diffs with `/commit_message`, developers receive organized commit groups with unique identifiers. This command allows committing ONLY the files associated with a specific group, maintaining clean and focused git history.

This ensures atomic commits where each commit represents a single logical change, making git history meaningful for code review, bisecting, and rollbacks.

## Requirements

**Task:** Commit files for a specific commit group with custom message

**Arguments:** `$ARGUMENTS` in format: `<unique-identifier> <type>(<scope>): <description>`

Example:
```bash
/commit create-shared-module feat(shared): create shared module for types and utils
```

**CRITICAL:** Only add files listed under the specified `<unique-identifier>` from the previous `/commit_message` output. NEVER use `git add .` or `git add -A`.

## Commit Workflow

### 1. Parse Command Arguments

Extract from `$ARGUMENTS`:
- `<unique-identifier>`: Name of commit group (e.g., `create-shared-module`)
- `<type>`: Conventional Commits type (feat, fix, docs, etc.)
- `<scope>`: Optional module/component name in parentheses
- `<description>`: Commit message in imperative mood

### 2. Retrieve File List

Look up the `<unique-identifier>` in the previous `/commit_message` output to get the exact file list:

```markdown
create-shared-module:
feat(shared): create shared module for types and utils
- apps/project-shared/src/index.ts:1
- apps/project-shared/src/types/auth.ts:1
- apps/project-shared/src/utils/index.ts:1
```

Extract file paths:
- `apps/project-shared/src/index.ts`
- `apps/project-shared/src/types/auth.ts`
- `apps/project-shared/src/utils/index.ts`

### 3. Stage Files Selectively

**IMPORTANT:** Add ONLY the files associated with this commit group. This works for BOTH tracked and untracked files.

```bash
# ✅ CORRECT: Add specific files only (works for tracked and untracked files)
git add apps/project-shared/src/index.ts
git add apps/project-shared/src/types/auth.ts
git add apps/project-shared/src/utils/index.ts

# Note: git add works the same for both:
# - Modified tracked files (already in repository)
# - New untracked files (not yet in repository)

# ❌ WRONG: Never do this
git add .
git add -A
git add apps/project-shared/  # May include unrelated files
```

### 4. Create Commit

Generate commit message from user-provided type, scope, and description:

```bash
git commit -m "$(cat <<'EOF'
feat(shared): create shared module for types and utils

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

**Commit Message Format:**
```
<type>(<scope>): <description>

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

### 5. Verify and Report

```bash
# Verify commit created successfully
git log -1 --oneline

# Show remaining uncommitted changes
git status --short

# List remaining commit groups
```

**Output:**
```markdown
✅ Committed: create-shared-module
📝 Commit: a1b2c3d feat(shared): create shared module for types and utils

📋 Remaining uncommitted changes:
- M apps/project-api/controller/org.ts (modified)
- M apps/project-api/controller/marketplace.ts (modified)
- ?? src/new-feature/index.ts (untracked)

🔄 Remaining commit groups:
- support-batch-invitation: feat(api): add batch invitation support
- add-pagination-support: feat(api): add pagination to search endpoints

**Next:** Run `/commit <identifier> <message>` for remaining groups
```

## Usage Examples

### Example 1: Standard Feature Commit

**Input:**
```bash
/commit create-shared-module feat(shared): create shared module for types and utils
```

**Process:**
1. Look up `create-shared-module` file list
2. Add only those specific files: `git add apps/project-shared/src/index.ts apps/project-shared/src/types/auth.ts ...`
3. Commit with message: `feat(shared): create shared module for types and utils`
4. Report success and remaining changes

### Example 2: Bug Fix Without Scope

**Input:**
```bash
/commit fix-auth-token fix: resolve JWT expiration validation bug
```

**Process:**
1. Look up `fix-auth-token` file list: `apps/api/middleware/jwt.ts`
2. Add file: `git add apps/api/middleware/jwt.ts`
3. Commit: `fix: resolve JWT expiration validation bug`

### Example 3: Breaking Change

**Input:**
```bash
/commit redesign-api-auth feat(api)!: redesign authentication flow
```

**Process:**
1. Look up `redesign-api-auth` files
2. Add specific files
3. Commit with breaking change indicator (`!`)
4. Add `BREAKING CHANGE:` footer if needed

**Commit Message:**
```
feat(api)!: redesign authentication flow

Token format changed from JWT to opaque tokens.
Clients must re-authenticate after deployment.

BREAKING CHANGE: Authentication token format changed

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

## Error Handling

### Identifier Not Found
```markdown
❌ Error: Commit group 'invalid-name' not found in previous analysis.

**Available groups:**
- create-shared-module
- support-batch-invitation
- add-pagination-support

Run `/commit_message` to regenerate or use an existing identifier.
```

### No Files to Commit
```markdown
❌ Error: No changes found for commit group 'create-shared-module'.

Files may already be committed. Run `git status` to verify.
```

### Invalid Commit Message Format
```markdown
❌ Error: Invalid commit message format.

**Expected:** <type>(<scope>): <description>
**Received:** $ARGUMENTS

**Examples:**
- feat(auth): add OAuth2 support
- fix: resolve memory leak in cache
- docs(api): update authentication guide

**Valid types:** feat, fix, docs, style, refactor, perf, test, chore
```

### Commit Fails (Pre-commit Hook)
```markdown
⚠️ Commit failed due to pre-commit hook.

**Common Issues:**
- Linting errors: Run `npm run lint:fix` or `yarn lint:fix`
- Type errors: Run `npm run type-check` or `tsc --noEmit`
- Test failures: Run `npm test` to identify failing tests

**Options:**
1. Fix issues and retry: `/commit <identifier> <message>`
2. Skip hooks (not recommended): Add `--no-verify` flag
```

## Git Safety Protocol

**ALWAYS Follow:**
- ✅ Add ONLY files listed for the specific identifier
- ✅ Use exact file paths from `/commit_message` output
- ✅ Verify commit created with `git log -1`
- ✅ Report remaining changes and commit groups
- ✅ Never skip pre-commit hooks unless explicitly requested
- ✅ Include attribution footer in commit message

**NEVER Do:**
- ❌ Use `git add .` or `git add -A`
- ❌ Add files not associated with the identifier
- ❌ Modify git config
- ❌ Force operations without user consent
- ❌ Skip pre-commit hooks automatically
- ❌ Amend commits unless explicitly requested

## Complete Example Workflow

### Step 1: Generate Commit Groups
```bash
/commit_message
```

**Output:**
```markdown
create-shared-module:
feat(shared): create shared module for types and utils
- apps/project-shared/src/index.ts
- apps/project-shared/src/types/auth.ts

add-batch-invitation:
feat(api): add batch invitation support
- apps/project-api/controller/org.ts:60
```

### Step 2: Commit First Group
```bash
/commit create-shared-module feat(shared): create shared module for types and utils
```

**Output:**
```markdown
✅ Committed: create-shared-module
📝 Commit: a1b2c3d feat(shared): create shared module for types and utils

Files committed:
- apps/project-shared/src/index.ts
- apps/project-shared/src/types/auth.ts

📋 Remaining: 1 uncommitted file
🔄 Next: /commit add-batch-invitation feat(api): add batch invitation support
```

### Step 3: Commit Remaining Groups
```bash
/commit add-batch-invitation feat(api): add batch invitation support for organization members
```

**Output:**
```markdown
✅ Committed: add-batch-invitation
📝 Commit: b2c3d4e feat(api): add batch invitation support for organization members

✨ All changes committed! Working directory clean.
```

## Summary

This command enables precise, atomic commits by:
1. **Selective staging**: Only files associated with specific identifier
2. **Custom messages**: User control over commit type, scope, description
3. **Safety**: Never uses `git add .`, preventing accidental commits
4. **Verification**: Reports commit success and remaining changes
5. **Guidance**: Suggests next steps for remaining commit groups

**Prerequisites:**
- Must run `/commit_message` first to generate commit groups
- Must use exact unique identifier from analysis output

**Best Practices:**
- Commit shared/common modules first
- Review commit message before confirming
- Verify remaining changes after each commit
- Keep commits atomic and focused on single logical change
