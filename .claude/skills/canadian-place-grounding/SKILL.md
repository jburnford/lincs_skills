---
name: canadian-place-grounding
description: Ground Canadian place entities to Wikidata QIDs using MCP semantic search. Handles CSDs, communities, townships, parishes, First Nations reserves, cities, counties, and historical place names. Use for any Canadian geographic entity disambiguation.
user-invocable: true
allowed-tools: Read, Grep, Glob, Bash, Edit, Write, mcp__wikidata__search_items, mcp__wikidata__get_statements, mcp__wikidata__execute_sparql
argument-hint: [count-to-process | "status" | "verify" | "audit"]
---

# Canadian Place Grounding Workflow

Ground Canadian geographic entities to Wikidata QIDs using the Wikidata MCP server's semantic vector search. This skill handles any Canadian place type: census subdivisions, communities, townships, parishes, rural municipalities, First Nations reserves, cities, counties, census divisions, geographic features, and historical place names.

## Why This Skill Exists

The Wikidata REST API (`wbsearchentities`) is USELESS for Canadian place disambiguation. It returns results by string similarity only, producing wrong matches like:
- "Sarnia" → genus of molluscs (not the Ontario city)
- "Walpole Island" → island in New Caledonia (not the Ontario First Nation reserve)
- "Montagnais Indians" → a Winslow Homer watercolour

**Always use MCP semantic vector search** (`mcp__wikidata__search_items`), which returns contextually relevant matches.

## MCP Tools

| Tool | Use For |
|------|---------|
| `mcp__wikidata__search_items(query)` | Find QIDs by semantic search |
| `mcp__wikidata__get_statements(qid)` | Verify a QID — check P31, P131, P625, P17 |
| `mcp__wikidata__execute_sparql(query)` | Bulk lookups — e.g., all municipalities in a county |

**Rate limit**: 5 requests/minute on the MCP endpoint. Plan batch jobs accordingly.

## Commands

If `$ARGUMENTS` is "status":
- Report grounding progress from whatever output file the project uses

If `$ARGUMENTS` is "verify":
- Batch-verify all QIDs in the output against Wikidata live data (P31, P131, P625)

If `$ARGUMENTS` is "audit":
- Pick 20 random matches and deep-verify via `get_statements`

Otherwise, treat `$ARGUMENTS` as a target number of places to process (default: 50).

## Matching Procedure

### 1. Identify unmatched places

Determine the project's input format (CSV, JSONL, or plain list). Load unmatched places sorted by importance/frequency. Common input fields: `name`, `province`, `latitude`, `longitude`, `place_type`.

### 2. Search Wikidata via MCP

For each place, call `mcp__wikidata__search_items` with a well-formed query. Send 6-8 parallel queries per batch.

**Canada-specific search strategies by province:**

| Province | Strategy | Example query |
|----------|----------|---------------|
| **QC** | Include CD/county name + "Quebec" to disambiguate French saint-name parishes | `"Saint-Charles-Borromée parish Bellechasse Quebec"` |
| **ON** | Include county name | `"Thurlow township Hastings Ontario"` |
| **NB** | Include "parish New Brunswick" | `"Sussex Parish Kings New Brunswick"` |
| **NS** | Include county | `"Guysborough municipality Nova Scotia"` |
| **SK** | Include province + type; for RMs use "No. NNN rural municipality" | `"Estevan No. 5 rural municipality Saskatchewan"` |
| **AB** | Include province + type; for LIDs/IDs append "Alberta" | `"Banff town Alberta"` |
| **MB** | Include province | `"Portage la Prairie city Manitoba"` |
| **BC** | Include "British Columbia" | `"Grand Forks city British Columbia"` |
| **PE** | Include "Prince Edward Island" (small island, many namesakes elsewhere) | `"Township 38 Prince Edward Island"` |
| **NL** | Include "Newfoundland" or "Labrador" | `"Corner Brook Newfoundland"` |

**General search tips:**
- Start with the place name + province. If ambiguous, add the county/CD name.
- For First Nations reserves, include "First Nation" or "Indian reserve" + province.
- For historical townships, include "township" + county.
- For French place names, search both accented and unaccented forms.
- If a search returns empty, try a shorter query (just the name + province).
- Normalize saint names: "St." / "Ste." / "Saint" / "Sainte" are interchangeable.
- Handle hyphenation: "Rivière-du-Loup" and "Riviere du Loup" may match differently.

### 3. Verify EVERY QID via MCP

**MANDATORY for every match.** Call `mcp__wikidata__get_statements` on the QID.

Check these properties:

| Property | Check | Fail action |
|----------|-------|-------------|
| **P31** (instance of) | Must be a geographic entity — city, town, village, township, parish, municipality, rural municipality, Indian reserve, census division, etc. **NOT** a weather station, railway station, river, electoral district, painting, or mollusc genus. | Reject match. |
| **P131** (located in) | Must chain to the expected province QID within 5 hops. Province QIDs: ON=Q1904, QC=Q176, NS=Q1952, NB=Q1965, MB=Q1948, BC=Q1974, PE=Q1979, SK=Q1989, AB=Q1951, NL=Q2003, NT=Q2007, NU=Q2023, YT=Q2009. | Reject match. |
| **P625** (coordinates) | Should be within 50km of the expected location (if coordinates are available in the source data). | Warning (Wikidata coords can be stale, especially for SK rural municipalities). |
| **Label/description** | Should be recognizably the same place. | Use judgment. |

**NEVER write a QID from memory. Every QID must come from search results or SPARQL output.**

### 4. Decision outcomes

| Status | When to use | URI outcome |
|--------|-------------|-------------|
| `matched` | QID found and verified | Use `http://www.wikidata.org/entity/{QID}` as the entity URI |
| `mint_uri` | No suitable Wikidata entity after thorough search | Use `http://temp.lincsproject.ca/{project}/place/{id}` until LINCS mints a permanent URI |
| `skip` | Not a real place (parsing artifact, "NO DATA", generic label) | No URI needed |

**`mint_uri` is expected for many Canadian historical places.** Historic townships, unorganized territories, obsolete parishes, and small historical settlements often have no Wikidata coverage. Minting is honest, not a failure.

### 5. Common Canadian place types on Wikidata

When checking P31 (instance of), these are typical valid types:

| Wikidata class | QID | Examples |
|---------------|-----|----------|
| city in Canada | Q1093829 | Toronto, Vancouver, Regina |
| town in Canada | Q3153110 | Banff, Canmore |
| village in Canada | Q3269977 | small settlements |
| township in Canada | Q15640105 | Ontario/Quebec geographic townships |
| municipality of Quebec | Q3327873 | Quebec municipalities |
| parish of New Brunswick | Q26160164 | NB civil parishes |
| rural municipality | Q15643668 | SK/MB rural municipalities |
| Indian reserve | Q1428439 | First Nations reserves |
| census subdivision | Q7228621 | generic fallback type |
| municipality of Ontario | Q15643637 | ON municipalities |
| regional municipality | Q15636674 | ON regional municipalities |

Also valid: county, census division, district municipality, township municipality, improvement district, etc. When in doubt, check if the P31 class has "Canada" or a Canadian province in its label/hierarchy.

### 6. Tricky disambiguation patterns

**French saint-name parishes (QC, NB, NS):**
Many Quebec CSDs are Catholic parishes named after saints. "Saint-Jean" exists in dozens of counties. ALWAYS include the county/CD name in the search. Verify P131 carefully — the wrong Saint-Jean will have a plausible-looking P31 but a wrong P131 chain.

**Saskatchewan rural municipalities:**
Format is often "NNN. Name" (e.g., "218. Cupar"). Search as "Cupar No. 218 rural municipality Saskatchewan". The RM number is the most reliable disambiguator.

**Ontario historic townships:**
Many are geographic townships that were never incorporated municipalities. Wikidata coverage is spotty. Include the county: "Thurlow township Hastings County Ontario". If no Wikidata entity exists, `mint_uri` is the correct outcome.

**First Nations communities:**
Use "First Nation" in the search, not "Indian reserve" (though both may work). For Wikidata, the reserve (land) and the First Nation (government/people) are often separate entities — match the geographic entity (the reserve), not the band government, unless the project specifically wants the governance entity.

**Bilingual names:**
Some places have official English and French names. Wikidata typically has both as labels/aliases. Search whichever form appears in your source data; the MCP semantic search handles both.

**Name changes over time:**
"Berlin" → "Kitchener" (Ontario, renamed 1916). Search with the name as it appears in your source data. Wikidata usually has historical names as aliases. For census projects anchored to a specific year, match the entity regardless of what it was called at that time — the Wikidata QID is the timeless referent.

### 7. Output format

Adapt to the project's existing format. Common pattern:

```json
{
  "place_id": "ON082003",
  "place_name": "Westmeath",
  "province": "ON",
  "wikidata_qid": "Q3132705",
  "wikidata_label": "Westmeath",
  "wikidata_desc": "township municipality in Ontario, Canada",
  "status": "matched",
  "match_type": "mcp_verified"
}
```

For `mint_uri` entries, include `"mint_reason"` explaining why no Wikidata entity was found.

### 8. Verification checkpoint — every 50 matches

After every 50 new matches, run a batch verification pass. If the project has a `--verify` script, use it. Otherwise, spot-check 10-15 QIDs manually via `get_statements`.

Signs of trouble:
- Multiple matches to the same QID (may be valid for variant names but flag it)
- QIDs that look "round" (Q123, Q456) — may indicate hallucination
- Large batches written without any `search_items` calls in between — STOP and verify

### 9. Commit periodically

After each verified batch:
```bash
git add OUTPUT_FILE
git commit -m "Ground N places to Wikidata (total: NNN)"
```

## Quality Standards

- Every QID MUST come from MCP tool results — never invented or recalled from memory
- Every QID MUST be verified via `get_statements` before writing
- P31 must be a geographic entity
- P131 must chain to the expected Canadian province
- When in doubt, `mint_uri` — a false positive is worse than a missing match
- Prefer precision over recall
