---
name: place-disambig
description: Disambiguate place/toponym entities to Wikidata QIDs using MCP search. Use when grounding geographic entities from NER output, historical documents, or knowledge graphs to Wikidata.
user-invocable: true
allowed-tools: Read, Grep, Glob, Bash, Edit, Write, mcp__wikidata__search_items, mcp__wikidata__get_statements, mcp__wikidata__execute_sparql
argument-hint: [count-to-add | "status" | "audit" | "verify"]
---

# Place Disambiguation Workflow

You are disambiguating place/toponym entities to Wikidata QIDs using the Wikidata MCP server. This skill handles geographic entities from NER output, historical documents, knowledge graphs, and colonial/historical place names.

## Overview

Place disambiguation is harder than person disambiguation because:
- **Colonial vs. modern names**: "Ceylon" (Q854) vs "Sri Lanka" (Q854) — same entity, different eras
- **Historical political entities**: "Colony of Sierra Leone 1808-1896" (Q30059027) is NOT modern Sierra Leone (Q1044)
- **Namesakes**: "London" could be London GB (Q84), London Ontario (Q92561), or dozens of others
- **Ambiguous short forms**: "Georgia" (US state? Country? Colony?)
- **Hierarchical places**: "Bengal" could be the region, the province, or the presidency

## Input Formats

The skill handles several input formats depending on the project:

### JSONL Cluster Format (NER output)
```json
{"cluster_id": "london", "label": "London", "variants": ["london", "lond.", "lond"], "total_mentions": 2847, "match_type": "none"}
```

### JSON Node Format (Knowledge graph)
```json
{"colony_id": "sierra_leone_colony_1808_1896", "name": "Sierra Leone Colony", "region": "Africa", "established_year": 1808, "qid": "Q30059027", "verified": false}
```

### Simple List Format
One place name per line, or CSV with name column.

## MCP Tools

Use these tools EXCLUSIVELY for QID lookup. **Never invent or guess QIDs.**

| Tool | Use For |
|------|---------|
| `mcp__wikidata__search_items(query)` | Find QIDs by place name |
| `mcp__wikidata__get_statements(qid)` | Verify a QID — check P31 (instance of), P17 (country), P625 (coordinates), P571/P576 (inception/dissolved) |
| `mcp__wikidata__execute_sparql(query)` | Complex lookups — e.g., find all colonies of a specific empire |

### MCP Search Tips
- **Short, exact name queries work best**: "Sierra Leone" not "Sierra Leone British colony West Africa"
- **The server is intermittent**: if a query returns empty, try shorter phrasing or retry
- **Add one disambiguator if needed**: "Georgia country" vs "Georgia US state"
- **For historical entities**, search the historical name: "Colony of Sierra Leone", "Crown Colony of Malta"
- **Send 8-10 parallel queries per batch** for efficiency, but space batches if results start coming back empty

## Commands

If `$ARGUMENTS` is "status":
- Count matched vs unmatched places in the output file
- Report progress and remaining

If `$ARGUMENTS` is "audit":
- Pick 20 random matches from the output file
- Verify each QID via `mcp__wikidata__get_statements`
- Check P31 is geographic (not a person, concept, etc.)
- Check coordinates exist (P625) where expected
- Report any mismatches

If `$ARGUMENTS` is "verify":
- For places that already have QIDs but are unverified
- Verify each QID against its expected properties (dates, country, type)
- Mark as confirmed/wrong/uncertain

Otherwise, treat `$ARGUMENTS` as a target number of new matches to add (default: 50).

## Matching Procedure

### 1. Identify unmatched places

Determine input format and load unmatched places, sorted by frequency/importance.

### 2. Search Wikidata via MCP

For each place:

1. Search with `mcp__wikidata__search_items(place_name)`
2. Review candidates — look for geographic entities (not people, concepts, media)
3. If multiple candidates, use `mcp__wikidata__get_statements(qid)` to check:
   - **P31** (instance of): Should be geographic — city, country, island, colony, etc.
   - **P625** (coordinates): Confirms it's a real place
   - **P17** (country): Helps disambiguate namesakes
   - **P571/P576** (inception/dissolved): Critical for historical political entities
   - **P1566** (GeoNames ID): Useful cross-reference

### 3. Historical Entity Rules

**CRITICAL**: For historical political entities (colonies, kingdoms, empires), match the SPECIFIC entity for the time period, not the modern country.

| Place | Wrong QID | Right QID | Why |
|-------|-----------|-----------|-----|
| Mauritius (Colony 1814-1968) | Q1027 (modern republic) | Q12053604 (Colony of Mauritius) | Different political entity |
| Ceylon | Q854 (Sri Lanka, acceptable) | Q854 | Same entity, historical name |
| Gold Coast Colony | Q117 (modern Ghana) | Q503585 (Gold Coast) | Colony ≠ modern state |
| Bengal Presidency | Q1194 (modern Bengal) | Q1249829 (Bengal Presidency) | Administrative entity |

**When to use the modern QID**: If no specific historical entity exists on Wikidata, the modern QID is acceptable — note it in the match.

### 4. Disambiguation Heuristics

When multiple Wikidata results match:

1. **Population/significance**: Prefer the most notable place (capital > city > town > village)
2. **Historical context**: For pre-1900 documents, prefer Old World over New World namesakes
   - "Canterbury" → Canterbury, England (Q29303) not Canterbury, New Zealand
   - "Cambridge" → Cambridge, England (Q350) not Cambridge, Massachusetts
3. **Document context**: Use surrounding text to disambiguate
   - "Georgia" in a British Empire context → Colony of Georgia or country
   - "Georgia" in a US context → US state
4. **Coordinates**: If source data has lat/lon, verify against Wikidata P625

### 5. Skip Rules — do NOT match these

- **Generic directional terms**: "the North", "the East", "the Interior"
- **Relative locations**: "the opposite shore", "nearby", "adjacent territory"
- **Ambiguous without context**: "the colony", "the island", "the port"
- **Non-place NER errors**: words misidentified as places by NER (e.g., "Ammonia", "Bleaching")
- **Single-letter abbreviations**: "N.", "S.", "E.", "W." (compass directions)

### 6. Output Format

#### For JSONL cluster files
Append to match file:
```json
{"cluster_id": "london", "wikidata_qid": "Q84", "wikidata_label": "London", "wikidata_desc": "capital and largest city of England and the United Kingdom", "match_type": "mcp_verified", "geonames_id": "2643743"}
```

#### For knowledge graph verification
```json
{"node_id": "sierra_leone_colony_1808_1896", "name": "Sierra Leone Colony", "original_qid": "Q30059027", "verified_qid": "Q30059027", "wikidata_label": "Colony of Sierra Leone", "verdict": "confirmed", "notes": "Label and dates match exactly"}
```

Verdict values: `confirmed`, `wrong`, `uncertain`, `no_match_found`

### 7. Validate after each batch

- Verify QIDs are geographic entities (P31 check)
- Check for duplicate QIDs assigned to different places (may be valid for hierarchical names, but flag)
- Deduplicate if processing cluster-format data

### 8. Variant Awareness

Many places appear under multiple names/spellings. When you match one form, check for variants:
- Historical spellings: "Cawnpore" / "Kanpur", "Bombay" / "Mumbai"
- Abbreviated forms: "Lond." / "London", "Edin." / "Edinburgh"
- Colonial names: "Batavia" / "Jakarta", "Ceylon" / "Sri Lanka"
- With/without articles: "The Gambia" / "Gambia"
- Transliterations: "Peking" / "Beijing", "Calcutta" / "Kolkata"

Use `grep -i 'PATTERN' INPUT_FILE | head` to find all variants of a place name.

### 9. Colonial-to-Modern Mappings

Common colonial name → modern equivalents (for reference):

| Colonial Name | Modern Name | QID (colonial) | QID (modern) |
|--------------|-------------|-----------------|--------------|
| Ceylon | Sri Lanka | Q854 | Q854 |
| Gold Coast | Ghana | Q503585 | Q117 |
| British Honduras | Belize | Q505038 | Q242 |
| British Guiana | Guyana | Q499035 | Q734 |
| Rhodesia | Zimbabwe | Q217169 | Q954 |
| Formosa | Taiwan | Q22502 | Q865 |
| Siam | Thailand | Q869 | Q869 |
| Persia | Iran | Q794 | Q794 |
| Abyssinia | Ethiopia | Q115 | Q115 |
| Mesopotamia | Iraq | Q796 | Q796 |

### 10. Commit periodically

After adding 30+ matches, commit and push:
```bash
git add OUTPUT_FILE
git commit -m "Add N more place disambiguation matches (total: NNNN)"
git push
```

## Quality Standards

- Every QID MUST come from MCP tool results — never invented
- For historical political entities, the QID must match the specific time period
- Verify P31 is geographic for at least a sample of each batch
- When in doubt, skip — a false match is worse than a missing one
- Flag cases where modern QID was used because no historical entity exists
- Prefer precision over recall: only match when confident
