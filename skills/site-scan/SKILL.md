---
name: site-scan
description: Scan a public website the way Googlebot and the AI crawlers see it (datacenter IP, no JavaScript, no cookies) and score it 0-100 across 22 weighted checks for SEO and AI readability, then order the fixes. Use when the user asks to "scan my site", "audit SEO", "check AI readiness", "can ChatGPT or Perplexity read my site", "why isn't AI citing us", or wants a GEO or AEO check. Runs through Growthr's scanner, as the MCP tool growthr_scan when connected or over HTTPS otherwise.
license: MIT
metadata:
  author: Growthr
  version: "1.0.0"
  notice: |
    Copyright (c) 2026 Growthr, github.com/Growthr/site-scan. MIT License.
---

# Site scan

Run Growthr's scanner against a domain and turn the result into a prioritized fix list. The scan must run from Growthr's server, not from this machine: it fetches from a datacenter IP without JavaScript, which is how GPTBot, ClaudeBot, PerplexityBot and Googlebot arrive. A fetch from the user's own computer hides firewall blocks and renders JavaScript, so it would pass sites the crawlers cannot read.

## 1. Run the scan

Take the bare domain (`example.com`, no scheme, no path). Only public sites; the scanner refuses localhost and private hosts.

**If the Growthr MCP is connected:** call `growthr_scan` with `{ "domain": "example.com" }`. For an ordered plan, call `growthr_fix_order` with the same input; it runs the scan and sorts failures into buckets.

**Otherwise, call the same server over HTTPS** (no key needed):

```bash
curl -s -m 90 -X POST https://mcp.growthr.com/mcp \
  -H 'content-type: application/json' \
  -H 'accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"growthr_scan","arguments":{"domain":"example.com"}}}'
```

The reply is JSON-RPC. `result.structuredContent` holds `domain`, `score` (0-100), `checks` (each with `id`, `label`, `pass`, `detail`, `weight`), and `throttledCount` (how many requests the site answered with HTTP 429). `result.content[0].text` is the same thing as readable text. Use `growthr_fix_order` in the `name` field for the bucketed version.

A scan reads about a dozen public URLs once and takes up to 20 seconds. The limit is three scans per minute per caller. Growthr logs the domain and tool name for usage stats and stores nothing else; say so if the user asks what the scan sends.

## 2. What the 22 checks cover

Reachability as a browser, as GPTBot, and as ClaudeBot (weights 10, 7, 7). Real content without JavaScript (12). Metadata: lang, canonical, og:image, og:type (8). JSON-LD present and strictly valid (8). Organization schema with contact, address, sameAs (7). robots.txt lets AI crawlers in (8). llms.txt exists (8) and is linked in the page (5). sitemap.xml exists with URLs (6) and lastmod dates (3). Missing pages return a real 404 (8). `Accept: text/markdown` negotiation (4). Trust pages: /about, /contact, /privacy (6). Homepage answers within 3 seconds (4). On-page SEO: title 15-65 characters (6), description 50-165 (6), exactly one h1 (5), not noindexed (6), viewport meta (3), image alt text (4).

The heaviest checks are the ones that hide everything else: if the homepage is unreachable from a datacenter or has no server-rendered text, the rest of the score is noise until that is fixed.

## 3. Report it

Lead with the score and the failed checks, each with what the scanner found and the fix hint it returned. Then order the work:

1. **Blockers.** Reachability failures and no content without JavaScript. Nothing downstream counts until these pass.
2. **This afternoon.** Metadata, llms.txt (the `llms-txt-generator` skill or `growthr_llms_txt` drafts one), trust pages, on-page SEO.
3. **Needs a sprint.** Schema, markdown negotiation, speed.
4. **Off-site, which no scan measures.** Reviews, directories, third-party mentions. These decide whether AI engines recommend a business, so say it even though the scan cannot score it.

Quote the scanner's findings; do not add failures it did not report or guess at causes it did not observe. If `throttledCount` is above zero, or reachability failed with 403 or 429, say the site (or its CDN) is rate-limiting or blocking datacenter traffic and that the result understates the site; a block like that also hits the real crawlers, so it is a finding, not noise.

## Limits, said plainly

The score measures whether crawlers can read the site and find the basics. It does not measure rankings, traffic, or whether an AI answer names the business. A site can score 100 and still never be recommended; third-party corroboration does that work. Say this when the user treats the score as the goal.
