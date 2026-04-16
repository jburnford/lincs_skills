# LINCS Skills for Claude Code

Claude Code skills for working with CIDOC-CRM ontology, LINCS (Linked Infrastructure for Networked Cultural Scholarship), and Wikidata entity disambiguation.

## Skills

| Skill | Slash Command | Description |
|-------|---------------|-------------|
| [cidoc-crm.md](cidoc-crm.md) | `/cidoc-crm` | CIDOC-CRM v7.1.1 ontology reference — classes, properties, CRMgeo/CRMdig extensions, modeling decision guide, anti-patterns |
| [lincs-profile.md](lincs-profile.md) | `/lincs-profile` | LINCS application profile — 8 reusable patterns, person/organization/government/document templates from all 3 History of Canada datasets |
| [lincs-validate.md](lincs-validate.md) | `/lincs-validate` | Model validator — 9-category checklist producing PASS/WARN/FAIL report against LINCS requirements |
| [neo4j-to-rdf.md](neo4j-to-rdf.md) | `/neo4j-to-rdf` | Neo4j → RDF/Turtle converter — 8 translation rules for namespace mapping, relationship property reification, authority URI insertion |
| [lincs-sparql.md](lincs-sparql.md) | `/lincs-sparql` | SPARQL query builder — 10 templates for the LINCS Fuseki endpoint plus local census data queries and LINCS↔census bridging queries |
| [person-disambig.md](person-disambig.md) | `/person-disambig` | Wikidata person disambiguation — match person NER clusters to Wikidata QIDs using the Wikidata MCP server |
| [place-disambig.md](place-disambig.md) | `/place-disambig` | Wikidata place disambiguation — ground geographic entities, toponyms, and colonial place names to Wikidata QIDs |
| [canadian-place-grounding.md](canadian-place-grounding.md) | `/canadian-place-grounding` | Canadian place Wikidata grounding — CSDs, townships, parishes, First Nations reserves, rural municipalities with province-specific search strategies |

## Prerequisites: Wikidata MCP Server

The disambiguation skills (`person-disambig`, `place-disambig`, `canadian-place-grounding`) require the [official Wikidata MCP server](https://www.wikidata.org/wiki/Wikidata:MCP) for semantic entity search. No API key required.

- **Endpoint**: `https://wd-mcp.wmcloud.org/mcp/`
- **Documentation**: https://www.wikidata.org/wiki/Wikidata:MCP
- **Test UI**: https://wd-mcp.wmcloud.org/docs
- **Hosted by**: Wikimedia Cloud Services

### Setup

1. **Add the MCP server** from your project directory:

```bash
claude mcp add --transport http wikidata https://wd-mcp.wmcloud.org/mcp/
```

This writes the config to `.claude.json`:

```json
{
  "mcpServers": {
    "wikidata": {
      "type": "http",
      "url": "https://wd-mcp.wmcloud.org/mcp/"
    }
  }
}
```

2. **Restart Claude Code** (the MCP server is discovered at startup):

```
/exit
claude
```

3. **Allow the MCP tools** in your project settings (`.claude/settings.local.json`):

```json
{
  "permissions": {
    "allow": [
      "mcp__wikidata__search_items",
      "mcp__wikidata__get_statements",
      "mcp__wikidata__get_statement_values",
      "mcp__wikidata__execute_sparql",
      "mcp__wikidata__search_properties",
      "mcp__wikidata__get_instance_and_subclass_hierarchy"
    ]
  }
}
```

### Available MCP Tools

| Tool | Description |
|------|-------------|
| `mcp__wikidata__search_items(query)` | Hybrid keyword/vector search for Wikidata items |
| `mcp__wikidata__search_properties(query)` | Search for Wikidata properties |
| `mcp__wikidata__get_statements(item_id)` | Get all statements (claims) for an entity |
| `mcp__wikidata__get_statement_values(item_id, property_id)` | Get values for a specific property |
| `mcp__wikidata__get_instance_and_subclass_hierarchy(item_id)` | Get type hierarchy |
| `mcp__wikidata__execute_sparql(query)` | Run SPARQL queries against Wikidata |

### Tips

- **Short, focused queries work best**: "Frontenac County Ontario" not "Frontenac County census division in Ontario province Canada"
- **The server uses semantic vector search**, so it handles synonyms and context better than the Wikidata REST API (`wbsearchentities`)
- **No API key needed** — the server is public and hosted by Wikimedia
- **Rate limit**: ~5 requests/minute. Space batches accordingly for large disambiguation jobs
- **Send 6-8 parallel queries per batch** for efficiency
- If a query returns empty, try shorter phrasing or retry

### Comparison with Other Approaches

| Method | Rate Limits | Quality | Auth Required |
|--------|-------------|---------|---------------|
| **Wikidata MCP** (recommended) | ~5/min | Semantic search, best results | No |
| Wikidata REST API (`wbsearchentities`) | Moderate | String similarity only, poor for disambiguation | No |
| Wikidata SPARQL (`query.wikidata.org`) | Aggressive IP blocking | Structured queries | No |
| QLever (`qlever.dev/api/wikidata`) | Generous | Good for bulk SPARQL | No |

**Recommendation**: Use MCP for interactive entity disambiguation. Use QLever for bulk SPARQL (batch operations with >50 queries).

## Usage

### As Claude Code slash commands
```
/cidoc-crm E93_Presence
/lincs-profile how to model an occupation
/lincs-validate my-model.ttl
/neo4j-to-rdf Westmeath 1871 census data
/lincs-sparql find all people born in Saskatchewan
/person-disambig 100
/person-disambig status
/place-disambig 50
/place-disambig verify
```

### As reference documents
Each file is self-contained markdown with tables, Turtle examples, and decision guides. Read them directly in any editor or browser.

## Sources

Built from analysis of:
- [LINCS Historical Canadian Persons](https://lincsproject.ca/docs/explore-lod/project-datasets/hist-cdns/)
- [LINCS Historical Indian Affairs Agents](https://lincsproject.ca/docs/explore-lod/project-datasets/ind-affairs/)
- [LINCS Cabinet Conclusions](https://lincsproject.ca/docs/explore-lod/project-datasets/cabinet-conclusions/)
- CIDOC-CRM v7.1.1 specification (ISO 21127)
- CRMgeo geospatial extension
- CRMdig digital extension
- Canadian Historical GIS (CHGIS) Temporal Census Polygons project
