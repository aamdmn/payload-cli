---
name: payload-cli
description: PayloadCMS CLI for AI agents. Use when the user needs to create, read, update, or delete CMS content, upload/download media files, inspect collection schemas, or manage PayloadCMS data from the command line.
allowed-tools: Bash(npx payload-cli:*), Bash(payload-cli:*), Bash(pnpm payload-cli:*)
---

# payload-cli - PayloadCMS CLI for Agents

A command-line tool that gives you direct access to PayloadCMS data. No MCP, no protocol overhead. Just commands.

## Core Workflow

Always follow this pattern when working with PayloadCMS data:

```bash
# 1. DISCOVER what collections exist
payload-cli collections

# 2. UNDERSTAND the schema before writing
payload-cli describe posts              # TypeScript interface (exact data shape)
payload-cli describe posts --fields     # detailed field breakdown (constraints, localized flags)
payload-cli describe posts --examples   # see json field shapes

# 3. READ existing data
payload-cli find posts --limit 5

# 4. WRITE data
payload-cli create posts --data '{"title": "My Post", "status": "draft"}'

# 5. VERIFY your changes
payload-cli find-by-id posts <id>
```

## Rules

1. **ALWAYS run `payload-cli describe <collection>` before creating or updating documents.** This outputs the TypeScript interface from `payload-types.ts` showing the exact data shape Payload expects — optional fields have `?`, union types show valid values, relationship types show what they resolve to. Use `--fields` for a detailed field breakdown with constraints, localized flags, and defaults. Use `--examples` to see the expected structure of `json` fields (custom editors, tables, etc.).

2. **ALWAYS preview destructive operations.** `payload-cli delete` and `payload-cli delete-many` show a preview by default. Only add `--confirm` after verifying the preview.

3. **Use `--json` when you need to parse output programmatically.** Human-readable output is the default.

4. **Use `--select` to limit returned fields** when you only need specific data. This reduces output size.

5. **Use `--dry-run` for write operations** when you want to validate data without persisting it.

## Command Reference

### Introspection

```bash
payload-cli collections                          # List all collections
payload-cli describe <collection|global>         # Show TypeScript interface (exact data shape)
payload-cli describe <collection> --fields       # Show detailed field breakdown
payload-cli describe <collection> --examples     # Show schema + example json field structures
payload-cli globals                              # List all globals
payload-cli status                               # Show instance status
```

### Reading Data

```bash
# Find documents with optional filtering
payload-cli find <collection> [--where '{"field":{"operator":"value"}}'] [--limit N] [--page N] [--sort field] [--select '{"field":true}'] [--depth N]

# Find a specific document
payload-cli find-by-id <collection> <id> [--depth N] [--select '...']

# Read a global
payload-cli get-global <slug> [--depth N] [--select '...']
```

### Writing Data

```bash
# Create a document
payload-cli create <collection> --data '{"field":"value"}' [--dry-run]

# Create with file upload (auto-uploads file and injects ID)
payload-cli create <collection> --data '{"title":"About"}' --file 'heroImage=./hero.jpg'

# Update a single document
payload-cli update <collection> <id> --data '{"field":"new value"}' [--dry-run]

# Update with file upload
payload-cli update <collection> <id> --data '{}' --file 'heroImage=./new-hero.jpg'

# Update multiple documents
payload-cli update-many <collection> --where '{"field":{"equals":"value"}}' --data '{"field":"new value"}' [--dry-run]

# Update a global
payload-cli update-global <slug> --data '{"field":"value"}' [--dry-run]
```

### Media (Upload / Download)

```bash
# Upload a file to an upload-enabled collection
payload-cli upload <collection> <file|dir> [--data '{"alt":"..."}'] [--dry-run]

# Upload multiple files
payload-cli upload <collection> ./file1.jpg ./file2.png

# Upload all files in a directory
payload-cli upload <collection> ./photos/

# Download a file by ID
payload-cli download <collection> <id> [--out ./path/]

# Download files matching a query
payload-cli download <collection> --where '{"alt":{"contains":"hero"}}' [--out ./path/]
```

### Deleting Data (requires --confirm)

```bash
# Delete a single document (preview first, then confirm)
payload-cli delete <collection> <id>              # Shows preview
payload-cli delete <collection> <id> --confirm    # Executes delete

# Delete multiple documents
payload-cli delete-many <collection> --where '{"status":{"equals":"draft"}}'            # Shows preview
payload-cli delete-many <collection> --where '{"status":{"equals":"draft"}}' --confirm  # Executes delete
```

### Global Flags

| Flag | Description |
|------|-------------|
| `--json` | Output as JSON for machine parsing |
| `--dry-run` | Validate without executing writes |
| `--confirm` | Confirm destructive operations |
| `--config <path>` | Path to payload.config.ts |
| `--include-sensitive` | Include sensitive fields in output |

## Where Clause Syntax

The `--where` flag uses Payload's query syntax as JSON:

```bash
# Equals
--where '{"status":{"equals":"published"}}'

# Not equals
--where '{"status":{"not_equals":"draft"}}'

# Greater than
--where '{"createdAt":{"greater_than":"2024-01-01"}}'

# Contains (text search)
--where '{"title":{"contains":"hello"}}'

# AND (multiple conditions)
--where '{"and":[{"status":{"equals":"published"}},{"title":{"contains":"hello"}}]}'

# OR
--where '{"or":[{"status":{"equals":"draft"}},{"status":{"equals":"archived"}}]}'
```

## Common Patterns

### Create a blog post
```bash
payload-cli describe posts                       # Check required fields
payload-cli create posts --data '{"title":"My New Post","status":"draft","slug":"my-new-post"}'
```

### Find and update a document
```bash
payload-cli find posts --where '{"title":{"contains":"hello"}}' --select '{"id":true,"title":true}'
payload-cli update posts <id> --data '{"status":"published"}'
```

### Bulk publish drafts
```bash
payload-cli find posts --where '{"status":{"equals":"draft"}}' --limit 100   # Preview
payload-cli update-many posts --where '{"status":{"equals":"draft"}}' --data '{"status":"published"}' --dry-run  # Dry run
payload-cli update-many posts --where '{"status":{"equals":"draft"}}' --data '{"status":"published"}'            # Execute
```

### Clean up old content
```bash
payload-cli delete-many posts --where '{"status":{"equals":"archived"}}'            # Preview what will be deleted
payload-cli delete-many posts --where '{"status":{"equals":"archived"}}' --confirm  # Execute after verifying
```

### Upload media and attach to content
```bash
payload-cli describe pages                          # Find upload/relationship fields
payload-cli upload media ./hero.jpg --data '{"alt":"Hero image"}'
payload-cli create pages --data '{"title":"About"}' --file 'heroImage=./hero.jpg'   # Auto-upload + inject
```

### Bulk upload images
```bash
payload-cli upload media ./photos/                  # Upload all files in directory
payload-cli upload media ./img1.jpg ./img2.png      # Upload specific files
```

## Error Handling

payload-cli provides AI-friendly error messages:

- **Unknown field names**: Suggests the closest matching field
- **Missing required fields**: Lists which fields are required
- **Invalid collection**: Shows available collections with suggestions
- **Validation errors**: Tells you exactly what failed and hints at how to fix it

When you see an error, run `payload-cli describe <collection>` to review the schema.

## Output Modes

- **Human mode** (default): Readable tables and formatted output
- **JSON mode** (`--json`): Raw JSON, suitable for piping to `jq` or parsing

Sensitive fields (password hashes, API keys, etc.) are automatically redacted unless `--include-sensitive` is passed.

## Deep-Dive References

- [Full Command Reference](references/commands.md)
- [Schema Discovery Workflow](references/schema-workflow.md)
- [Common Patterns & Recipes](references/common-patterns.md)
