---
name: search-web
description: "Conduct comprehensive web research and fetch up-to-date library documentation. Use when asked to research topics, investigate trends, search the web, scrape URLs, fetch web content, or when needing current documentation, code examples, API references, or setup guides for any programming library or framework. Combines web search, content extraction, Context7 documentation, and synthesis."
allowed-tools: ["Bash", "Read", "Write"]
argument-hint: "[topic, library name, or URL]"
---

# Web Research & Documentation Skill

Comprehensive research combining web search, content extraction, and up-to-date library documentation via Context7.

## Tools Used

- **WebSearch** - Search the web for current information
- **WebFetch** - Extract content from specific URLs as markdown
- **mcp__context7__resolve-library-id** - Find Context7 library ID for a package
- **mcp__context7__query-docs** - Query documentation with a specific topic

## Known Library IDs

For common library IDs (Quarkus, HTMX, PostgreSQL, etc.), see `library-ids.md` in this skill directory. Use these IDs directly to skip the resolve step.

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
