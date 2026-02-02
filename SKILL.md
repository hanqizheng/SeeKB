---
name: seekb
description: GitHub Knowledge Base Manager - Search, bookmark, and selectively clone repositories with layered storage strategy
---

# SeeKB - GitHub Knowledge Base Manager

A layered knowledge base system for GitHub. Prioritizes lightweight bookmarking, clones only when necessary.

## Core Philosophy

```
收藏优先，按需克隆，及时清理
```

| 层级 | 内容 | 磁盘占用 | 适用场景 |
|-----|------|---------|---------|
| **Bookmark** | URL + 元数据 | ~1KB | 发现有用的，先记下 |
| **Shallow** | `--depth=1` | 较小 | 需要看源码结构 |
| **Full** | 完整历史 | 大 | 深入学习、贡献代码 |

## Configuration

```bash
export SEEKB_PATH="$HOME/knowledge-base"  # Default
```

## Knowledge Base Structure

```
$SEEKB_PATH/
├── CLAUDE.md                 # Master catalog (bookmarks + cloned)
├── cloned/
│   ├── repo-a/              # Cloned repos live here
│   │   └── CLAUDE.md
│   └── repo-b/
│       └── CLAUDE.md
└── archives/                 # Archived notes (code deleted)
    └── repo-c.md
```

---

## Workflows

### 0. Preflight (gh auth)

```bash
gh auth status
```

### 1. Search GitHub

```bash
# Search repositories
gh search repos "query" --limit 10
gh search repos "query" --language=python --stars=">1000" --json name,owner,description,stargazersCount,url

# Search issues
gh search issues "query" --repo owner/repo --state open --json number,title,state,url

# Search PRs
gh search prs "query" --repo owner/repo --json number,title,state,url
```

### 2. Bookmark Repository (Lightweight)

When user wants to save a repo for later, **DO NOT clone**. Just add to catalog:

```bash
KB_PATH="${SEEKB_PATH:-$HOME/knowledge-base}"
mkdir -p "$KB_PATH"

# Get repo metadata
gh repo view owner/repo --json name,description,url,stargazerCount,primaryLanguage,repositoryTopics
```

Then add entry to `$KB_PATH/CLAUDE.md` with status `bookmark` (see `templates/catalog-entry.md` and `references/catalog-format.md`).

### 3. Online Inspection (No Clone)

Fetch information without cloning:

```bash
# Resolve default branch (avoid hardcoding "main")
DEFAULT_BRANCH=$(gh repo view owner/repo --json defaultBranchRef --jq '.defaultBranchRef.name')

# View repo info
gh repo view owner/repo

# Get directory structure
gh api "repos/owner/repo/git/trees/$DEFAULT_BRANCH?recursive=1" --jq '.tree[].path' | head -50

# Read README (auto-detect)
gh api "repos/owner/repo/readme?ref=$DEFAULT_BRANCH" --jq '.content' | base64 -d

# Read any file
gh api "repos/owner/repo/contents/path/to/file.js?ref=$DEFAULT_BRANCH" --jq '.content' | base64 -d

# Get recent commits
gh api "repos/owner/repo/commits?sha=$DEFAULT_BRANCH" --jq '.[0:5] | .[].commit.message'

# Get releases
gh release list --repo owner/repo --limit 5
```

### 4. Clone Repository (When Needed)

Only clone when user needs to:
- Search across multiple files
- Understand complex architecture
- Run/test the code locally

```bash
KB_PATH="${SEEKB_PATH:-$HOME/knowledge-base}"
mkdir -p "$KB_PATH/cloned"

# Shallow clone (recommended default)
gh repo clone owner/repo "$KB_PATH/cloned/repo" -- --depth=1

# Full clone (only if history needed)
gh repo clone owner/repo "$KB_PATH/cloned/repo"
```

After cloning:
1. Generate `CLAUDE.md` for the repo (see templates/repo-claude.md)
2. Update catalog entry status from `bookmark` to `cloned`

### 5. Query Local Knowledge Base

```bash
KB_PATH="${SEEKB_PATH:-$HOME/knowledge-base}"

# View all entries (bookmarks + cloned)
cat "$KB_PATH/CLAUDE.md"

# List cloned repos only
ls "$KB_PATH/cloned/"

# Search in catalog
grep -i "keyword" "$KB_PATH/CLAUDE.md"

# Search in cloned repos
grep -r "pattern" "$KB_PATH/cloned/" --include="*.py"
```

### 6. Cleanup - Archive Learned Repos

When user has finished learning from a repo:

```bash
KB_PATH="${SEEKB_PATH:-$HOME/knowledge-base}"
REPO="repo-name"

# Create archive note from template
mkdir -p "$KB_PATH/archives"
cp "templates/archive-note.md" "$KB_PATH/archives/$REPO.md"

# Fill placeholders using the repo's CLAUDE.md
cat "$KB_PATH/cloned/$REPO/CLAUDE.md"

# Delete cloned code
rm -rf "$KB_PATH/cloned/$REPO"

# Update catalog: change status from 'cloned' to 'archived'
```

### 7. Update Cloned Repos

```bash
KB_PATH="${SEEKB_PATH:-$HOME/knowledge-base}"

# Update specific repo
git -C "$KB_PATH/cloned/repo-name" pull

# Check for updates without pulling
git -C "$KB_PATH/cloned/repo-name" fetch
git -C "$KB_PATH/cloned/repo-name" status
```

---

## Decision Tree

```
User Request
    │
    ├─ "搜索 GitHub..."
    │   └─ gh search repos/issues/prs
    │
    ├─ "收藏/记录这个仓库"
    │   └─ Bookmark: 只加目录，不克隆
    │
    ├─ "看看这个仓库的 README/结构"
    │   └─ Online: gh api 在线获取
    │
    ├─ "我要深入学习/本地运行"
    │   └─ Clone: 浅克隆到 cloned/
    │
    ├─ "我的知识库有什么"
    │   └─ Query: 读取 CLAUDE.md
    │
    ├─ "这个学完了，清理掉"
    │   └─ Archive: 保留笔记，删除代码
    │
    └─ "搜索本地代码"
        └─ 只能搜索 cloned/ 下的仓库
```

## Catalog Entry Status

| Status | Meaning | Can Read Code? |
|--------|---------|----------------|
| `bookmark` | Only metadata saved | ❌ Use gh api |
| `cloned` | Code exists locally | ✅ Full access |
| `archived` | Notes kept, code deleted | ❌ Re-clone if needed |

---

## Best Practices

1. **默认只收藏** - 发现有趣的先 bookmark，别急着 clone
2. **在线优先** - 简单查看用 gh api，不要克隆
3. **浅克隆优先** - 需要克隆时用 `--depth=1`
4. **及时清理** - 学完的仓库归档，释放空间
5. **保持索引完整** - 即使删除代码，目录里仍保留记录

## Error Handling

```bash
# Check gh auth
gh auth status

# If rate limited
gh api rate_limit --jq '.resources.core.remaining'

# If file too large for API (>1MB)
# Must clone to read
```

## References

- [gh-commands.md](references/gh-commands.md) - CLI reference
- [catalog-format.md](references/catalog-format.md) - Catalog format spec
- [repo-claude.md](templates/repo-claude.md) - Repo CLAUDE.md template
- [catalog-entry.md](templates/catalog-entry.md) - Entry templates
