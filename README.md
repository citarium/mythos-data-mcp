# MythosData — MCP server

> Comparative mythology as citable data — Greek, Roman, Norse and Egyptian gods and primary sources

> **Status (2026-09-04): deployed, domain propagating.** The instance is
> live on Vercel — `citarium verify . --remote=<deployment>` passes 12
> contract checks over 204 pages, and every surface below (MCP, `llms.txt`,
> `ai-index.json`, `server.json`, sitemap) answers 200. `mythosdata.dev` is
> attached to the deployment and its DNS resolves; the endpoint below has
> not yet been confirmed answering from outside Vercel's edge. This note
> goes away once that confirmation lands.

A **remote MCP server** over a curated knowledge graph of comparative
mythology. Every claim it returns is bound to a passage of a primary text
— Hesiod, Homer, the Homeric Hymns, Apollodorus, Ovid, Virgil, Livy,
Herodotus, Plutarch, the Poetic Edda, Budge's Egyptian anthology — with the
line, section or stanza, so an agent can quote the witness instead of an
encyclopaedia, and say *according to whom*.

Nothing to install. It is a hosted streamable-HTTP endpoint:

```
https://mythosdata.dev/mcp
```

## Add it to a client

**Claude Code**

```bash
claude mcp add --transport http mythosdata https://mythosdata.dev/mcp
```

**Claude Desktop / any client reading `mcpServers`**

```json
{
  "mcpServers": {
    "mythosdata": {
      "type": "streamable-http",
      "url": "https://mythosdata.dev/mcp"
    }
  }
}
```

No API key, no account, no auth. Read-only.

**Check it answers, without any client at all:**

```bash
curl -s https://mythosdata.dev/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -H 'mcp-protocol-version: 2025-06-18' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## Tools

Eight, each with an `outputSchema`, each returning `structuredContent`.
(A ninth, `compare` — two to six objects side by side with their facets,
shared relations and claims — ships with the next framework release; this
corpus is built for it.)

| tool | arguments | what it does |
|---|---|---|
| `get_overview` | — | Corpus overview: what this instance knows, counts by type, published tags, freshness. **Start here** when you land and do not yet know whether this corpus can answer your question. |
| `search` | `query`, `limit?` | Full-text search over the knowledge graph. Accent- and apostrophe-insensitive, so query in the user's own words; every hit carries its relevance score and the fields it matched. |
| `answer` | `question` | Answer a question from the corpus. Returns the matched object's claims with sources and confidence — **never an unsourced answer**. |
| `get_entity` | `id` | Fetch one knowledge object by id, with its claims and the sources each claim cites. |
| `get_topic` | `tag` | List the knowledge objects carrying a tag (topics are content-backed tags). |
| `get_related` | `id` | Graph neighbours of an object: outgoing and incoming relations, each with its relation type — `equivalent_of` links the pantheons (Zeus ↔ Jupiter, Amun ↔ Zeus per Herodotus, Set ↔ Typhon). |
| `get_sources` | `object_id?` | The whole source registry, or just the sources cited by one object. Use it to judge the corpus before trusting it. |
| `get_latest` | `limit?` | Most recently verified knowledge objects — a freshness signal. |

The intended path is `get_overview` → `search` or `answer` → `get_entity`
→ `get_related`.

## What is in the corpus

| | |
|---|---|
| knowledge objects | **201** |
| registered sources | **44** |
| published topics | **239** |

Four traditions — Greek, Roman, Norse, Egyptian — as deities, Titans and
primordials, heroes, creatures, places, events and the texts themselves;
Greek↔Roman pairs and cross-pantheon comparisons (creation, flood and
end-of-the-world myths, underworlds, thunder gods, tricksters, sun, love,
war, wisdom and mother goddesses); guides (the Twelve Olympians, family
trees, the Greek–Roman equivalence table, which primary source tells which
myth, how to cite them); a glossary; FAQs. Every object carries indexed
facets — `pantheon`, `kind`, `group`, `domains`, `symbols`, `wikidata` —
in `api/index.json`, so the whole corpus can be filtered by pantheon in
one request.

### Questions it is built to answer

- *Where was Zeus born according to Hesiod, and what does Apollodorus say instead?*
- *Is Jupiter the same god as Zeus?* — the same myths, a different cult.
- *Which Greek god did Herodotus identify with Amun?*
- *How does Thor die at Ragnarök, stanza by stanza?*

## Machine-readable surfaces

| surface | what it is |
|---|---|
| [`/llms.txt`](https://mythosdata.dev/llms.txt) | the index, as `text/plain` |
| [`/llms-full.txt`](https://mythosdata.dev/llms-full.txt) | the whole corpus in one file |
| [`/ai-index.json`](https://mythosdata.dev/ai-index.json) | every surface this instance publishes, with its content type |
| [`/api/index.json`](https://mythosdata.dev/api/index.json) | one JSON document per knowledge object, with the indexed facets |
| [`/api/sources.json`](https://mythosdata.dev/api/sources.json) | the source registry, in full |
| [`/.well-known/mcp/server.json`](https://mythosdata.dev/.well-known/mcp/server.json) | this server's manifest |

Each knowledge object has a human page and a machine twin at the same id,
with a canonical URL that agrees across all of them.

## Behaviour worth knowing before you integrate

- **`POST` only.** Every other method answers `405` with an `Allow: POST, OPTIONS` header.
- **Rate limit:** 120 requests per minute per client, counted in a shared
  store, published on every response as `RateLimit-Limit`,
  `RateLimit-Remaining` and `RateLimit-Reset` (all three exposed via CORS).
  It fails **open**: if the store is unreachable the request is served.
- **Malformed input** gets a spec-correct JSON-RPC error — `-32700` for
  unparseable bodies, `-32602` for an unknown tool — never an HTML error page.
- **Request bodies are capped** and validated before transport.

## Privacy

No accounts, no cookies, no ads. Usage is measured in aggregate with
daily-rotating hashed identifiers and a 200-day retention; raw IPs are
never stored. Full policy: [PRIVACY.md](./PRIVACY.md).

## Provenance and licence

Knowledge content is **CC-BY-4.0**: use it, cite it. The primary texts were
read in the Perseus Digital Library's public-domain translations
(CC BY-SA 3.0), the Bellows Poetic Edda (1923) and Budge (1914), and every
claim names the passage. Wikipedia-derived objects are tiered `derived` and
kept under a quarter of the graph. There is no `secondary` tier: the
authoring environment could reach primary texts and Wikipedia and nothing
in between, and the corpus says so rather than pretending otherwise.

## How it is built

Compiled and served by Citarium, a source-available framework for turning a
knowledge graph into a website, an API, an MCP server and agent-readable
files from a single source, under external evaluation.

The framework's code is licensed under the Business Source License 1.1 and
its repository is not public. What is public — and what actually matters for
trusting an answer — is this server, the corpus it serves, and the registered
source behind every claim: `get_sources` returns what any given claim rests
on, so it can be checked rather than trusted.

This repository is the server's public face: its manifest and its
documentation. The corpus itself lives at [mythosdata.dev](https://mythosdata.dev).
