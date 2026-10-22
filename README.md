# Site Scan

A skill that scans a website the way Googlebot and the AI crawlers see it, scores it 0 to 100 across 22 checks, and orders the fixes. One SKILL.md in the open Agent Skills format, so it works in Claude Code, Codex, Cursor, Gemini CLI, GitHub Copilot and 70+ other agents.

The scan runs on Growthr's scanner, the same one behind [growthr.com/scan](https://growthr.com/scan/) and the `growthr_scan` tool on Growthr's MCP server (`https://mcp.growthr.com/mcp`). It has to, because the scanner fetches from a datacenter IP without JavaScript or cookies, which is how GPTBot, ClaudeBot, PerplexityBot and Googlebot arrive. A fetch from your own computer renders JavaScript and carries your cookies, so it passes sites the crawlers cannot read.

## What it checks

- Reachability as a browser, as GPTBot and as ClaudeBot
- Real content without JavaScript
- Metadata (lang, canonical, og:image, og:type), strict JSON-LD, Organization schema
- robots.txt access for AI crawlers, llms.txt and a link to it, sitemap with lastmod dates
- Honest 404s, `Accept: text/markdown` negotiation, trust pages (/about, /contact, /privacy)
- Homepage speed, and on-page SEO: title, description, one h1, viewport, alt text, not noindexed

Each check is weighted. The heavy ones are the blockers: if the homepage is unreachable from a datacenter or has no server-rendered text, the rest of the score is noise until that is fixed.

## What you get back

The score, every failed check with what the scanner found and a fix hint, and the work in four buckets: blockers, this afternoon, needs a sprint, and the off-site work no scan can measure (reviews, directories, mentions), which is what decides whether AI engines recommend a business.

## Install

### Claude Code, Codex, Cursor, Gemini CLI and other agents

```
npx skills add Growthr/site-scan
```

### Claude.ai

1. Download [site-scan.zip](https://github.com/Growthr/site-scan/releases/latest/download/site-scan.zip).
2. Turn on code execution in Settings > Capabilities.
3. Go to Customize > Skills, click +, then Create skill > Upload a skill, and choose the zip.

Or add Growthr's MCP server as a connector (`https://mcp.growthr.com/mcp`) and ask Claude to scan a site; no skill needed.

### Claude Code plugin

```
/plugin marketplace add growthr/no-ai-writing-slop
/plugin install site-scan@growthr
```

### Any other AI tool

Paste [SKILL.md](skills/site-scan/SKILL.md) into the tool's custom instructions, or into the chat before you ask.

## Using it

"Scan example.com" or "can AI crawlers read my site?" The skill calls `growthr_scan` if the MCP is connected, and otherwise calls the same server over HTTPS with curl. No key, no signup. A scan reads about a dozen public URLs once, takes up to 20 seconds, and is limited to three per minute. Growthr logs the domain and tool name for usage stats and stores nothing else.

Missing an llms.txt? The [llms-txt-generator](https://github.com/Growthr/llms-txt-generator) skill drafts one.

## License

MIT. See [LICENSE](LICENSE).

Made by [Growthr](https://growthr.com/scan/).
