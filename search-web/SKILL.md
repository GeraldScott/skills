---
name: search-web
description: "Search the web, research topics, fetch URLs, look up library docs, or find current information. Use when the user asks to research, search, look up, investigate, find out about, or get documentation for anything — including libraries, frameworks, APIs, trends, best practices, tutorials, or any topic requiring up-to-date information. Also use when the user provides a URL to scrape or asks about recent developments."
allowed-tools: ["WebSearch", "WebFetch", "Read", "Write", "Bash", "mcp__context7__resolve-library-id", "mcp__context7__query-docs"]
argument-hint: "[topic, library name, or URL]"
context: fork
---

# Web Research & Documentation Skill

Comprehensive research combining web search, content extraction, and up-to-date library documentation via Context7.

## Source Priority

Use the most authoritative source available, in this order:

1. **Context7 docs** — most current, structured library documentation
2. **WebFetch on official docs** — when you know the URL (e.g., official project site)
3. **WebSearch** — general web results, community discussions, blog posts

## Known Library IDs

For common library IDs (Quarkus, HTMX, PostgreSQL, etc.), see `library-ids.md` in this skill directory. Use these IDs directly to skip the resolve step.

## Research Discipline

**Write after searching.** Never do two search rounds in a row without capturing findings first. After each search or fetch, write key findings to your response or a scratch note before searching again. This prevents research loops and ensures nothing is lost.

---

## Research Workflows

### General web research
1. Use `WebSearch` with specific query terms
2. Use `WebFetch` to extract full content from top 2-3 URLs
3. Synthesize findings with source citations

### Library/framework documentation
1. Use `mcp__context7__resolve-library-id` to find the library ID (or use known ID from `library-ids.md`)
2. Use `mcp__context7__query-docs` with the library ID and focused topic query
3. Combine with `WebSearch` for recent updates or community discussions

### Comprehensive technical research
1. Fetch official docs via Context7
2. Search web for recent blog posts, tutorials, Stack Overflow
3. Synthesize into actionable guidance with code examples

---

## Examples

**Example 1: Library documentation**
```
User: "How do I set up authentication in Quarkus?"
→ mcp__context7__query-docs with libraryId="/quarkusio/quarkus", query="authentication setup security"
→ Return code examples with source attribution
```

**Example 2: General research**
```
User: "What are the best practices for PostgreSQL indexing?"
→ WebSearch "PostgreSQL indexing best practices 2026"
→ WebFetch top 2-3 relevant URLs
→ Synthesize findings with citations
```

**Example 3: Unknown library**
```
User: "How do I use Tanstack Query?"
→ mcp__context7__resolve-library-id with libraryName="Tanstack Query"
→ mcp__context7__query-docs with returned ID and user's topic
→ Return documentation with examples
```

---

## Output Guidelines

- Always include source URLs as citations
- Summarize key findings first, then provide details
- Note when information conflicts between sources
- Include dates when available to indicate freshness
- For code examples, prefer Context7 docs (most current)

## Error Handling

- If Context7 returns no results → fall back to WebSearch
- If a URL fails to fetch → try alternative sources from search results
- If library ID not found → try alternate library names or search web directly
- Always verify library IDs exist before querying docs (use resolve first for unknown libraries)
