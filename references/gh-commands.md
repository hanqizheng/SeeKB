# GitHub CLI (gh) Command Reference

## Table of Contents

- Authentication
- Repository Operations
- Issue Operations
- Pull Request Operations
- API Access
- Common Patterns
- Output Formatting
- Rate Limiting

## Authentication

```bash
# Check authentication status
gh auth status

# Login to GitHub
gh auth login

# Login with specific options
gh auth login --web              # Browser-based login
gh auth login --with-token       # Token from stdin

# Logout
gh auth logout
```

## Repository Operations

### Search Repositories

```bash
# Basic search
gh search repos "query"

# With filters
gh search repos "query" \
  --language=python \
  --stars=">1000" \
  --topic=machine-learning \
  --limit=20

# JSON output with specific fields
gh search repos "query" --json name,owner,description,stargazersCount,url,language,topics

# All available JSON fields for repos:
# name, owner, description, url, stargazersCount, forksCount,
# isArchived, isPrivate, isFork, language, topics, pushedAt,
# createdAt, updatedAt, license
```

### Clone Repository

```bash
# Clone by owner/repo
gh repo clone owner/repo

# Clone to specific directory
gh repo clone owner/repo target-directory

# Clone with specific depth
gh repo clone owner/repo -- --depth=1
```

### View Repository

```bash
# View repo info
gh repo view owner/repo

# View in browser
gh repo view owner/repo --web

# JSON output
gh repo view owner/repo --json name,description,url,stargazerCount
```

### List Repositories

```bash
# List user's repos
gh repo list

# List org's repos
gh repo list organization

# With filters
gh repo list --language=rust --limit=50
```

## Issue Operations

### Search Issues

```bash
# Search in specific repo
gh search issues "query" --repo owner/repo

# Search with filters
gh search issues "query" \
  --repo owner/repo \
  --state=open \
  --label=bug \
  --assignee=username \
  --limit=20

# JSON output
gh search issues "query" --json number,title,state,author,labels,url,createdAt

# All available JSON fields for issues:
# number, title, state, body, author, labels, assignees,
# url, createdAt, updatedAt, closedAt, comments
```

### View Issue

```bash
# View issue
gh issue view <number> --repo owner/repo

# View in browser
gh issue view <number> --repo owner/repo --web

# JSON output
gh issue view <number> --repo owner/repo --json number,title,body,state,comments
```

### List Issues

```bash
# List issues in current repo
gh issue list

# List with filters
gh issue list --repo owner/repo --state open --label bug --limit 20

# JSON output
gh issue list --repo owner/repo --json number,title,state,labels
```

### Create Issue

```bash
# Interactive create
gh issue create --repo owner/repo

# With options
gh issue create --repo owner/repo \
  --title "Issue title" \
  --body "Issue description" \
  --label bug,priority
```

## Pull Request Operations

### Search PRs

```bash
# Search PRs
gh search prs "query" --repo owner/repo

# With filters
gh search prs "query" \
  --repo owner/repo \
  --state=open \
  --draft=false \
  --label=enhancement \
  --limit=20

# JSON output
gh search prs "query" --json number,title,state,author,url,isDraft

# All available JSON fields for PRs:
# number, title, state, body, author, labels, assignees,
# url, createdAt, updatedAt, closedAt, mergedAt, isDraft,
# headRefName, baseRefName, additions, deletions, changedFiles
```

### View PR

```bash
# View PR
gh pr view <number> --repo owner/repo

# View in browser
gh pr view <number> --repo owner/repo --web

# View diff
gh pr diff <number> --repo owner/repo

# JSON output
gh pr view <number> --repo owner/repo --json number,title,body,state,commits,files
```

### List PRs

```bash
# List PRs
gh pr list --repo owner/repo

# With filters
gh pr list --repo owner/repo --state open --label feature --limit 20

# JSON output
gh pr list --repo owner/repo --json number,title,state,author,headRefName
```

### PR Comments

```bash
# View PR comments via API
gh api repos/owner/repo/pulls/<number>/comments

# View issue/PR comments
gh api repos/owner/repo/issues/<number>/comments
```

## API Access

```bash
# Generic API call
gh api <endpoint>

# With method
gh api -X POST <endpoint>

# With JSON body
gh api -X POST <endpoint> -f field=value

# Paginated results
gh api <endpoint> --paginate

# JQ filtering
gh api <endpoint> --jq '.[] | {name, url}'
```

## Common Patterns

### Get Repository Info

```bash
# Full repo details
gh repo view owner/repo --json name,description,url,stargazerCount,forkCount,isArchived,defaultBranchRef,languages,repositoryTopics
```

### Get Latest Release

```bash
gh release view --repo owner/repo
gh release list --repo owner/repo --limit 5
```

### Get Contributors

```bash
gh api repos/owner/repo/contributors --jq '.[].login'
```

### Get Languages

```bash
gh api repos/owner/repo/languages
```

## Output Formatting

```bash
# Table format (default)
gh search repos "query"

# JSON format
gh search repos "query" --json name,url

# Template format
gh search repos "query" --template '{{range .}}{{.name}}: {{.url}}{{"\n"}}{{end}}'
```

## Rate Limiting

```bash
# Check rate limit
gh api rate_limit

# View remaining requests
gh api rate_limit --jq '.resources.core.remaining'
```
