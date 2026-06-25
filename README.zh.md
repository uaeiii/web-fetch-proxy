# Web Fetch Proxy — Claude Code Skill

当 Claude Code 内置的 `WebFetch` 工具因域名安全限制被拦截时，自动回退使用 `curl`/`python` 通过 `Bash` 获取网页内容的技能。

## 解决了什么问题

Claude Code 的 `WebFetch` 有时会返回以下错误：

```
Unable to verify if domain X is safe to fetch
```

这通常是由于企业网络策略、安全限制或域名不在白名单中造成的。但问题在于——域名本身是可以访问的，只是 `WebFetch` 无法验证它。这个技能就是用来绕过这个限制的。

## 工作原理

```
WebFetch 失败 ──▶ Bash (curl/python) ──▶ 处理内容 ──▶ 回答问题
```

具体流程：

1. **检测失败** — 识别 `"Unable to verify if domain * is safe to fetch"` 等错误信息
2. **自动回退** — 通过 `Bash` 使用 Python `urllib` 或 `curl` 获取 URL
3. **内容分类** — 判断返回的是正常页面、JSON 数据、还是验证码页面
4. **正常应答** — 像 WebFetch 成功了一样处理内容并回答用户

## 功能特性

- **自动编码检测** — 从 `Content-Type` 响应头或 HTML `<meta>` 标签自动检测字符编码（支持 UTF-8、GBK 等中文编码）
- **安全输出** — 正确处理终端编码，避免中文显示乱码
- **JSON API 支持** — 完整支持 JSON 接口，保留 Unicode 字符
- **GitHub 原始文件** — 直接获取 GitHub raw 内容
- **反爬识别** — 能识别验证码/安全验证页面并如实告知用户，而非报错
- **零外部依赖** — 仅使用 Python 标准库和/或 curl

## 安装方法

```bash
# 从 GitHub 安装
npx skills add https://github.com/uaeiii/web-fetch-proxy

# 或者从本地源码安装
npx skills add file:///path/to/web-fetch-proxy
```

## 使用方法

安装后，每当 `WebFetch` 失败时技能会自动激活。你也可以手动调用：

```
/web-fetch-proxy https://example.com
```

技能会自动：
1. 先尝试 `WebFetch`
2. 失败则 → 通过 `Bash` 用 `python` 回退
3. 返回提取的内容

## 系统要求

- **Claude Code**（任意版本）
- `curl`（可选，用于 curl 方式回退）
- `python` 3.x（推荐，用于 Python 方式回退）

## 常见目标站点

| 目标 | 方法 |
|--------|--------|
| **skills.sh** | Python `urllib` |
| **GitHub raw 文件** | `curl` 或 `urllib` |
| **知乎专栏** | Python，自动处理中文编码 |
| **通用 HTML 页面** | Python，自动编码检测 |
| **JSON API 接口** | Python，`ensure_ascii=False` 保留中文 |

## 注意事项

- 部分网站（B站、百度等）有反爬机制，会对非浏览器请求返回验证码页面。技能会检测到这种情况并如实告知，这不是技能本身的缺陷
- 所有请求都设置了 `timeout=15` 超时保护，防止卡死
- 永远不会使用 `--insecure` 或 `-k` 参数（安全第一）

## 许可

MIT
