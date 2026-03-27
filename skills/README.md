# LINCS Skills for Claude Code

Claude Code skills for working with CIDOC-CRM ontology, LINCS (Linked Infrastructure for Networked Cultural Scholarship), and Wikidata entity disambiguation.

## Skills

| Skill | Slash Command | Description |
|-------|---------------|-------------|
| [cidoc-crm.md](cidoc-crm.md) | `/cidoc-crm` | CIDOC-CRM v7.3.1 ontology reference — classes, properties, CRMgeo/CRMdig extensions, modeling decision guide, anti-patterns |
| [lincs-profile.md](lincs-profile.md) | `/lincs-profile` | LINCS application profile — 8 reusable patterns, person/organization/government/document templates from all 3 History of Canada datasets |
| [lincs-validate.md](lincs-validate.md) | `/lincs-validate` | Model validator — 9-category checklist producing PASS/WARN/FAIL report against LINCS requirements |
| [cidoc-to-rdf.md](cidoc-to-rdf.md) | `/cidoc-to-rdf` | Neo4j → RDF/Turtle converter — 8 translation rules for namespace mapping, relationship property reification, authority URI insertion |
| [lincs-sparql.md](lincs-sparql.md) | `/lincs-sparql` | SPARQL query builder — 10 templates for the LINCS Fuseki endpoint plus local census data queries and LINCS↔census bridging queries |
| [person-disambig.md](person-disambig.md) | `/person-disambig` | Wikidata entity disambiguation — match person NER clusters to Wikidata QIDs using the Wikidata MCP server |

## Prerequisites: Wikidata MCP Server

The `person-disambig` skill requires the [mcp-wikidata](https://github.com/zzaebok/mcp-wikidata) MCP server for searching Wikidata entities.

### Setup

1. **Get a Smithery API key** from [smithery.ai](https://smithery.ai)

2. **Add the MCP server** to your Claude Code settings (`~/.claude/settings.json` or project-level `.claude/settings.local.json`):

```json
{
  "mcpServers": {
    "mcp-wikidata": {
      "command": "npx",
      "args": [
        "-y",
        "@smithery/cli@latest",
        "run",
        "@zzaebok/mcp-wikidata",
        "--key",
        "YOUR_SMITHERY_KEY"
      ]
    }
  }
}
```

3. **Allow the MCP tools** in your project settings (`.claude/settings.local.json`):

```json
{
  "permissions": {
    "allow": [
      "mcp__wikidata__search_items",
      "mcp__wikidata__get_statements"
    ]
  }
}
```

### Available MCP Tools

| Tool | Description |
|------|-------------|
| `search_entity(query)` | Find Wikidata entity IDs by name |
| `search_property(query)` | Find Wikidata property IDs |
| `get_properties(entity_id)` | Get all properties for an entity |
| `execute_sparql(sparql_query)` | Run SPARQL queries against Wikidata |
| `get_metadata(entity_id, language)` | Get label and description for an entity |

### Tips

- **Short, exact name queries work best**: "John Locke" not "John Locke English philosopher empiricism"
- **The server is intermittent**: if a query returns empty, try shorter phrasing or retry
- **Send 8-10 parallel queries per batch** for efficiency, but space batches if results start coming back empty

## Usage

### As Claude Code slash commands
```
/cidoc-crm E93_Presence
/lincs-profile how to model an occupation
/lincs-validate my-model.ttl
/cidoc-to-rdf Westmeath 1871 census data
/lincs-sparql find all people born in Saskatchewan
/person-disambig 100
/person-disambig status
```

### As reference documents
Each file is self-contained markdown with tables, Turtle examples, and decision guides. Read them directly in any editor or browser.

## Sources

Built from analysis of:
- [LINCS Historical Canadian Persons](https://lincsproject.ca/docs/explore-lod/project-datasets/hist-cdns/)
- [LINCS Historical Indian Affairs Agents](https://lincsproject.ca/docs/explore-lod/project-datasets/ind-affairs/)
- [LINCS Cabinet Conclusions](https://lincsproject.ca/docs/explore-lod/project-datasets/cabinet-conclusions/)
- CIDOC-CRM v7.3.1 specification (ISO 21127)
- CRMgeo geospatial extension
- CRMdig digital extension
- Canadian Historical GIS (CHGIS) Temporal Census Polygons project
