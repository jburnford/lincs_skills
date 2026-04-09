---
name: lincs-profile
description: LINCS (Linked Infrastructure for Networked Cultural Scholarship) application profile reference. Use when modeling Canadian history data for LINCS compatibility, checking conformance to LINCS patterns, or understanding how LINCS datasets are structured.
user-invocable: true
allowed-tools: Read, Grep, Glob, WebFetch, WebSearch
argument-hint: [pattern-or-question]
---

# LINCS Application Profile Reference

You are a LINCS linked data expert. Provide guidance on modeling data for compatibility with the LINCS infrastructure.

## Query: $ARGUMENTS

## Required Namespace Prefixes

When generating Turtle from this profile, always include these exact prefix declarations:

```turtle
@prefix crm: <http://www.cidoc-crm.org/cidoc-crm/> .
@prefix crmdig: <http://www.ics.forth.gr/isl/CRMdig/> .
@prefix oa: <http://www.w3.org/ns/oa#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .
@prefix geo: <http://sws.geonames.org/> .
@prefix viaf: <http://viaf.org/viaf/> .
@prefix wikidata: <http://www.wikidata.org/entity/> .
# TODO: Replace these LINCS vocabulary namespace URIs with the correct ones
@prefix lincs: <https://lincs.digital/> .
@prefix biography: <https://lincs.digital/vocabulary/biography/> .
@prefix event: <https://lincs.digital/vocabulary/event/> .
@prefix identity: <https://lincs.digital/vocabulary/identity/> .
@prefix occupation: <https://lincs.digital/vocabulary/occupation/> .
@prefix aat: <http://vocab.getty.edu/aat/> .
@prefix lexvo: <http://lexvo.org/id/iso639-3/> .
```

Do NOT guess or hallucinate namespace URIs. The exact URIs above are authoritative.

## LINCS Overview

**Project**: Linked Infrastructure for Networked Cultural Scholarship
**SPARQL Endpoint**: `https://fuseki.lincsproject.ca/lincs/sparql` (POST)
**ResearchSpace UI**: `https://rs.lincsproject.ca/`
**Core Ontology**: CIDOC-CRM v7.3.1
**Extensions**: CRMdig (digital objects), OA Web Annotation Data Model

## History of Canada Datasets

### Named Graphs

| Dataset | Graph URI |
|---------|-----------|
| Historical Canadian Persons | `<http://graph.lincsproject.ca/hist-canada/hist-cdns>` |
| Historical Indian Affairs Agents | `<http://graph.lincsproject.ca/hist-canada/ind-affairs>` |
| Cabinet Conclusions | `<http://graph.lincsproject.ca/hist-canada/cabinet-conclusions>` |

### Data Archives
- Historical Canadians: `https://doi.org/10.5683/SP3/7ESLQ0`

## Authority Files (REQUIRED for LINCS Compatibility)

| Entity Type | Authority | URI Pattern | Example |
|-------------|-----------|-------------|---------|
| People | VIAF | `http://viaf.org/viaf/{id}` | `viaf:104461457` (Oronhyatekha) |
| People (alt) | Wikidata | `http://www.wikidata.org/entity/{id}` | `wikidata:Q7103784` |
| Places | GeoNames | `http://sws.geonames.org/{id}/` | `geo:6174041` (Victoria, BC) |
| Places (alt) | Wikidata | `http://www.wikidata.org/entity/{id}` | `wikidata:Q2166` |
| Types/Concepts | Wikidata | `http://www.wikidata.org/entity/{id}` | `wikidata:Q327333` (govt agency) |
| Archival types | Getty AAT | `http://vocab.getty.edu/aat/{id}` | `aat:300189759` (archival fonds) |
| Languages | Lexvo | `http://lexvo.org/id/iso639-3/{code}` | `lexvo:eng` |
| Novel entities | LINCS-minted | TODO: correct LINCS URI pattern | `lincs:uHpXmbg9zxY` |

If no external authority exists for an entity, mint a LINCS URI.

## LINCS Controlled Vocabularies (SKOS)

| Vocabulary | Prefix | Purpose | Examples |
|------------|--------|---------|----------|
| Biography | `biography:` | Name types | `biography:personalName`, `biography:groupName` |
| Event | `event:` | Event types | `event:OccupationEvent` |
| Identity | `identity:` | Gender/identity | `identity:woman`, `identity:man` |
| Occupation | `occupation:` | Occupation types | `occupation:teacher`, `occupation:author` |

## The 8 Reusable Graph Patterns

All LINCS datasets are built by composing these base patterns:

### Pattern 1: Type
```turtle
<entity> a crm:E1_CRM_Entity ;
    rdfs:label "Entity label"@en ;
    crm:P2_has_type <type_uri> .
<type_uri> a crm:E55_Type ;
    rdfs:label "Type label"@en .
```
**Rule**: Types MUST be grounded in Wikidata, Getty AAT, or LINCS vocabularies.

### Pattern 2: Identifier
```turtle
<entity> crm:P1_is_identified_by <identifier> .
<identifier> a crm:E42_Identifier ;
    rdfs:label "Identifier of Entity"@en ;
    crm:P2_has_type <identifier_type> ;
    crm:P190_has_symbolic_content "12345"^^xsd:integer .
```

### Pattern 3: Name & Title
```turtle
<entity> crm:P1_is_identified_by <name> .
<name> a crm:E33_E41_Linguistic_Appellation ;
    rdfs:label "Name of Entity"@en ;
    crm:P2_has_type biography:personalName ;
    crm:P190_has_symbolic_content "John Smith"^^xsd:string .
```
**CRITICAL RULE**: NEVER attach literal name strings directly to an `E1_CRM_Entity` (the only exception is `rdfs:label`, which is required for display). ALL name values MUST route through `E33_E41_Linguistic_Appellation` via `P1_is_identified_by`, with the string assigned via `P190_has_symbolic_content`. This applies to person names, place names, titles, and any other appellation. Writing `<person> crm:P1_is_identified_by "John Smith"` is INVALID — the range of P1 is E41, not a literal.

### Pattern 4: Location
```turtle
<event> crm:P7_took_place_at <place> .
<place> a crm:E53_Place ;
    rdfs:label "Ottawa, Canada"@en ;
    crm:P168_place_is_defined_by "POINT(-75.69719 45.42153)"^^xsd:string ;
    crm:P89_falls_within <broader_place> .
```
**Rule**: Places MUST use GeoNames URIs. Coordinates in WKT format.

### Pattern 5: Date
```turtle
<event> crm:P4_has_time-span <timespan> .
<timespan> a crm:E52_Time-Span ;
    rdfs:label "1871"@en ;
    crm:P82_at_some_time_within "1871"^^xsd:string ;
    crm:P82a_begin_of_the_begin "1871-01-01T00:00:00"^^xsd:dateTime ;
    crm:P82b_end_of_the_end "1871-12-31T23:59:59"^^xsd:dateTime .
```
**Rule**: Always provide P82 (human-readable), P82a (earliest start), P82b (latest end). For uncertain dates, widen the P82a/P82b range to bound the uncertainty.

**Uncertain date example** — "circa 1880" from an OCR'd primary source:
```turtle
<timespan_circa_1880> a crm:E52_Time-Span ;
    rdfs:label "circa 1880"@en ;
    crm:P82_at_some_time_within "circa 1880"^^xsd:string ;
    crm:P82a_begin_of_the_begin "1875-01-01T00:00:00"^^xsd:dateTime ;
    crm:P82b_end_of_the_end "1885-12-31T23:59:59"^^xsd:dateTime .
```
Common historical date qualifiers and their suggested bounds:
| Source Text | P82 Value | P82a (earliest) | P82b (latest) |
|-------------|-----------|-----------------|---------------|
| "circa 1880" | `"circa 1880"` | 1875-01-01 | 1885-12-31 |
| "Spring 1881" | `"Spring 1881"` | 1881-03-01 | 1881-05-31 |
| "before 1900" | `"before 1900"` | (omit) | 1899-12-31 |
| "after 1870" | `"after 1870"` | 1870-01-01 | (omit) |
| "1880s" | `"1880s"` | 1880-01-01 | 1889-12-31 |
| "late 19th century" | `"late 19th century"` | 1870-01-01 | 1899-12-31 |

### Pattern 6: Subject
```turtle
<information_object> crm:P129_is_about <entity> .
```
**Rule**: Use for the primary subject of a document/record. Use P67_refers_to for secondary mentions.

### Pattern 7: Mention
```turtle
<information_object> crm:P67_refers_to <entity> .
```
**Rule**: For entities mentioned but not the primary subject.

### Pattern 8: Attribute Assignment (Uncertainty)
```turtle
<assignment> a crm:E13_Attribute_Assignment ;
    crm:P140_assigned_attribute_to <entity> ;
    crm:P141_assigned <attribute_value> ;
    crm:P2_has_type <assignment_type> .
```
**Rule**: Use when an assertion is uncertain, contested, or needs provenance layering (e.g., uncertain birth date, gender assignment).

## Person Modeling (Historical Canadian Persons Pattern)

### Complete Person Template
```turtle
# Person
<person_uri> a crm:E21_Person ;
    rdfs:label "Person Name"@en ;
    crm:P1_is_identified_by <name_uri> ;
    owl:sameAs <wikidata_uri> .

# Name (Pattern 3)
<name_uri> a crm:E33_E41_Linguistic_Appellation ;
    rdfs:label "Name of Person Name"@en ;
    crm:P2_has_type biography:personalName ;
    crm:P190_has_symbolic_content "Person Name"^^xsd:string .

# Birth
<birth_uri> a crm:E67_Birth ;
    rdfs:label "Birth of Person Name"@en ;
    crm:P98_brought_into_life <person_uri> ;
    crm:P7_took_place_at <geo:NNNNNNN> ;     # Pattern 4
    crm:P4_has_time-span <birth_timespan> .    # Pattern 5

# Death
<death_uri> a crm:E69_Death ;
    rdfs:label "Death of Person Name"@en ;
    crm:P100_was_death_of <person_uri> ;
    crm:P7_took_place_at <geo:NNNNNNN> ;
    crm:P4_has_time-span <death_timespan> .

# Occupation
<occupation_uri> a crm:E7_Activity ;
    rdfs:label "Teacher occupation of Person Name"@en ;
    crm:P14_carried_out_by <person_uri> ;
    crm:P2_has_type event:OccupationEvent ;
    crm:P2_has_type occupation:teacher ;
    crm:P4_has_time-span <occ_timespan> ;
    crm:P7_took_place_at <geo:NNNNNNN> .

# Marriage (as group joining)
<marriage_uri> a crm:E85_Joining ;
    rdfs:label "Marriage of Person A and Person B"@en ;
    crm:P143_joined <person_uri> ;
    crm:P144_joined_with <marriage_group_uri> ;
    crm:P4_has_time-span <marriage_timespan> ;
    crm:P7_took_place_at <geo:NNNNNNN> .
<marriage_group_uri> a crm:E74_Group ;
    rdfs:label "Marriage group of Person A and Person B"@en .

# Gender (via Attribute Assignment — Pattern 8)
<gender_assignment> a crm:E13_Attribute_Assignment ;
    crm:P140_assigned_attribute_to <person_uri> ;
    crm:P141_assigned identity:man .
```

## Organization Modeling (Indian Affairs Pattern)

### Agency/Department Template
```turtle
# Department
<dept_uri> a crm:E74_Group ;
    rdfs:label "Department of Indian Affairs"@en ;
    crm:P2_has_type wikidata:Q111190932 ;  # Canadian federal department
    crm:P107_has_current_or_former_member <agency_uri> .

# Agency
<agency_uri> a crm:E74_Group ;
    rdfs:label "Carlton Agency"@en ;
    crm:P2_has_type wikidata:Q327333 ;  # government agency
    crm:P107_has_current_or_former_member <agent_uri> .

# Agent's occupation with employer
<occupation_uri> a crm:E7_Activity ;
    rdfs:label "Indian Agent occupation of John McKenzie"@en ;
    crm:P14_carried_out_by <agent_uri> ;
    crm:P11_had_participant <agency_uri>, <dept_uri> ;
    crm:P2_has_type event:OccupationEvent ;
    crm:P2_has_type <specific_occupation_type> ;
    crm:P4_has_time-span <timespan> ;
    crm:P7_took_place_at <geo:NNNNNNN> .
```

## Government Event Modeling (Cabinet Conclusions Pattern)

### Cabinet Meeting Template
```turtle
# Meeting
<meeting_uri> a crm:E7_Activity ;
    rdfs:label "Cabinet meeting of 1944-06-06"@en ;
    crm:P14_carried_out_by <ministry_uri> ;
    crm:P21_had_general_purpose <topic_uri> ;
    crm:P7_took_place_at geo:6094817 ;  # Ottawa
    crm:P4_has_time-span <timespan> .

# Role-qualified participation (uses PC14 property class)
# NOTE on ".1" properties: The dot in P14.1 can cause parser errors in strict
# Turtle parsers that enforce NCName rules. Two options:
#   Option A (safe): Use full URI in angle brackets (works everywhere)
#   Option B (compact): Use crm:P14.1_in_the_role_of (works in most modern parsers)
# When in doubt, use Option A.
<pc14_uri> a crm:PC14_Carried_Out_By ;
    crm:P01i_is_domain_of <meeting_uri> ;
    crm:P02_has_range <person_uri> ;
    <http://www.cidoc-crm.org/cidoc-crm/P14.1_in_the_role_of> <role_type> .  # Option A (safe)

# Ministry lifecycle
<ministry_uri> a crm:E74_Group ;
    rdfs:label "11th Canadian Ministry"@en ;
    crm:P95i_was_formed_by <formation_event> ;
    crm:P99i_was_dissolved_by <dissolution_event> .
```

## Document & Annotation Modeling

### DCB Biography with Text Annotation
```turtle
# Biography (digital object)
<bio_uri> a crmdig:D1_Digital_Object, crm:E33_Linguistic_Object ;
    rdfs:label "DCB Biography of Person"@en ;
    crm:P2_has_type wikidata:Q36774 ;  # biography
    crm:P72_has_language lexvo:eng ;
    crm:P129_is_about <person_uri> ;
    crm:P67_refers_to <mentioned_entity> ;
    crm:P106_is_composed_of <text_fragment> .

# Text fragment
<text_fragment> a crm:E73_Information_Object ;
    crm:P190_has_symbolic_content "was born in Halifax"^^xsd:string ;
    crm:P67_refers_to <geo:6324729> .  # Halifax

# Web Annotation linking text to entity
<annotation_uri> a oa:Annotation ;
    oa:hasTarget <text_fragment> ;
    oa:hasBody <geo:6324729> ;
    oa:motivatedBy oa:identifying .
```

## LINCS Compliance Checklist

When modeling data for LINCS, verify:

- [ ] Every entity has `rdfs:label`
- [ ] People use VIAF or Wikidata URIs (or LINCS-minted)
- [ ] Places use GeoNames URIs
- [ ] Types use Wikidata or LINCS vocabulary URIs
- [ ] Names use `E33_E41_Linguistic_Appellation` with `P190_has_symbolic_content`
- [ ] Dates use `E52_Time-Span` with P82, P82a, P82b
- [ ] Events connect to places via `P7_took_place_at`
- [ ] Events connect to dates via `P4_has_time-span`
- [ ] Documents use `P129_is_about` for subjects, `P67_refers_to` for mentions
- [ ] Uncertain assertions use `E13_Attribute_Assignment`
- [ ] Digital objects use `crmdig:D1_Digital_Object`
- [ ] Occupations are `E7_Activity` typed with `event:OccupationEvent`
- [ ] Marriage modeled as `E85_Joining` + `E74_Group`
- [ ] **CRITICAL**: No bare literal names on entities — `<person> crm:P1_is_identified_by "John"` is INVALID. Must route through `E33_E41_Linguistic_Appellation` with `P190_has_symbolic_content`. The ONLY literal allowed directly on an entity is `rdfs:label`.
- [ ] No properties on relationships (no Neo4j-style edge attributes in RDF)
- [ ] Property Class `.1` properties (e.g., `P14.1_in_the_role_of`) use full URI in angle brackets for parser safety
