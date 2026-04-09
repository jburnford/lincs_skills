---
name: neo4j-to-rdf
description: Convert CIDOC-CRM data from Neo4j property graph format to valid RDF/Turtle serialization. Handles namespace mapping, relationship property reification, authority URI insertion, and LINCS-compatible output.
user-invocable: true
allowed-tools: Read, Grep, Glob, Write, Edit
argument-hint: [neo4j-data-or-cypher-query]
---

# Neo4j to RDF/Turtle Converter

Convert Neo4j CIDOC-CRM data to LINCS-compatible RDF/Turtle.

## Input: $ARGUMENTS

## Important Notes

- **Base URI**: Replace the `base:` prefix below with the actual project URI provided by the user or derived from the data context. Do NOT use `http://example.org/` in production output.
- **Strict Typing**: Always append XSD datatype suffixes to RDF literals based on the Neo4j property type: `^^xsd:integer` for integers, `^^xsd:decimal` for floats, `^^xsd:string` for strings, `^^xsd:dateTime` for timestamps. Never leave literals untyped.
- **Labels**: Every entity MUST have `rdfs:label` with a language tag (e.g., `@en`).

## Namespace Prefixes (Always Include)

```turtle
@prefix crm: <http://www.cidoc-crm.org/cidoc-crm/> .
@prefix crmdig: <http://www.ics.forth.gr/isl/CRMdig/> .
@prefix oa: <http://www.w3.org/ns/oa#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .
@prefix geonames: <http://sws.geonames.org/> .
@prefix viaf: <http://viaf.org/viaf/> .
@prefix wikidata: <http://www.wikidata.org/entity/> .
@prefix lincs: <http://id.lincsproject.ca/> .
@prefix biography: <http://id.lincsproject.ca/biography/> .
@prefix event: <http://id.lincsproject.ca/event/> .
@prefix identity: <http://id.lincsproject.ca/identity/> .
@prefix occupation: <http://id.lincsproject.ca/occupation/> .
@prefix aat: <http://vocab.getty.edu/aat/> .
@prefix lexvo: <http://lexvo.org/id/iso639-3/> .
@prefix geosparql: <http://www.opengis.net/ont/geosparql#> .
@prefix base: <http://example.org/chgis/> .  # REPLACE with actual project URI
```

## Translation Rules

### Rule 1: Node Labels → rdf:type
```
Neo4j:  (:E53_Place {place_id: 'ON142032', name: 'Westmeath'})
Turtle: base:PLACE_ON142032 a crm:E53_Place ;
            rdfs:label "Westmeath"@en .
```
### Rule 2: Simple Relationships → Predicates
```
Neo4j:  (p:E93_Presence)-[:P166_was_a_presence_of]->(pl:E53_Place)
Turtle: base:ON082003_1871 crm:P166_was_a_presence_of base:PLACE_ON142032 .
```

### Rule 3: Relationship Properties → Reification
Neo4j allows properties on relationships. RDF does not. Convert using reification or intermediate nodes.

**P89_falls_within with during_period**:
```
Neo4j:  (csd)-[:P89_falls_within {during_period: '1871'}]->(cd)
```
DO NOT convert directly. Instead, use the E93_Presence → P10_falls_within path.

**IMPORTANT**: `crm:P10_falls_within` has domain/range `E92_Spacetime_Volume` (and subclasses: `E93_Presence`, `E4_Period`). Both subject and object MUST be typed as `crm:E93_Presence` (or `crm:E4_Period`), never bare `crm:E53_Place`.

```turtle
# Both nodes must be E93_Presence — not E53_Place
base:ON082003_1871 a crm:E93_Presence ;
    rdfs:label "Westmeath CSD presence (1871)"@en .
base:CD_ON_Renfrew_North_1871 a crm:E93_Presence ;
    rdfs:label "Renfrew North CD presence (1871)"@en .

base:ON082003_1871 crm:P10_falls_within base:CD_ON_Renfrew_North_1871 .
```
The temporal scoping is inherent in the presence nodes — no property needed. If the target is only an `E53_Place` with no presence node, you must first create the corresponding `E93_Presence` and link it via `crm:P166_was_a_presence_of`.

**P122_borders_with with shared_border_length**:
```
Neo4j:  (a)-[:P122_borders_with {shared_border_length_m: 14542.51}]->(b)
```
Option A (drop the measurement — simplest):
```turtle
base:PLACE_A crm:P122_borders_with base:PLACE_B .
```
Option B (reify with E16_Measurement):
```turtle
base:border_meas_A_B a crm:E16_Measurement ;
    rdfs:label "Border measurement between A and B"@en ;
    crm:P39_measured base:PLACE_A ;
    crm:P40_observed_dimension base:border_dim_A_B .
base:border_dim_A_B a crm:E54_Dimension ;
    crm:P90_has_value "14542.51"^^xsd:decimal ;
    crm:P91_has_unit base:UNIT_METRES .
```

### Rule 4: Custom Relationships → Events
```
Neo4j:  (a)-[:PLACE_LINEAGE {type: 'SPLIT_FROM', year: 1881}]->(b)
```
Convert to E81_Transformation:
```turtle
base:transform_1881_A a crm:E81_Transformation ;
    rdfs:label "Administrative split creating A from B (1881)"@en ;
    crm:P123_resulted_in base:PLACE_A ;
    crm:P124_transformed base:PLACE_B ;
    crm:P4_has_time-span base:ts_1881 .
```

### Rule 5: Internal IDs → Authority URIs + Internal
Keep internal URIs but add owl:sameAs links:
```turtle
base:PLACE_ON142032 a crm:E53_Place ;
    rdfs:label "Westmeath"@en ;
    owl:sameAs geonames:5914691/ .  # GeoNames ID for Westmeath, Ontario
```

### Rule 6: Measurement Chain (Complete)
```turtle
# Source provenance
base:SOURCE_1871_V1T1 a crm:E73_Information_Object ;
    rdfs:label "Census 1871 Volume 1 Table 1"@en ;
    crm:P70_documents base:MEAS_ON082003_1871_POP_XX_N .

# Measurement event
base:MEAS_ON082003_1871_POP_XX_N a crm:E16_Measurement ;
    rdfs:label "Population measurement of Westmeath (1871)"@en ;
    crm:P39_measured base:ON082003_1871 ;
    crm:P2_has_type base:VAR_POP_XX_N ;
    crm:P40_observed_dimension base:DIM_ON082003_1871_POP_XX_N ;
    crm:P4_has_time-span base:ts_1871 .

# Variable type
base:VAR_POP_XX_N a crm:E55_Type ;
    rdfs:label "Total population"@en .

# Dimension (value)
base:DIM_ON082003_1871_POP_XX_N a crm:E54_Dimension ;
    rdfs:label "2632"@en ;
    crm:P90_has_value "2632"^^xsd:integer ;
    crm:P91_has_unit base:UNIT_PERSONS .

# Unit
base:UNIT_PERSONS a crm:E58_Measurement_Unit ;
    rdfs:label "persons"@en .

# Time-span
base:ts_1871 a crm:E52_Time-Span ;
    rdfs:label "1871"@en ;
    crm:P82_at_some_time_within "1871"^^xsd:string ;
    crm:P82a_begin_of_the_begin "1871-01-01T00:00:00"^^xsd:dateTime ;
    crm:P82b_end_of_the_end "1871-12-31T23:59:59"^^xsd:dateTime .
```

### Rule 7: Name/Appellation Pattern

**IMPORTANT**: Match the appellation type to the entity kind. Do NOT use `biography:personalName` for places.

| Entity Kind | Appellation Type | URI |
|-------------|-----------------|-----|
| Person name | Personal name | `biography:personalName` |
| Group name | Group name | `biography:groupName` |
| Place name (toponym) | Toponym | `aat:300387122` (Getty AAT) |
| Place name (alt) | Geographic name | `wikidata:Q7884789` |
| Formal title | Title | `biography:title` |

```
Neo4j:  (:E41_Appellation {name: 'Westmeath', type: 'canonical'})
Turtle:
base:appellation_Westmeath a crm:E33_E41_Linguistic_Appellation ;
    rdfs:label "Name of Westmeath"@en ;
    crm:P2_has_type aat:300387122 ;  # toponym — NOT biography:personalName
    crm:P190_has_symbolic_content "Westmeath"^^xsd:string .
base:PLACE_ON142032 crm:P1_is_identified_by base:appellation_Westmeath .
```

For OCR variant names attached to a specific E93_Presence:
```turtle
base:appellation_Wesmeath a crm:E33_E41_Linguistic_Appellation ;
    rdfs:label "OCR variant name: Wesmeath"@en ;
    crm:P2_has_type aat:300387122 ;
    crm:P190_has_symbolic_content "Wesmeath"^^xsd:string .
base:ON114011_1881 crm:P1_is_identified_by base:appellation_Wesmeath .
```

### Rule 8: Spatial Coordinates → GeoSPARQL WKT

Neo4j stores spatial data as native Point types, lat/lon properties, or WKT strings. Convert to GeoSPARQL `wktLiteral` for compatibility with spatial triplestores and GIS tools.

**Point coordinates (centroid)**:
```
Neo4j:  (:E94_Space_Primitive {latitude: 45.763, longitude: -76.884})
Turtle:
base:SP_ON082003_1871 a crm:E94_Space_Primitive ;
    rdfs:label "Centroid of Westmeath (1871)"@en ;
    crm:P168_place_is_defined_by
        "<http://www.opengis.net/def/crs/EPSG/0/4326> POINT(-76.884 45.763)"^^geosparql:wktLiteral .
```

**Polygon boundaries**:
```
Neo4j:  (:E53_Place {geometry: 'POLYGON((-76.9 45.7, -76.8 45.7, ...))'})
Turtle:
base:PLACE_ON142032 crm:P168_place_is_defined_by
    "<http://www.opengis.net/def/crs/EPSG/0/4326> POLYGON((-76.9 45.7, -76.8 45.7, -76.8 45.8, -76.9 45.8, -76.9 45.7))"^^geosparql:wktLiteral .
```

**Key rules for spatial data:**
- WKT coordinate order is **longitude latitude** (x y), not lat/lon
- Always include the CRS URI prefix for GeoSPARQL 1.1: `<http://www.opengis.net/def/crs/EPSG/0/4326>`
- Use `geosparql:wktLiteral` datatype (not `xsd:string`) for spatial-capable triplestores
- For simpler deployments without GeoSPARQL, `"POINT(lon lat)"^^xsd:string` is acceptable
- Attach coordinates to `E53_Place` via `P168` for static places, or to `E94_Space_Primitive` via `P161` for time-varying presences

### Rule 9: Neo4j Datatype → XSD Datatype Mapping

Always strictly type RDF literals. Never leave literals untyped.

| Neo4j Type | XSD Datatype | Example |
|------------|-------------|---------|
| Integer / Long | `xsd:integer` | `"2632"^^xsd:integer` |
| Float / Double | `xsd:decimal` | `"14542.51"^^xsd:decimal` |
| String | `xsd:string` | `"Westmeath"^^xsd:string` |
| Boolean | `xsd:boolean` | `"true"^^xsd:boolean` |
| Date (no time) | `xsd:date` | `"1871-04-02"^^xsd:date` |
| DateTime | `xsd:dateTime` | `"1871-01-01T00:00:00"^^xsd:dateTime` |
| Year only | `xsd:string` via P82 | `"1871"^^xsd:string` (on E52_Time-Span) |

**Note**: For year-only dates, do NOT use `xsd:gYear` — CIDOC-CRM convention is to put the human-readable string on `P82_at_some_time_within` and use full `xsd:dateTime` on `P82a`/`P82b` with Jan 1 00:00:00 and Dec 31 23:59:59 bounds.

## Output Format

Generate valid Turtle with:
1. All namespace prefixes at top
2. Entities grouped by class (places, presences, measurements, etc.)
3. Comments separating sections
4. `rdfs:label` on every entity
5. Authority URI links where identifiable
