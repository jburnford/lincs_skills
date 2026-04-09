---
name: cidoc-crm
description: CIDOC-CRM ontology reference for cultural heritage knowledge graphs. Use when modeling entities, choosing classes/properties, or checking ontology correctness. Covers core CRM v7.3.1 (including former CRMgeo classes integrated in v7.1+) and CRMdig extension.
user-invocable: true
allowed-tools: Read, Grep, Glob, WebFetch
argument-hint: [class-or-property-or-question]
---

# CIDOC-CRM Ontology Reference

You are a CIDOC-CRM ontology expert. When the user asks about classes, properties, or modeling patterns, provide accurate guidance based on the reference below.

## Output Constraints

When generating data models, always default to valid **RDF Turtle (`.ttl`)** syntax unless asked otherwise. Always include standard prefixes (`rdf:`, `rdfs:`, `owl:`, `xsd:`, `skos:`, `crm:`, `crmdig:`). Every generated entity MUST have `rdfs:label`. Never attach dates or places directly to an entity — always route through an event node (this is CIDOC-CRM's event-centric pattern, not schema.org's flat model).

## Query: $ARGUMENTS

## Core Ontology: CIDOC-CRM v7.3.1

Namespace: `http://www.cidoc-crm.org/cidoc-crm/` (prefix: `crm:`)
ISO Standard: ISO 21127
Current Version: 7.3.1 (there is NO version 2.0 — version history goes 3.x → 4.x → 5.x → 6.x → 7.x)

### Frequently Used Classes

| Class | Label | Scope |
|-------|-------|-------|
| E1_CRM_Entity | CRM Entity | Top-level abstract class |
| E2_Temporal_Entity | Temporal Entity | Things with temporal extent |
| E4_Period | Period | Named historical periods |
| E5_Event | Event | Something that happened |
| E7_Activity | Activity | Intentional human action |
| E12_Production | Production | Creation of physical things |
| E13_Attribute_Assignment | Attribute Assignment | Scholarly assertion about an attribute. Use when extracting uncertain facts from noisy sources (OCR, ambiguous records). Allows attaching provenance and certainty to the assertion itself, not just the value. |
| E16_Measurement | Measurement | Observation of a quantitative property |
| E21_Person | Person | Individual human being |
| E33_Linguistic_Object | Linguistic Object | Identifiable expression in language |
| E33_E41_Linguistic_Appellation | Linguistic Appellation | A name expressed in language |
| E39_Actor | Actor | Person or group |
| E41_Appellation | Appellation | Name used to identify something |
| E42_Identifier | Identifier | Assigned code/number |
| E52_Time-Span | Time-Span | Temporal extent with begin/end |
| E53_Place | Place | Persistent spatial extent |
| E54_Dimension | Dimension | Quantitative measurement value |
| E55_Type | Type | Classification concept |
| E56_Language | Language | A natural language |
| E58_Measurement_Unit | Measurement Unit | Unit of measure |
| E65_Creation | Creation | Intellectual creation event |
| E66_Formation | Formation | Establishment of a group |
| E67_Birth | Birth | Birth of a person |
| E68_Dissolution | Dissolution | End of a group |
| E69_Death | Death | Death of a person |
| E73_Information_Object | Information Object | Immaterial item with propositional content |
| E74_Group | Group | Collection of persons |
| E77_Persistent_Item | Persistent Item | Enduring entity |
| E78_Curated_Holding | Curated Holding | Collection managed together |
| E81_Transformation | Transformation | Change destroying one thing and creating another |
| E85_Joining | Joining | Becoming a member of a group |
| E86_Leaving | Leaving | Ceasing membership in a group |
| E90_Symbolic_Object | Symbolic Object | Identifiable symbolic content |
| E30_Right | Right | Legal entitlement |
| E93_Presence | Presence | Spatiotemporal extent of a persistent item during a period (integrated from CRMgeo in v7.1+) |
| E94_Space_Primitive | Space Primitive | Geometric coordinates — points, polygons (integrated from CRMgeo in v7.1+) |

### Frequently Used Properties

| Property | Domain | Range | Meaning |
|----------|--------|-------|---------|
| P1_is_identified_by | E1 | E41/E42 | Entity has this name/identifier. Use E41_Appellation (or E33_E41) for human-readable names (strings). Use E42_Identifier for formal codes, database IDs, and URIs. |
| P2_has_type | E1 | E55 | Entity is classified as |
| P4_has_time-span | E2 | E52 | Temporal entity has this time extent |
| P7_took_place_at | E4 | E53 | Event/period occurred at this place |
| P10_falls_within | E4 | E4 | Period is within another period |
| P11_had_participant | E5 | E39 | Event involved this actor |
| P14_carried_out_by | E7 | E39 | Activity done by this actor |
| P14i_performed | E39 | E7 | Actor performed this activity |
| P39_measured | E16 | E1 | Measurement observed this entity |
| P40_observed_dimension | E16 | E54 | Measurement yielded this dimension |
| P67_refers_to | E89 | E1 | Information object mentions entity |
| P70_documents | E31 | E1 | Document records information about |
| P72_has_language | E33 | E56 | Written/spoken in this language |
| P82_at_some_time_within | E52 | xsd:string | Human-readable date label |
| P82a_begin_of_the_begin | E52 | xsd:dateTime | Earliest possible start. Use `xsd:date` if time unknown, but ensure valid ISO 8601 (e.g., `"1871-01-01"^^xsd:date`). Use `xsd:dateTime` when time is known (e.g., `"1871-01-01T00:00:00"^^xsd:dateTime`). |
| P82b_end_of_the_end | E52 | xsd:dateTime | Latest possible end. Same datatype rules as P82a. |
| P86_falls_within | E52 | E52 | Time-span is within another |
| P89_falls_within | E53 | E53 | Place is spatially within another |
| P91_has_unit | E54 | E58 | Dimension measured in this unit |
| P94_has_created | E65 | E28 | Creation event produced this |
| P98_brought_into_life | E67 | E21 | Birth brought this person into life |
| P100_was_death_of | E69 | E21 | Death was the death of this person |
| P104_is_subject_to | E72 | E30 | Entity is subject to this right |
| P106_is_composed_of | E90 | E90 | Composed of parts |
| P107_has_current_or_former_member | E74 | E39 | Group includes/included this actor |
| P127_has_broader_term | E55 | E55 | Type hierarchy (narrower → broader) |
| P129_is_about | E89 | E1 | Information object is about entity |
| P130_shows_features_of | E70 | E70 | Digital copy shows features of original |
| P140_assigned_attribute_to | E13 | E1 | Attribute assignment target |
| P141_assigned | E13 | E1 | Attribute value assigned |
| P143_joined | E85 | E39 | Joining added this actor |
| P144_joined_with | E85 | E74 | Joining added to this group |
| P168_place_is_defined_by | E53 | `geo:wktLiteral` or xsd:string | WKT coordinate geometry. Use `"POINT(lon lat)"^^geo:wktLiteral` for GeoSPARQL-compatible triplestores. Use `"POINT(lon lat)"^^xsd:string` for simpler deployments. For polygons: `"POLYGON((lon1 lat1, lon2 lat2, ...))"^^geo:wktLiteral`. |
| P190_has_symbolic_content | E90 | xsd:string | Literal string value |
| P161_has_spatial_projection | E93 | E53 | Presence has this spatial extent (integrated from CRMgeo in v7.1+) |
| P164_is_temporally_specified_by | E93 | E52 | Presence has this temporal extent (integrated from CRMgeo in v7.1+) |
| P166_was_a_presence_of | E93 | E77 | Presence manifests this persistent item (integrated from CRMgeo in v7.1+) |
| P132_spatiotemporally_overlaps_with | E92 | E92 | Two spacetime volumes overlap (integrated from CRMgeo in v7.1+) |
| P134_continued | E93 | E93 | Temporal continuity between presences (integrated from CRMgeo in v7.1+) |

## Golden Pattern: The Event-Centric Model

CIDOC-CRM is event-centric: dates and places attach to **events**, not directly to entities. This is the single most important pattern to internalize. LLMs trained on schema.org will instinctively put `birthDate` directly on a Person — this is WRONG in CIDOC-CRM.

### Example: A Person's Birth

```turtle
@prefix crm: <http://www.cidoc-crm.org/cidoc-crm/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix geo: <http://sws.geonames.org/> .
@prefix viaf: <http://viaf.org/viaf/> .

# The person
viaf:29539039 a crm:E21_Person ;
    rdfs:label "Louis Riel"@en ;
    crm:P1_is_identified_by [ a crm:E33_E41_Linguistic_Appellation ;
        rdfs:label "Name of Louis Riel"@en ;
        crm:P190_has_symbolic_content "Louis Riel"^^xsd:string ] .

# The birth EVENT — dates and places attach here, not on the person
<http://example.org/event/birth_riel> a crm:E67_Birth ;
    rdfs:label "Birth of Louis Riel"@en ;
    crm:P98_brought_into_life viaf:29539039 ;
    crm:P7_took_place_at geo:6183235 ;          # Red River Settlement
    crm:P4_has_time-span [ a crm:E52_Time-Span ;
        rdfs:label "22 October 1844"@en ;
        crm:P82_at_some_time_within "1844-10-22"^^xsd:string ;
        crm:P82a_begin_of_the_begin "1844-10-22T00:00:00"^^xsd:dateTime ;
        crm:P82b_end_of_the_end "1844-10-22T23:59:59"^^xsd:dateTime ] .

# The place
geo:6183235 a crm:E53_Place ;
    rdfs:label "Red River Settlement"@en ;
    crm:P89_falls_within geo:6065171 .           # Manitoba
```

**Key structural rules visible in this example:**
1. Person has NO `birthDate` or `birthPlace` property — those go on the Birth event
2. Every node has `rdfs:label`
3. Time-span has all three date properties (P82, P82a, P82b)
4. Place uses GeoNames authority URI, not an internal ID
5. Names route through E33_E41 with P190 for the literal string

### Example: A Census Measurement

```turtle
@prefix crm: <http://www.cidoc-crm.org/cidoc-crm/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix ex: <http://example.org/chgis/> .

# The measurement EVENT — the value attaches here, not on the place
ex:MEAS_ON082003_1871_POP a crm:E16_Measurement ;
    rdfs:label "Population measurement of Westmeath (1871)"@en ;
    crm:P39_measured ex:ON082003_1871 ;         # What was measured (presence)
    crm:P2_has_type ex:VAR_POP_XX_N ;           # What variable
    crm:P40_observed_dimension ex:DIM_ON082003_1871_POP ;
    crm:P4_has_time-span ex:ts_1871 .

ex:DIM_ON082003_1871_POP a crm:E54_Dimension ;
    rdfs:label "2632"@en ;
    crm:P90_has_value "2632"^^xsd:integer ;
    crm:P91_has_unit ex:UNIT_PERSONS .

ex:ts_1871 a crm:E52_Time-Span ;
    rdfs:label "1871"@en ;
    crm:P82_at_some_time_within "1871"^^xsd:string ;
    crm:P82a_begin_of_the_begin "1871-01-01T00:00:00"^^xsd:dateTime ;
    crm:P82b_end_of_the_end "1871-12-31T23:59:59"^^xsd:dateTime .

# The temporal presence (E93 is core CRM since v7.1+)
ex:ON082003_1871 a crm:E93_Presence ;
    rdfs:label "Westmeath presence (1871)"@en ;
    crm:P166_was_a_presence_of ex:PLACE_ON142032 .

# The enduring place
ex:PLACE_ON142032 a crm:E53_Place ;
    rdfs:label "Westmeath"@en ;
    owl:sameAs <http://sws.geonames.org/5914691/> .
```

## Spatiotemporal Modeling: E93_Presence

**Note**: E93_Presence, E94_Space_Primitive, and properties P166, P164, P161, P132, P134 were originally defined in the CRMgeo extension but were **integrated into core CIDOC-CRM in v7.1+**. They use the `crm:` prefix, not `crmgeo:`.

### When to Use E93_Presence vs. E53_Place Alone

Use E93_Presence when:
- Place boundaries change over time and you need to track each configuration
- You need to record spatial overlap between temporal snapshots
- You're building a historical GIS with time-scoped geometries

Use E53_Place + P168 + P89 alone when:
- Places are treated as stable points with fixed coordinates
- Spatial hierarchy is static (no boundary changes)

## Extension: CRMdig (Digital)

Namespace: `http://www.ics.forth.gr/isl/CRMdig/` (prefix: `crmdig:`)

| Class | Description |
|-------|-------------|
| D1_Digital_Object | Digital surrogate of physical or conceptual item |
| D7_Digital_Machine_Event | Processing by software |

## Common Anti-Patterns (Things to Avoid)

1. **Properties on relationships in RDF**: CRM properties (P89, P122, etc.) are simple triples in RDF. They cannot carry their own attributes (like `during_period` or `border_length`). Use reification, RDF-Star, or route through events instead.

2. **Confusing E4_Period with E52_Time-Span**: E4_Period is a named historical period (e.g., "The Victorian Era"). E52_Time-Span is a date range attached to events via P4. Don't use E4_Period as a general temporal index.

3. **Skipping E52_Time-Span**: Always attach dates via `E52_Time-Span` with `P82a`/`P82b` bounds. Don't put date properties directly on events or entities.

4. **Missing rdfs:label**: Every entity should have an `rdfs:label` for discoverability and tooling compatibility.

5. **Internal-only identifiers**: Use external authority URIs (GeoNames, VIAF, Wikidata) alongside internal IDs for interoperability.

6. **Using E13 when E16 is appropriate**: E13_Attribute_Assignment is for scholarly interpretations and uncertain attributions. E16_Measurement is for empirical observations with quantitative results. Census counts are E16, not E13.

7. **Flat schema.org-style modeling**: Never put `birthDate`, `birthPlace`, `occupation`, or similar properties directly on a Person or Place node. CIDOC-CRM is event-centric: create an event node (E67_Birth, E7_Activity, etc.) and attach dates/places to the event. See the Golden Pattern examples above.

8. **E55_Type without vocabulary grounding**: Don't create ad-hoc E55_Type nodes with only internal labels. Ground types in external vocabularies: Wikidata QIDs, Getty AAT, or LINCS SKOS vocabularies. When using E55_Type for categorical/taxonomic data, also declare the node as `skos:Concept` to enable interoperability with SKOS-based thesauri:
    ```turtle
    base:VAR_POP_XX_N a crm:E55_Type, skos:Concept ;
        rdfs:label "Total population"@en ;
        skos:prefLabel "Total population"@en ;
        skos:broader base:VAR_POPULATION ;
        crm:P127_has_broader_term base:VAR_POPULATION .
    ```
    Use both `skos:broader` and `crm:P127_has_broader_term` when bridging CRM with external thesauri.

## Handling Uncertainty and Noisy Data

When building knowledge graphs from primary historical sources (OCR'd documents, ambiguous records, contested attributions), use E13_Attribute_Assignment to wrap the uncertain assertion. This separates the claim from the evidence, allowing consumers to assess reliability.

### Pattern: Uncertain Birth Date (from OCR'd Census)

```turtle
# The assertion itself is uncertain
<http://example.org/assignment/birth_date_riel> a crm:E13_Attribute_Assignment ;
    rdfs:label "Uncertain birth date assignment for Louis Riel"@en ;
    crm:P140_assigned_attribute_to <http://example.org/event/birth_riel> ;
    crm:P141_assigned [ a crm:E52_Time-Span ;
        rdfs:label "circa 1844"@en ;
        crm:P82_at_some_time_within "c. 1844"^^xsd:string ;
        crm:P82a_begin_of_the_begin "1843-01-01T00:00:00"^^xsd:dateTime ;
        crm:P82b_end_of_the_end "1845-12-31T23:59:59"^^xsd:dateTime ] ;
    crm:P2_has_type <http://example.org/type/uncertain_date> ;
    crm:P17_was_motivated_by [ a crm:E73_Information_Object ;
        rdfs:label "OCR'd census record (poor quality)"@en ] .
```

**Key rules for uncertainty:**
- Widen `P82a`/`P82b` bounds to reflect the range of possibility
- Use `P82_at_some_time_within` for the human-readable qualified date ("c. 1844", "before 1900", "1870s")
- Attach the source via `P17_was_motivated_by` so the provenance of the uncertain claim is traceable
- For completely unknown dates, omit the E52_Time-Span entirely rather than inventing one
- For uncertain places, use the same E13 wrapper around the P7_took_place_at assertion

### When to Use E13 vs. E16

| Scenario | Use | Reason |
|----------|-----|--------|
| Census counted 2,632 people | E16_Measurement | Empirical observation with numeric result |
| Scholar attributes painting to Rembrandt | E13_Attribute_Assignment | Scholarly interpretation, potentially contested |
| OCR extracted a date but confidence is low | E13_Attribute_Assignment | Uncertain extraction from noisy source |
| Gender recorded in historical document | E13_Attribute_Assignment | Social category assigned by observer |
| Thermometer reads 20.5°C | E16_Measurement | Direct physical measurement |

## Modeling Decision Guide

When asked "how should I model X?", follow this decision tree:

1. **Is it a person?** → E21_Person (link to VIAF/Wikidata)
2. **Is it a group/organization?** → E74_Group
3. **Is it a place?** → E53_Place (link to GeoNames)
4. **Does the place change over time?** → Add CRMgeo E93_Presence per period
5. **Is it something that happened?** → E7_Activity or more specific event subclass
6. **Is it a birth/death?** → E67_Birth / E69_Death
7. **Is it a group forming/ending?** → E66_Formation / E68_Dissolution
8. **Is it someone joining/leaving a group?** → E85_Joining / E86_Leaving
9. **Is it a measurement/observation?** → E16_Measurement → E54_Dimension + E58_Unit
10. **Is it a document/text?** → E33_Linguistic_Object or E73_Information_Object
11. **Is it a digital file?** → CRMdig:D1_Digital_Object
12. **Is it a classification/category?** → E55_Type (link to Wikidata/SKOS vocab)
13. **Is it a name?** → E33_E41_Linguistic_Appellation with P190 for string value
14. **Is it an uncertain assertion?** → E13_Attribute_Assignment wrapping the pattern

Always attach temporal extent via P4 → E52_Time-Span, locations via P7 → E53_Place, and provenance via P70_documents or P67_refers_to.
