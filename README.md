# @pipeworx/virgin-islands-code

Virgin Islands Code (V.I.C.), the statutes of the US Virgin Islands, 2026
edition — full section text by citation, keyword search within a title, and
the list of all 37 titles. Keyless.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1866+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `vi_statute(citation)` — full text of a V.I.C. section by citation
  ("14 V.I.C. § 922" is first and second degree murder; "11A V.I.C. § 2-201"
  is the UCC statute of frauds). Also accepts "14 VIC 922",
  "V.I. Code Ann. tit. 14, § 922" and "Title 14, Section 922". Returns the
  heading, the statutory text and the "History:" amendment line.
- `vi_search(title, query, limit?)` — keyword search inside ONE title; every
  word must appear in a section's heading or text. Heading matches list first,
  then code order. Returns citations, headings and a snippet.
- `vi_titles()` — all 37 titles (1-34 plus 11A, 12A, 15A) with their official
  names and section counts, for routing a subject to a title.

A miss is an answer, not an error: an unknown title, a section the title does
not carry, or a search with no match returns `found: false` with a reason and
`data_as_of`. An upstream failure (non-404 status, timeout, a page that no
longer has the section markup) is a loud error, never `found: false`.

## Auth

Keyless.

## Data sources

- USVI Public Records (a VI Update project, usvipublicrecords.com), Virgin
  Islands Code 2026 edition — the archive's own text edition, set from the
  Virgin Islands Code 2026 Edition, all 37 titles and 12,420 sections.

## Why this source

The Virgin Islands was the one US jurisdiction without a code pack after DC
and the other four inhabited territories shipped (fleet #2888-#2892). The only
free copy of the current Code we had found sat behind a bot check, and the
commercial publisher bars automated access (fleet #2907). Bruce pointed at
USVI Public Records, which publishes the Code as readable text.

## Reuse terms

From the publisher's own pages, quoted:

- The Code page: *"This is the archive's own edition of a public text; the
  statutory text is an edict of government and belongs to no one."*
- USVI Public Records' terms page, on what the archive itself carries:
  *"U.S. government works (public domain), acts and edicts of the V.I.
  government (uncopyrightable by law) …"*
- Its robots policy: *"Search engines, AI answer engines and assistants acting
  for a person are all welcome"*, with `Content-Signal: search=yes,
  ai-input=yes, ai-train=yes`.

This pack fetches per request and copies nothing in bulk.

## Being a polite client

The publisher bills the website host per gigabyte and asks software not to
crawl it. The Code is only published there, one page per title (Title 14 is
~770 KB, Title 11A ~1.1 MB), so:

- every fetch asks Cloudflare to keep the title page at the edge for a day, so
  repeated lookups in the same title cost the publisher about one request a day;
- `vi_search` needs a title instead of fanning out across all 37 pages for
  every query.

If traffic grows enough that a full copy makes sense, ask the publisher first
(tips@viupdate.com is on their machine-access page). Don't start crawling.

## Edge probe

Checked 2026-10-12 from a throwaway `elias-vi-probe` Worker running under
`wrangler dev --remote`, sending this pack's User-Agent:

| Page | Status | Bytes |
|---|---|---|
| the Code's title index | 200 | 27,788 |
| Title 14 (Crimes) | 200 | 768,043 |

There's no challenge page and no block on Worker egress.

## Page shape, for whoever touches the parser

Each title page puts every section in an `<article class="sec" id="s-<n>">`
with the citation in `span.num`, the heading after it in the `h3`, and the
text plus a trailing "History:" line in `div.body`. Ids are the lowercased
section number and are unique per page (checked on Titles 14 and 11A): `922`,
`1508a`, and the UCC's article-dashed `2a-525`. There are no per-section
pages. A title that doesn't exist returns 404, which the pack reports as
`unknown_title`.

Some headings carry a stray "v2" from the source edition (for example,
14 V.I.C. § 151 "v2 Abortion Defined; When Authorized"). The pack returns them
exactly as published.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "virgin-islands-code": {
      "url": "https://gateway.pipeworx.io/virgin-islands-code/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/virgin-islands-code/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1866+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/vi_statute \
  -H 'Content-Type: application/json' \
  -d '{"citation":"14 V.I.C. § 922"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/vi_statute`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-virgin-islands-code` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/virgin-islands-code/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "virgin-islands-code": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-virgin-islands-code"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Virgin Islands Code data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
