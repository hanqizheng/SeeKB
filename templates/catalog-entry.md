# Catalog Entry Templates

Templates for adding entries to the master catalog.

## Entry Formats by Status

### Bookmark (Default)

When first saving a repo, always start as bookmark:

```markdown
| [owner/repo](https://github.com/owner/repo) | {{DESCRIPTION}} | {{LANGUAGE}} | bookmark | {{DATE}} |
```

**Example:**
```markdown
| [anthropics/claude-code](https://github.com/anthropics/claude-code) | CLI for Claude AI | TypeScript | bookmark | 2024-01-15 |
```

### Cloned

After cloning to local:

```markdown
| [repo](./cloned/repo/) | {{DESCRIPTION}} | {{LANGUAGE}} | cloned | {{DATE}} |
```

**Example:**
```markdown
| [claude-code](./cloned/claude-code/) | CLI for Claude AI | TypeScript | cloned | 2024-01-15 |
```

### Archived

After removing code but keeping notes:

```markdown
| [repo](./archives/repo.md) | {{DESCRIPTION}} | {{LANGUAGE}} | archived | {{DATE}} |
```

**Example:**
```markdown
| [claude-code](./archives/claude-code.md) | CLI for Claude AI | TypeScript | archived | 2024-01-15 |
```

---

## Quick Commands

### Get Info for New Bookmark

```bash
# Get all needed metadata
gh repo view owner/repo --json name,owner,description,primaryLanguage,stargazerCount,repositoryTopics

# One-liner to format bookmark entry
gh repo view owner/repo --json name,owner,description,primaryLanguage \
  --template '| [{{.owner.login}}/{{.name}}](https://github.com/{{.owner.login}}/{{.name}}) | {{.description | truncate 40}} | {{.primaryLanguage.name}} | bookmark | '
echo "$(date +%Y-%m-%d) |"

# Fallback (no --template support; requires jq)
DATE=$(date +%Y-%m-%d)
gh repo view owner/repo --json name,owner,description,primaryLanguage \
  | jq -r --arg date "$DATE" \
  '"| [\(.owner.login)/\(.name)](https://github.com/\(.owner.login)/\(.name)) | \((.description // "")[0:40]) | \((.primaryLanguage // {}) | .name // "") | bookmark | \($date) |"'
```

### Batch Bookmark from Search

```bash
# Search and format as bookmark entries
gh search repos "query" --limit 5 --json owner,name,description,primaryLanguage \
  --template '{{range .}}| [{{.owner.login}}/{{.name}}](https://github.com/{{.owner.login}}/{{.name}}) | {{.description | truncate 40}} | {{.primaryLanguage.name}} | bookmark | {{now | date "2006-01-02"}} |
{{end}}'

# Fallback (no --template support; requires jq)
DATE=$(date +%Y-%m-%d)
gh search repos "query" --limit 5 --json owner,name,description,primaryLanguage \
  | jq -r --arg date "$DATE" \
  '.[] | "| [\(.owner.login)/\(.name)](https://github.com/\(.owner.login)/\(.name)) | \((.description // "")[0:40]) | \((.primaryLanguage // {}) | .name // "") | bookmark | \($date) |"'
```

---

## Status Transitions

```
                    ┌─────────────┐
                    │  (discover) │
                    └──────┬──────┘
                           ▼
                    ┌─────────────┐
          ┌─────────│  bookmark   │─────────┐
          │         └──────┬──────┘         │
          │                │                │
          │           (clone)               │
          │                │                │
          │                ▼           (remove)
          │         ┌─────────────┐         │
          │         │   cloned    │         │
          │         └──────┬──────┘         │
          │                │                │
          │          (archive)              │
          │                │                │
          │                ▼                │
          │         ┌─────────────┐         │
          └────────▶│  archived   │◀────────┘
                    └──────┬──────┘
                           │
                      (remove)
                           │
                           ▼
                    ┌─────────────┐
                    │  (deleted)  │
                    └─────────────┘
```

---

## Section Templates

### By Status Section

```markdown
## By Status

### Bookmarked
- [owner/repo1](https://github.com/owner/repo1) - Description
- [owner/repo2](https://github.com/owner/repo2) - Description

### Cloned
- [repo3](./cloned/repo3/) - Description
- [repo4](./cloned/repo4/) - Description

### Archived
- [repo5](./archives/repo5.md) - Description
```

### By Topic Section

```markdown
## By Topic

### Machine Learning
- owner/ml-repo (bookmark)
- local-ml (cloned)

### Web Development
- owner/web-framework (bookmark)
- my-app (cloned)
- old-project (archived)
```

---

## Validation Checklist

Before adding an entry:

- [ ] Repository exists and is accessible
- [ ] Description is under 50 characters
- [ ] Language field matches GitHub's detection
- [ ] Date is in YYYY-MM-DD format
- [ ] Status is one of: `bookmark`, `cloned`, `archived`
- [ ] Link matches status:
  - bookmark → GitHub URL
  - cloned → `./cloned/repo/`
  - archived → `./archives/repo.md`

Before changing status:

- [ ] bookmark → cloned: Directory exists at `./cloned/repo/`
- [ ] cloned → archived: Notes saved at `./archives/repo.md`
- [ ] Any → removed: Entry deleted from all sections
