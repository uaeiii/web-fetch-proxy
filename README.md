# Web Fetch Proxy — Claude Code Skill

A **Claude Code** skill that automatically falls back to `curl`/`python` via `Bash` when the built-in `WebFetch` tool fails due to domain security restrictions.

## Problem

Claude Code's built-in `WebFetch` tool sometimes returns:

```
Unable to verify if domain X is safe to fetch
```

This typically happens due to enterprise network policies, security restrictions, or the domain not being on an allowlist. The domain is still reachable — `WebFetch` just can't verify it.

## Solution

This skill intercepts those failures and automatically falls back to fetching content using `curl` or `python` through the `Bash` tool, which bypasses the WebFetch restriction layer entirely.

## How It Works

```
WebFetch fails ──▶ Bash (curl/python) ──▶ Process content ──▶ Answer user
```

The skill follows this flow:

1. **Detect failure** — Matches error patterns like `"Unable to verify if domain * is safe to fetch"`
2. **Fall back** — Uses Python `urllib` or `curl` via `Bash` to fetch the URL
3. **Classify response** — Determines if the response is a normal page, JSON API, or captcha/verification page
4. **Answer normally** — Processes the content as if WebFetch had succeeded

## Features

- **Automatic encoding detection** — Detects charset from `Content-Type` header or HTML `<meta>` tag (supports UTF-8, GBK, and more)
- **Safe output** — Properly handles terminal encoding to avoid display errors
- **JSON API support** — Full JSON fetching with `json.dumps(..., ensure_ascii=False)` for Unicode
- **GitHub raw file support** — Direct GitHub raw content fetching
- **Anti-bot detection** — Recognizes captcha/verification pages and reports them honestly
- **No external dependencies** — Uses only Python standard library and/or curl

## Installation

```bash
# From GitHub
npx skills add https://github.com/uaeiii/web-fetch-proxy

# Or from local source
npx skills add file:///path/to/web-fetch-proxy
```

## Usage

Once installed, the skill activates automatically whenever `WebFetch` fails. You can also invoke it manually:

```
/web-fetch-proxy https://example.com
```

The skill will:
1. Try `WebFetch` first
2. If it fails → fall back to `python` via `Bash`
3. Return the extracted content

## Requirements

- **Claude Code** (any version)
- `curl` (optional, for curl-based fallback)
- `python` 3.x (for Python-based fallback, recommended)

## Common Targets

| Target | Method |
|--------|--------|
| **skills.sh** | Python `urllib` |
| **GitHub raw files** | `curl` or `urllib` |
| **zhuanlan.zhihu.com** | Python with Chinese encoding |
| **Generic HTML** | Python with auto charset detection |
| **JSON APIs** | Python with `json.dumps` |

## Notes

- Some sites (Bilibili, Baidu, etc.) have anti-bot protection that returns captcha pages to non-browser requests. This is detected and reported, not a bug in the skill.
- Always sets `timeout=15` to prevent hanging.
- Never uses `--insecure` or `-k` flags (security first).
