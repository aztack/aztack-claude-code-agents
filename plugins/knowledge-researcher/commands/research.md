---
description: Search for specific knowledge recursively by following markdown links in documentation
argument-hint: "<topic/keywords> [in entry-file.md]"
---

# Knowledge Research Command

You are executing a knowledge research task to find specific information within markdown documentation by recursively following internal links.

## Context

This command helps users discover information scattered across interconnected markdown files in documentation repositories. It intelligently follows markdown links to build a comprehensive understanding of topics that span multiple documents.

## Requirements

**Task**: Search for knowledge about a specific topic by recursively traversing markdown files

**Arguments**: `$ARGUMENTS`

Parse the arguments to extract:
1. **Topic/Keywords** (required): The subject matter to search for
2. **Entry File** (optional): Starting point for the search (can be prefixed with "in")

Example argument patterns:
- `authentication API in docs/api/readme.md` -> topic: "authentication API", entry: "docs/api/readme.md"
- `database migrations` -> topic: "database migrations", entry: (none, use default)
- `"error handling" in docs/guide.md` -> topic: "error handling", entry: "docs/guide.md"

## Research Workflow

### Step 1: Determine Entry Point

**If entry file specified in arguments**:
- Validate the file exists using Read tool
- If not found, inform user and ask for correction
- Use provided path as starting point

**If no entry file specified**:
- Look for `.llms-txt` directory
- If not found, look for `docs/` or `documentation/` directories
- If not found, look for `README.md` in repository root
- If none found, ask user to specify an entry file

### Step 2: Execute Recursive Search

Initialize search parameters:
```
visited_files = []
max_depth = 10
max_files = 100
current_depth = 0
results = []
```

For each file to search:

1. **Read File Content**:
   - Use Read tool to get markdown content
   - Add file to visited_files list
   - Check if max_files limit reached

2. **Search for Topic**:
   - Look for exact phrase matches (case-insensitive)
   - Look for partial keyword matches
   - Extract surrounding context (2-3 paragraphs)
   - Record file path and position

3. **Extract Markdown Links**:
   - Parse markdown link syntax: `[text](path)`, `[text](path.md)`
   - Resolve relative paths based on current file location
   - Filter out external URLs (http://, https://)
   - Filter out non-markdown files unless they're documentation

4. **Follow Links Recursively**:
   - For each markdown link found:
     - Check if already visited (skip if yes)
     - Check if depth limit reached (skip if yes)
     - Recursively search the linked file
     - Increment depth counter

5. **Track Search Progress**:
   - Every 10 files, report progress to user
   - Show: files searched, links followed, matches found

### Step 3: Analyze and Rank Results

For each finding:
- Calculate relevance score based on:
  - Exact phrase match: +10 points
  - Multiple keyword matches: +5 points per match
  - Match in heading: +5 points
  - Match in code block: +3 points
  - Proximity to related terms: +2 points

- Classify relevance:
  - High: score >= 15
  - Medium: score >= 8
  - Low: score < 8

- Sort results by relevance score (descending)

### Step 4: Present Results

Generate structured output:

```markdown
## Research Results: "[topic]"

**Search Statistics**:
- Entry Point: [file-path]
- Files Searched: [count]
- Links Followed: [count]
- Maximum Depth Reached: [depth]
- Total Matches: [count]

---

### High Relevance Findings

#### 1. [Section or File Name]

**Source**: `[file-path]`
**Link Chain**: [entry] -> [file1] -> [file2] -> [current]
**Match Type**: [Exact phrase / Keywords / Partial]

[Extracted content with surrounding context]

---

#### 2. [Next Finding]
...

### Medium Relevance Findings
...

### Low Relevance Findings
...

---

## Summary

[Brief overview of what was found, key themes, and suggestions for further exploration]
```

### Step 5: Handle No Results

If no matches found:

1. Report search statistics (files searched, links followed)
2. Ask user: "I searched [N] files starting from [entry] but found no matches for '[topic]'. The entry file appears correct. Would you like me to:"
   - Option A: Try different search terms
   - Option B: Search from root documentation (.llms-txt)
   - Option C: Specify a different entry file

If user chooses Option B and root search also fails:
- Report comprehensive search statistics
- Suggest the topic might not be documented
- Offer to search with broader terms

## Technical Implementation

### Reading Files

```bash
# Use Read tool for each markdown file
# Example: Read /path/to/docs/file.md
```

### Searching Content

```bash
# Use Grep tool for efficient keyword searching
# Example: grep -i "authentication" docs/**/*.md
```

### Finding Documentation Files

```bash
# Use Glob tool to discover markdown files
# Example: glob pattern="**/*.md" path="docs"
```

### Resolving Relative Paths

For a link `[text](../other/file.md)` found in `/docs/api/guide.md`:
1. Get directory of current file: `/docs/api/`
2. Resolve relative path: `/docs/api/../other/file.md` -> `/docs/other/file.md`
3. Normalize path and verify existence

## Important Rules

### Read-Only Constraint

- **CRITICAL**: This command is READ-ONLY
- NEVER create, modify, or delete any files
- NEVER use Write, Edit, or NotebookEdit tools
- Only use Read, Grep, Glob, and Bash tools for reading/searching
- If user requests modifications, politely decline

### Performance Limits

- Maximum files to search: 100 (configurable)
- Maximum recursion depth: 10 levels
- Maximum file size to read: 1MB per file
- Timeout: 60 seconds total search time
- Progress update interval: every 10 files

### Error Handling

**File Not Found**:
- Report clearly which file is missing
- Show the link chain that led to the missing reference
- Continue searching other branches

**Circular References**:
- Detect when visiting previously seen file
- Skip and log the circular reference
- Continue with other paths

**Parse Errors**:
- Handle malformed markdown gracefully
- Skip problematic links
- Report parsing issues in search statistics

**Permission Errors**:
- Report files that cannot be read
- Continue searching accessible files
- Include error count in statistics

## Example Execution

**User Command**: `/research JWT authentication in docs/security/auth.md`

**Execution Flow**:

1. Parse arguments:
   - Topic: "JWT authentication"
   - Entry: "docs/security/auth.md"

2. Validate entry file exists (Read tool)

3. Search entry file for "JWT authentication":
   - Found 2 matches
   - Extract context
   - Record results

4. Extract links from entry file:
   - Found: `[Token Validation](./validation.md)`
   - Found: `[API Security](../api/security.md)`

5. Follow link 1: `docs/security/validation.md`
   - Search for topic
   - Found 3 matches
   - Extract more links

6. Follow link 2: `docs/api/security.md`
   - Search for topic
   - Found 1 match
   - No relevant links

7. Continue recursively until depth limit or no more links

8. Rank all findings by relevance

9. Present structured results with link chains

## Best Practices

1. **Start Conservative**: Begin with specified entry file
2. **Progress Updates**: Report progress every 10 files for long searches
3. **Context Matters**: Include surrounding paragraphs, not just matching lines
4. **Show Path**: Always display the link chain showing how you found each result
5. **Respect Limits**: Stop at reasonable depth/file limits
6. **Ask Permission**: Always ask before expanding to root documentation
7. **Be Accurate**: Don't hallucinate content; only show what's actually in the files
8. **Handle Failures Gracefully**: Provide helpful suggestions when searches fail

## Summary

This command enables deep, recursive knowledge discovery across interconnected markdown documentation. It combines intelligent link following with relevance-ranked results to help users find information quickly, even when it's scattered across many files. The read-only nature ensures safe operation, while the fallback mechanisms and user confirmations make it flexible and user-friendly.
