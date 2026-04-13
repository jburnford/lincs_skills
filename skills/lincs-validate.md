---
name: lincs-validate
description: Validate a CIDOC-CRM data model or RDF output against LINCS application profile requirements. Checks for missing authority URIs, incorrect patterns, namespace issues, and interoperability gaps.
user-invocable: true
allowed-tools: Read, Grep, Glob, WebFetch
argument-hint: [file-or-description-to-validate]
---

# LINCS Compatibility Validator

Validate the provided model, file, or description against LINCS requirements.

## Target: $ARGUMENTS

## Validation Procedure

Read or analyze the target, then check against each category below. Report findings as PASS, WARN, or FAIL with specific remediation steps.

### Category 1: Namespace Correctness

Check that:
- Core CRM classes (E21, E53, E55, E7, E93, E94, etc.) and properties (P166, P164, P161, P132, P134) use `crm:` prefix (`http://www.cidoc-crm.org/cidoc-crm/`)
- CRMdig classes (D1_Digital_Object) use `crmdig:` prefix
- OA Annotation classes use `oa:` prefix
- No invented property names that aren't in any CRM spec (e.g., `PLACE_LINEAGE`, `SPLIT_FROM`)
- Don't use inverse CIDOC CRM properties.

**FAIL if**: Any of the above conditions are not met.


### Category 2: Authority URIs

Check that:
- Places have GeoNames URIs (`http://sws.geonames.org/{id}/`)
- People have VIAF (`http://viaf.org/viaf/{id}`) or Wikidata URIs
- Types/concepts have Wikidata QIDs or LINCS vocabulary URIs
- Every entity NOT in an external authority has a LINCS-minted URI

**FAIL if**: Only internal IDs (e.g., `PLACE_ON142032`) with no external authority link.

### Category 3: Required Properties

Check that every entity has:
- `rdfs:label` (human-readable label, language-tagged)
    - Where possible, include English and French labels, add language tags to all labels and literals (e.g. `"label"@en`).
- `rdf:type` (explicit class declaration)


All E42_Identifier and E33_E41_Linguistic_Appellation should have a P190_has_symbolic_content:
- `crm:P1_is_identified_by` → `crm:E33_E41_Linguistic_Appellation` or `crm:E42_Identifier`
- String value via `crm:P190_has_symbolic_content`


### Category 4: Temporal Modeling

Check that:
- Dates are always on `E52_Time-Span` nodes, not directly on entities
- Time-spans have `P82_at_some_time_within` (human string), `P82a_begin_of_the_begin`, `P82b_end_of_the_end`
- `P82a` and `P82b` use `^^xsd:dateTime` with full ISO 8601 format (e.g., `"1871-01-01T00:00:00"^^xsd:dateTime`)
- Events connect to time-spans via `P4_has_time-span`
- `E4_Period` is used only for named historical periods, not as a general temporal index

**FAIL if**: `P82a` or `P82b` values are typed as `xsd:string` or `xsd:date` instead of `xsd:dateTime`. Example of INVALID: `"1871-01-01"^^xsd:string`. Must be: `"1871-01-01T00:00:00"^^xsd:dateTime`.

**WARN if**: E4_Period used as a census-year grouping node (valid CRM, but not LINCS-compatible)

### Category 5: Spatial Modeling
Note: Coordinates are assumed to be in EPSG:4326 (WGS84) and will be transformed internally to EPSG:3857

Valid WKT Literals:
- "POINT(long lat)"@en
- "MULTIPOINT(long lat,long lat)"@en
- "LINESTRING(long lat,long lat,long lat)"@en
- "POLYGON((long lat, long lat, long lat))"@en
- "MULTIPOLYGON(((long lat, long lat), (long lat)))"@en
- "MULTILINESTRING((long lat, long lat), (long lat))"@en

Check that:
- Places use `P168_place_is_defined_by` with WKT literal: `"POINT(lon lat)"@en`
- Coordinate order is **longitude latitude** (x y), NOT latitude longitude
- Spatial hierarchy uses `P89_falls_within` (static, no temporal qualifier)
- For time-varying hierarchy, route through E93_Presence → P10_falls_within (both subject and object must be `E93_Presence` or `E4_Period`, never bare `E53_Place`)
- P89 does NOT carry properties (no `during_period`, `shared_border_length`)

**FAIL if**: Coordinates typed as `^^geosparql:wktLiteral` (We are not using GeoSPARQL).
**FAIL if**: Relationship properties on P89 or P122 (not valid in RDF).



### Category 6: Event-Centric Patterns

Check that:
- Occupations are `E7_Activity` typed `event:OccupationEvent` (not direct type on person)
- Birth uses `E67_Birth` with `P98_brought_into_life`
- Death uses `E69_Death` with `P100_was_death_of`
- Marriage uses `E85_Joining` + `E74_Group` (not a direct relationship between people)
- Group membership uses `P107_has_current_or_former_member`
- Gender assigned via `E13_Attribute_Assignment` (not a direct property)

### Category 7: Measurement Pattern (Census/Statistical Data)

Check that:
- Measurements use `E16_Measurement` (not `E13_Attribute_Assignment`)
- Each measurement connects: `E16 → P39_measured → target`, `E16 → P40_observed_dimension → E54`, `E54 → P91_has_unit → E58`
- `E54_Dimension` has `P90_has_value` with a numeric XSD datatype (`^^xsd:integer` or `^^xsd:decimal`)
- `E54_Dimension` has `P91_has_unit` pointing to a valid `E58_Measurement_Unit` node (not a bare string)
- Variable classification via `E16 → P2_has_type → E55_Type`
- Provenance via `E73 → P70_documents → E16`

**FAIL if**: `E54_Dimension` is missing `P90_has_value` (measurement has no numeric result).
**FAIL if**: `E54_Dimension` is missing `P91_has_unit` (measurement has no unit — renders data uninterpretable).
**FAIL if**: `P90_has_value` uses `^^xsd:string` instead of a numeric datatype.
**PASS if**: Measurement chain is complete with provenance and correctly typed values.

### Category 8: Document Modeling

Check that:
- Conceptual Content/Immaterial Objects use `crm:E73_Information_Object`
- Digital surrogates use `crmdig:D1_Digital_Object`
- Digital-to-physical link uses `P130_shows_features_of` or `P130i_features_are_also_found_on`
- Subject link uses `P129_is_about`
- Mention link uses `P67_refers_to`
- Text annotations use OA Web Annotation model (`oa:Annotation`, `oa:hasTarget`, `oa:hasBody`)

### Category 9: Neo4j ↔ RDF Translation Issues

If validating a Neo4j property graph, flag:
- **Relationship properties**: Any edge with attributes (not expressible in basic RDF)
- **Custom relationship types**: Any type not in CRM/CRMdig specs
- **Missing namespaces**: Node labels without proper URI prefixes
- **Bare string properties**: Values that should route through E33_E41 or E42 patterns
- **Multiple labels**: Neo4j multi-label nodes need explicit `rdf:type` for each class

### Category 10: Dangling URI Check

If validating generated RDF/Turtle output, check for **referential integrity**:

- Every local URI (using project prefix or temporary/LINCS minted URI) that appears in **object position** of a triple must also appear as a **subject** with an `rdf:type` declaration somewhere in the output.
- External authority URIs (`geonames:`, `viaf:`, `wikidata:`, `aat:`, `lexvo:`) are exempt — they are defined externally.
- URIs for `E52_Time-Span` and `E54_Dimension` nodes should be minted.

**Example of a dangling URI (FAIL)**:
```turtle
# This measurement references a presence that is never defined
<http://temp.lincsproject.ca/datasetID/MEAS_1 a crm:E16_Measurement ;
    crm:P39_measured <http://temp.lincsproject.ca/datasetID/ON082003_1871 .  # <-- DANGLING: <http://temp.lincsproject.ca/datasetID/ON082003_1871 has no rdf:type triple
```

**Corrected**:
```turtle
<http://temp.lincsproject.ca/datasetID/MEAS_1 a crm:E16_Measurement ;
    crm:P39_measured <http://temp.lincsproject.ca/datasetID/ON082003_1871 .

<http://temp.lincsproject.ca/datasetID/ON082003_1871 a crm:E93_Presence ;   # <-- Now defined
    rdfs:label "Westmeath presence (1871)"@en .
```

**FAIL if**: Any local URI referenced in object position has no `rdf:type` declaration as a subject.
**WARN if**: A local URI is defined with `rdf:type` but is missing `rdfs:label`.


### Category 11: No Blank Nodes

Check that:
- All RDF subjects and objects are **named URIs** (with a namespace prefix or full URI)
- No blank nodes (anonymous nodes like `_:b1`, `_:node123`) appear anywhere in the output
- Intermediate nodes (E52_Time-Span, E54_Dimension, E93_Presence, etc.) must have minted URIs using your project namespace

**Why**: Blank nodes are not dereferenceable and cannot be linked across federated datasets. LINCS data must be fully interoperable with external knowledge graphs. (This is also a limitation of Researchspace)

**Example of invalid blank nodes (FAIL)**:
```turtle
<http://temp.lincsproject.ca/datasetID/MEAS_1 a crm:E16_Measurement ;
    crm:P40_observed_dimension [ 
      a crm:E54_Dimension ;
      crm:P90_has_value 1000 ;
      crm:P91_has_unit <http://temp.lincsproject.ca/datasetID/UNIT_PERSONS
    ] .
```

The `[ ... ]` syntax creates an anonymous blank node that cannot be referenced elsewhere.

**Corrected**:
```turtle
<http://temp.lincsproject.ca/datasetID/MEAS_1 a crm:E16_Measurement ;
    crm:P40_observed_dimension <http://temp.lincsproject.ca/datasetID/DIM_POP_1871_WEST .

<http://temp.lincsproject.ca/datasetID/DIM_POP_1871_WEST a crm:E54_Dimension ;
    rdfs:label "Population of Westmeath (1871)"@en ;
    crm:P90_has_value "1000"^^xsd:integer ;
    crm:P91_has_unit <http://temp.lincsproject.ca/datasetID/UNIT_PERSONS .
```

**FAIL if**: Any blank node (including intermediate nodes for time-spans, dimensions, presences, or attribute assignments) appears in the RDF.
**Remediation**: Mint a URI for every intermediate node using your projects namespace URI (e.g., `temp.lincsproject.ca/montreal/population/dimension/westmeath`). Use a consistent URI scheme (e.g., `temp.lincsproject.ca/{dataset}/{types}`). 

### Category 12: Incorrect Domain/Range

Check every triple against the LINCS Property Reference Table. This is the most critical check to ontological correctness.

**FAIL if**:
- The Subject is not an instance of the property's defined Domain or one of its subclasses.
- The Object is not an instance of the property's defined Range or one of its subclasses.

**WARN if**:
- A property is used that is not in the approved LINCS list.
- A class is used that is not in the approved LINCS list.


### Category 13: No Deprecated Classes or Properties

Check that no deprecated classes or properties are used. 

| Deprecated Class | Migration Instruction |
|------------------|-----------------------|
| E38_Image | use E36_Visual_Item |
| E40_Legal_Body | use E74_Group |
| E44_Place_Appellation | use E41_Appellation |
| E45_Address | use E41_Appellation, P2 has type: “Address” |
| E46_Section_Definition | use E41_Appellation |
| E47_Spatial_Coordinates | use E94_Space_Primitive |
| E48_Place_Name | use E41_Appellation |
| E49_Time_Appellation | use E41_Appellation |
| E50_Date | use E61_Time_Primitive |
| E51_Contact_Point | use E41_Appellation, P2 has type: “Contact Point” |
| E75_Conceptual_Object_Appellation | use E41_Appellation |
| E82_Actor_Appellation | use E41_Appellation |
| E84_Information_Carrier | use E22_Human-Made_Object, P2 has type: “Information Carrier” |

**Deprecated Properties**:
- P58_has_section_definition
- P78_is_identified_by
- P88_consists_of
- P83_had_at_least_duration
- P84_had_at_most_duration
- P87_is_identified_by
- P114_is_equal_in_time_to
- P115_finishes
- P116_starts 
- P117_occurs_during
- P118_overlaps_in_time_with
- P119_meets_in_time_with
- P120_occurs_before 
- P131_is_identified_by
- P149_is_identified_by
- P178_ends_after_or_with
- P181_has_amount

**FAIL if**: Any deprecated class or property is used.

### Output Format

```
## LINCS Validation Report

### Summary
- PASS: N checks
- WARN: N checks
- FAIL: N checks

### Findings

#### [PASS/WARN/FAIL] Category Name
Description of finding.
**Remediation**: Specific steps to fix.
```

Prioritize FAIL items first, then WARN. For each finding, provide a concrete Turtle/Cypher example showing the current state and the corrected version.

**Categories**: 1. Namespace Correctness, 2. Authority URIs, 3. Required Properties, 4. Temporal Modeling, 5. Spatial Modeling, 6. Event-Centric Patterns, 7. Measurement Pattern, 8. Document Modeling, 9. Neo4j ↔ RDF Translation, 10. Dangling URI Check
