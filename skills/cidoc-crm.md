---
name: cidoc-crm
description: CIDOC-CRM ontology reference for cultural heritage knowledge graphs. Use when modeling entities, choosing classes/properties, or checking ontology correctness. Covers core CRM v7.1.3, and CRMdig extensions.
user-invocable: true
allowed-tools: Read, Grep, Glob, WebFetch
argument-hint: [class-or-property-or-question]
---

# CIDOC-CRM Ontology Reference

You are a CIDOC-CRM ontology expert. When the user asks about classes, properties, or modeling patterns, provide accurate guidance based on the reference below.

## Output Constraints

When generating data models, always default to valid **RDF Turtle (`.ttl`)** syntax unless asked otherwise. Always include standard prefixes (`rdf:`, `rdfs:`, `owl:`, `xsd:`, `skos:`, `crm:`, `crmdig:`). Every generated entity MUST have at least one `rdfs:label` and at least one `rdf:type`. Never attach dates or places directly to an entity — always route through an event node (this is CIDOC-CRM's event-centric pattern, not schema.org's flat model).

## Query: $ARGUMENTS

## Core Ontology: CIDOC-CRM v7.1.3

Namespace: `http://www.cidoc-crm.org/cidoc-crm/` (prefix: `crm:`)
ISO Standard: ISO 21127
Current Version: 7.1.3 (there is NO version 2.0)

### CIDOC CRM Classes that LINCS Uses

| Class | Label | Scope |
|-------|-------|-------|
|  E1_CRM_Entity  |  CRM Entity  |  Top-level abstract class  |
|  E2_Temporal_Entity  |  Temporal Entity  |  Things with temporal extent  |
|  E4_Period  |  Period  |  Named historical periods  |
|  E5_Event  |  Event  |  Something that happened  |
|  E7_Activity  |  Activity  |  Intentional human action  |
|  E8_Acquisition | Acquisition |  Transfer of legal ownership between people or groups |
|  E9_Move | Move |  Change in physical location of physical objects |
|  E12_Production  |  Production  |  Creation of physical things  |
|  E13_Attribute_Assignment  |  Attribute Assignment  |  Scholarly assertion about an attribute. Use when extracting uncertain facts from noisy sources (OCR, ambiguous records). Allows attaching provenance and certainty to the assertion itself, not just the value.  |
|  E18_Physical_Thing | Physical Thing |  Persistent physical item with a relatively stable form, human made or natural  |
|  E19_Physical_Object | Physical Object |  Sub-type of Physical Thing. Physical objects that take up space with concrete boundaries that separate them from other objects. Includes biological objects and human made objects. Typically the objects can be moved. |
|  E21_Person  |  Person  |  Individual human being  |
|  E22_Human-Made_Object | Human-Made Object |  Persistent physical object purposely created by human activity |
|  E24_Physical_Human-Made_Thing | Physical Human-Made Thing |  Includes human-made objects like a sword and human-made features such as rock art. |
|  E28_Conceptual_Object | Conceptual Object |  Non-material products of human minds |
|  E30_Right  |  Right  |  Legal entitlement  |
|  E33_Linguistic_Object  |  Linguistic Object  |  Identifiable expression in language  |
|  E33_E41_Linguistic_Appellation  |  Linguistic Appellation  |  A name expressed in language  |
|  E39_Actor  |  Actor  |  Person or group  |
|  E41_Appellation  |  Appellation  |  Name used to identify something  |
|  E42_Identifier  |  Identifier  |  Assigned code/number  |
|  E52_Time-Span  |  Time-Span  |  Temporal extent with begin/end  |
|  E53_Place  |  Place  |  Persistent spatial extent  |
|  E54_Dimension  |  Dimension  |  Quantitative measurement value  |
|  E55_Type  |  Type  |  Classification concept  |
|  E56_Language  |  Language  |  A natural language  |
|  E57_Material |  Material |  A physical material |
|  E58_Measurement_Unit  |  Measurement Unit  |  Unit of measure  |
|  E65_Creation  |  Creation  |  Intellectual creation event  |
|  E66_Formation  |  Formation  |  Establishment of a group  |
|  E67_Birth  |  Birth  |  Birth of a person  |
|  E68_Dissolution  |  Dissolution  |  End of a group  |
|  E69_Death  |  Death  |  Death of a person  |
|  E73_Information_Object  |  Information Object  |  Immaterial item with propositional content  |
|  E74_Group  |  Group  |  Collection of persons  |
|  E77_Persistent_Item  |  Persistent Item  |  Enduring entity  |
|  E78_Curated_Holding  |  Curated Holding  |  Collection managed together  |
|  E85_Joining  |  Joining  |  Becoming a member of a group  |
|  E86_Leaving  |  Leaving  |  Ceasing membership in a group  |
|  E89_Propositional_Object |  Propositional Object |  Immaterial items such as stories, plots, procedures, algorhtms, laws of physics |
|  E92_Spacetime_Volume |  Spacetime Volume |  4-dimensional volumes in physical spacetime |
|  E93_Presence |  Presence |  Spacetime Volume during a by a timespan.  |
|  E97_Monetary_Amount |  Monetary Amount |  Quantities of monetary possessions or oblications |
|  E99_Product_Type  |  Type  |  Type of product. Sub class of E55  |
|  PC14_carried_out_by |  Carried Out By |  Property class used to say what role an actor had in an event |

### Properties Used by LINCS

| Property | Domain | Range | Meaning |
|----------|--------|-------|---------|
|  P01_has_domain | PC14 | E1 | Used to define an actors participation in an event, with a PC14 carried out by |
|  P02_has_range | PC14 | E1 | Used to define an actors participation in an event, with a PC14 carried out by |
|  P1_is_identified_by  |  E1  |  E41/E42  |  Entity has this name/identifier. Use E41_Appellation (or E33_E41) for human-readable names (strings). Use E42_Identifier for formal codes, database IDs, and URIs.  |
|  P2_has_type  |  E1  |  E55  |  Entity is classified as  |
|  P3_has_note | E1 |  xsd:string  | Attach a general note |
|  P4_has_time-span  |  E2  |  E52  |  Temporal entity has this time extent  |
|  P7_took_place_at  |  E4  |  E53  |  Event/period occurred at this place  |
|  P9_consists_of | E4 | E4 | Declare that an activity took place as part of another activity. |
|  P10_falls_within  | E92 | E92 | A spacetime volume falls within another spacetime volume |
|  P11_had_participant  |  E5  |  E39  |  Event involved this actor  |
|  P14_carried_out_by  |  E7  |  E39  |  Activity done by this actor  |
|  P14.1_in_the_role_of | PC14 | E55 | Used to define an actors role in an event, with a PC14 carried out by |
|  P16_used_specific_object | E7 | E70 | Used to add specifics to assertions and attribution assignments, such as gender identity |
|  P22_transferred_title_to | E8 | E39 | Defines an actor in an acquisition |
|  P23_transferred_title_from | E8 | E39 | Defines an actor in an acquisition |
|  P24_transferred_title_of | E8 | E18 | Defines the object in an acquisition |
|  P25_moved | E9 | E19 | A physical object moves physical locations through an E9 move |
|  P26_moved_to | E9 | E53 | A physical object's new location |
|  P27_moved_from | E9 | E53 | A physical object's old location |
|  P43_has_dimension | E70 | E54 | Connects an object to information about dimensions such as measurements |
|  P45_consists_of | E18 | E57 | Defines the material an object is made of |
|  P46_is_composed_of | E18 | E18 | A physical thing forms part of another physical thing |
|  P49_has_former_or_current_keeper | E18 | E39 | Connect an object directly to current or previous keeper without an acquisition event |
|  P51_has_former_or_current_owner | E18 | E39 | Connect an object directly to current or previous owner without an acquisition event |
|  P52_has_current_owner | E18 | E39 | Connect an object directly to current  owner without an acquisition event |
|  P53_has_former_or_current_location | E18 | E53 | Connect something direclty to a current or former location without an event |
|  P67_refers_to  |  E89  |  E1  |  Information object mentions entity  |
|  P72_has_language  |  E33  |  E56  |  Written/spoken in this language  |
|  P74_has_current_or_former_residence | E19 | E53 | Connect an actor to a current or former residence without a move event |
|  P75_possesses | E39 | E30 | Connect rigths to the holder |
|  P76_has_contact_point | E39 | E41 | Attach an address to a group |
|  P82_at_some_time_within  |  E52  |  xsd:string  |  Human-readable date label  |
|  P82a_begin_of_the_begin  |  E52  |  xsd:dateTime  |  Earliest possible start. Use `xsd:date` if time unknown, but ensure valid ISO 8601 (e.g., `"1871-01-01"^^xsd:date`). Use `xsd:dateTime` when time is known (e.g., `"1871-01-01T00:00:00"^^xsd:dateTime`).  |
|  P82b_end_of_the_end  |  E52  |  xsd:dateTime  |  Latest possible end. Same datatype rules as P82a.  |
|  P86_falls_within  |  E52  |  E52  |  Time-span is within another  |
|  P89_falls_within  |  E53  |  E53  |  Place is spatially within another  |
|  P90_has_value | E54 |  xsd:string  | The string value of a dimension |
|  P91_has_unit  |  E54  |  E58  |  Dimension measured in this unit  |
|  P94_has_created  |  E65  |  E28  |  Creation event produced this  |
|  P96_by_mother | E67 | E21 | Connect a birth event to a mother |
|  P97_from_father | E67 | E21 | Connect a birth event to a father |
|  P98_brought_into_life  |  E67  |  E21  |  Birth brought this person into life  |
|  P99_dissolved | E68 | E74 | Connect a group to a dissolution event |
|  P100_was_death_of  |  E69  |  E21  |  Death was the death of this person  |
|  P104_is_subject_to  |  E72  |  E30  |  Entity is subject to this right  |
|  P106_is_composed_of  |  E90  |  E90  |  Composed of parts  |
|  P107_has_current_or_former_member  |  E74  |  E39  |  Group includes/included this actor  |
|  P108_has_produced | E12 | E24 | The output of a production event |
|  P121_overlaps_with | E53 | E53 | A place geometrically overlaps with another place |
|  P122_borders | E53 | E53 | A place borders another place |
|  P127_has_broader_term  |  E55  |  E55  |  Type hierarchy (narrower → broader)  |
|  P129_is_about  |  E89  |  E1  |  Information object is about entity  |
|  P130_shows_features_of  |  E70  |  E70  |  Digital copy shows features of original  |
|  P132_spatiotemporally_overlaps_with | E92 | E92 | Two spacetime volumnes have some of their extents in common. A sub property of P10 falls within. |
|  P138_represents | E36 | E1 | Used to say what a visual item represents |
|  P140_assigned_attribute_to  |  E13  |  E1  |  Attribute assignment target  |
|  P141_assigned  |  E13  |  E1  |  Attribute value assigned  |
|  P143_joined  |  E85  |  E39  |  Joining added this actor  |
|  P144_joined_with  |  E85  |  E74  |  Joining added to this group  |
|  P148_has_component | E89 | E89 | Components of expressions, works, and information objects |
|  P156_occupies | E18 | E53 | A physical thing fills a place |
|  P160_has_temporal_projetion | E92 | E55 | The temporal projection of a spacetime volume. Used to connect a timespand to a spacetime volume or presence. |
|  P161_has_spatial_projection | E92 | E53 | The spatial projection of a spacetime volume. Used to connect a place to a spacetime volume or presence. |
|  P166_was_a_presence_of | E93 | E92 | Used to connect an event to a presence. |
|  P168_place_is_defined_by  |  E53  |  E94  | Use `"POINT(lon lat)"@en`, For polygons: `"POLYGON((lon1 lat1, lon2 lat2, ...))"@en`. |
|  P186_produced_thing_of_product_type | E12 | E99 | Production of a certain type of product |
|  P190_has_symbolic_content  |  E90  |  xsd:string  |  Literal string value  |


## Golden Pattern: The Event-Centric Model

CIDOC-CRM is event-centric: dates and places attach to **events**, not directly to entities. This is the single most important pattern to internalize. LLMs trained on schema.org will instinctively put `birthDate` directly on a Person — this is WRONG in CIDOC-CRM.

### Example: A Person's Birth

```turtle
@prefix crm: <http://www.cidoc-crm.org/cidoc-crm/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix geonames: <http://sws.geonames.org/> .

# The person
<http://temp.lincsproject.ca/datasetID/person/louis_riel> a crm:E21_Person ;
    rdfs:label "Louis Riel"@en ;
    crm:P1_is_identified_by <http://temp.lincsproject.ca/dataset/louis_riel/name/louis_riel> .

<http://temp.lincsproject.ca/datasetID/person/louis_riel/name/louis_riel> a crm:E33_E41_Linguistic_Appellation ;
        rdfs:label "Name of Louis Riel"@en ;
        crm:P190_has_symbolic_content "Louis Riel"@en ;
        crm:P2_has_type <http://id.lincsproject.ca/biography/personalName> .

# The name type
<http://id.lincsproject.ca/biography/personalName> a crm:E55_Type;
        rdfs:label "Personal Name"@en .

# The birth EVENT — dates and places attach here, not on the person
<http://temp.lincsproject.ca/datasetID/person/louis_riel/birth_event> a crm:E67_Birth ;
    rdfs:label "Birth of Louis Riel"@en ;
    crm:P98_brought_into_life <http://temp.lincsproject.ca/dataset/person/1234> ;
    crm:P7_took_place_at geonames:6183235/ ;          # Red River Settlement
    crm:P4_has_time-span <http://temp.lincsproject.ca/datasetID/person/riel_birth_event/timespan> .

<http://temp.lincsproject.ca/datasetID/person/louis_riel/birth_event/timespan> a crm:E52_Time-Span ;
    rdfs:label "Date of birth of Louis Riel"@en ;
    crm:P82_at_some_time_within "1844-10-22"@en ;
    crm:P82a_begin_of_the_begin "1844-10-22T00:00:00"^^xsd:dateTime ;
    crm:P82b_end_of_the_end "1844-10-22T23:59:59"^^xsd:dateTime . 

# The place
    geonames:6183235/ a crm:E53_Place ;
    rdfs:label "Red River Settlement"@en ;
    crm:P89_falls_within geonames:6065171/ .           # Manitoba
```

**Key structural rules visible in this example:**
1. Person has NO `birthDate` or `birthPlace` property — those go on the Birth event
2. Every node has `rdfs:label`
3. Time-span has all three date properties (P82, P82a, P82b)
4. When an external identifier is provided in the data, use it as the entity's URI (e.g., geonames URIs were available in the source data in this example). Do not make up an external identifer.
5. When no external URIs are provided, create temporary URIs with the prefix `http://temp.lincsproject.ca/`. They should be understandable to a human and uniquely identify the entity based on the information in the provided data.
5. Names route through E33_E41 with P190 for the literal string. They can optionally has more specific types like Personal Name in this example.

### Example: A Census Measurement

```turtle
@prefix crm: <http://www.cidoc-crm.org/cidoc-crm/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .


# The measurement EVENT — the value attaches here, not on the place
<http://temp.lincsproject.ca/datasetID/MEAS_ON082003_1871_POP> a crm:E16_Measurement ;
    rdfs:label "Population measurement of Westmeath (1871)"@en ;
    crm:P39_measured <http://temp.lincsproject.ca/datasetID/ON082003_1871> ;         # What was measured (presence)
    crm:P2_has_type <http://temp.lincsproject.ca/datasetID/VAR_POP_XX_N> ;           # What variable
    crm:P40_observed_dimension <http://temp.lincsproject.ca/datasetID/MEAS_ON082003_1871_POP/dimension> ;
    crm:P4_has_time-span  <http://temp.lincsproject.ca/datasetID/MEAS_ON082003_1871_POP/timespan>.

<http://temp.lincsproject.ca/datasetID/MEAS_ON082003_1871_POP/dimension> a crm:E54_Dimension ;
    rdfs:label "2632"@en ;
    crm:P90_has_value "2632"^^xsd:integer ;
    crm:P91_has_unit <http://temp.lincsproject.ca/datasetID/UNIT_PERSONS>.

<http://temp.lincsproject.ca/datasetID/MEAS_ON082003_1871_POP/timespan> a crm:E52_Time-Span ;
    rdfs:label "1871"@en ;
    crm:P82_at_some_time_within "1871"^^xsd:string ;
    crm:P82a_begin_of_the_begin "1871-01-01T00:00:00"^^xsd:dateTime ;
    crm:P82b_end_of_the_end "1871-12-31T23:59:59"^^xsd:dateTime .


# The temporal presence
<http://temp.lincsproject.ca/datasetID/ON082003_1871> a crm:E93_Presence ;
    rdfs:label "Westmeath presence (1871)"@en ;
    crm:P166_was_a_presence_of <http://temp.lincsproject.ca/datasetID/PLACE_ON142032> .

# The enduring place
<http://temp.lincsproject.ca/datasetID/PLACE_ON142032> a crm:E53_Place ;
    rdfs:label "Westmeath"@en ;
    owl:sameAs <http://sws.geonames.org/5914691/> .
```


### Example: A Farm Produced Wheat in Halifax in 1871
```turtle
@prefix crm: <http://www.cidoc-crm.org/cidoc-crm/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix temp_lincs: <http://temp.lincsproject.ca/>

temp_lincs:farm_type a crm:E55_Type ;
    rdfs:label "Farm"@en .

temp_lincs:wheat_type a crm:E99_Product_Type ;
    rdfs:label "Wheat"@en .

temp_lincs:bushel_type a crm:E58_Measurement_Unit ;
    rdfs:label "Bushel"@en .

temp_lincs:family1 a crm:E74_Group ;
    rdfs:label "Family that owned farm1"@en .


# if this was a house, it would be E19_Physical_Object
# since it is a farm, it is E18_Physical_Thing
temp_lincs:farm1 a crm:E18_Physical_Thing ;
    rdfs:label "A farm in Halifax"@en ;
    crm:P2_has_type temp_lincs:farm_type ;
    crm:P51_has_former_or_current_owner temp_lincs:family1 ;
    crm:P156_occupies temp_lincs:farm1_location .

temp_lincs:production_wheat_1871_farm1 a crm:E12_Production ;
    rdfs:label "Production of Wheat by Farm1 in 1871"@en ;
    crm:P16_used_specific_object temp_lincs:farm1 ;
    crm:P14_carried_out_by temp_lincs:family1 ;
    crm:P186_produced_thing_of_product_type temp_lincs:wheat_type ;
    crm:P108_has_produced temp_lincs:400_bushels_wheat ;
    crm:P4_has_time-span temp_lincs:wheat_production_1871_timespan .

temp_lincs:400_bushels_wheat a crm:E19_Physical_Object ;
    rdfs:label "400 Bushels of Wheat"@en ;
    crm:P2_has_type temp_lincs:wheat_type ;
    crm:P43_has_dimension temp_lincs:400_bushells_wheat_dimension .

temp_lincs:400_bushells_wheat_dimension a crm:E54_Dimension ;
    rdfs:label "Number of bushels"@en ;
    crm:P91_has_unit temp_lincs:bushel_type ;
    crm:P90_has_value "400"xsd:integer .

temp_lincs:wheat_production_1871_timespan a crm:E52_Time-Span ;
    rdfs:label "Timespan of farm1 in halifax producing wheat in 1871"@en ;
    crm:P82_at_some_time_within "1871"@en ;
    crm:P82a_begin_of_the_begin "1871-01-01T00:00:00"^^xsd:dateTime ;
    crm:P82b_end_of_the_end "1871-12-31T23:59:59"^^xsd:dateTime .

temp_lincs:farm1_presence_1871 a crm:E93_Presence ;
    rdfs:label "Farm1 presence in 1871"@en ;
    crm:P195_was_a_presence_of temp_lincs:farm1
    crm:P160_has_temporal_projection temp_lincs:wheat_production_1871_timespan ;
    crm:P161_has_spatial_projection temp_lincs:farm1_location ;
    crm:P10_falls_within temp_lincs:halifax_presence_1871 .

temp_lincs:halifax_presence_1871 a crm:E93_Presence ;
    rdfs:label "Halifax in 1871"@en ;
    crm:P160_has_temporal_projection temp_lincs:wheat_production_1871_timespan ;
    crm:P161_has_spatial_projection temp_lincs:halifax .

temp_lincs:farm1_location a crm:E53_Place ;
    rdfs:label "Farm1 location"@en ;
    crm:P89_falls_within temp_lincs:halifax .

temp_lincs:halifax a crm:E53_Place ;
    rdfs:label "Halifax"@en .

```


### When to Use E93_Presence vs. E53_Place

Use E93_Presence when:
- Place boundaries change over time and you need to track each configuration
- You need to record spatial overlap between temporal snapshots
- You're building a historical GIS with time-scoped geometries

Use Core CRM (E53_Place + P168 + P89) when:
- Places are treated as stable points with fixed coordinates
- Spatial hierarchy is static (no boundary changes)
- Interoperability with other sources is a priority. There is a shared global understanding of the idea of the place and you want to connect to a shared identifier.

## Extension: CRMdig (Digital)

Namespace: `http://www.ics.forth.gr/isl/CRMdig/` (prefix: `crmdig:`)

| Class | Description |
|-------|-------------|
| D1_Digital_Object | Digital surrogate of physical or conceptual item |

## Common Anti-Patterns (Things to Avoid)

1. **Properties on relationships in RDF**: CRM properties (P89, P122, etc.) are simple triples in RDF. They cannot carry their own attributes (like `during_period` or `border_length`). Use reification (e.g., PC14) or route through events instead.

2. **Confusing E4_Period with E52_Time-Span**: E4_Period is a named historical period (e.g., "The Victorian Era"). E52_Time-Span is a date range attached to events via P4. Don't use E4_Period as a general temporal index. A given E52_Time-Span is typically only connected to a single event.

3. **Skipping E52_Time-Span**: Always attach dates via `E52_Time-Span` with `P82`/`P82a`/`P82b` bounds. Don't put date properties directly on events or entities.

4. **Missing rdfs:label**: Every entity should have at least one `rdfs:label` for discoverability and tooling compatibility. When provided in the data, attach a language tag like `@en` or `@fr`.

5. **Internal-only identifiers**: When present in the data, use external authority URIs (GeoNames, VIAF, Wikidata) alongside internal IDs for interoperability. When no URI is provided, do not make up an authority URI, instead create an `http://temp.lincsproject.ca/` URI.

6. **Using E13 when E16 is appropriate**: E13_Attribute_Assignment is for scholarly interpretations and uncertain attributions. E16_Measurement is for empirical observations with quantitative results. Census counts are E16, not E13.

7. **Flat schema.org-style modeling**: Never put `birthDate`, `birthPlace`, `occupation`, or similar properties directly on a Person or Place node. CIDOC-CRM is event-centric: create an event node (E67_Birth, E7_Activity, etc.) and attach dates/places to the event. See the Golden Pattern examples above.

8. **E55_Type without vocabulary grounding**: Don't create ad-hoc E55_Type nodes with only internal labels. Ground types in external vocabularies provided in the data: Wikidata QIDs, Getty AAT, or LINCS SKOS vocabularies. When using E55_Type for categorical/taxonomic data, also declare the node as `skos:Concept` to enable interoperability with SKOS-based thesauri:
    ```turtle
    <http://temp.lincsproject.ca/datasetID/VAR_POP_XX_N> a crm:E55_Type, skos:Concept ;
        rdfs:label "Total population"@en ;
        skos:prefLabel "Total population"@en ;
        skos:broader <http://temp.lincsproject.ca/datasetID/VAR_POPULATION> ;
        crm:P127_has_broader_term <http://temp.lincsproject.ca/datasetID/VAR_POPULATION> .
    ```
    Use both `skos:broader` and `crm:P127_has_broader_term` when bridging CRM with external thesauri.

## Handling Uncertainty and Noisy Data

When building knowledge graphs from primary historical sources (OCR'd documents, ambiguous records, contested attributions), use E13_Attribute_Assignment to wrap the uncertain assertion. This separates the claim from the evidence, allowing consumers to assess reliability.

### Pattern: Uncertain Birth Date (from OCR'd Census)

```turtle
# The assertion itself is uncertain
<http://temp.lincsproject.ca/datasetID/riel_birth_event/birth_date_riel_assignment> a crm:E13_Attribute_Assignment ;
    rdfs:label "Uncertain birth date assignment for Louis Riel"@en ;
    crm:P140_assigned_attribute_to <http://temp.lincsproject.ca/datasetID/riel_birth_event> ;
    crm:P141_assigned <http://temp.lincsproject.ca/datasetID/riel_birth_event/birth_date_riel_assignment/timespan> ;
    crm:P2_has_type edit:certaintyUnknown ;
    crm:P17_was_motivated_by <http://temp.lincsproject.ca/datasetID/riel_birth_event/birth_date_riel_assignment/motivation> .

# Not sure if this should be the same timespan from the first example if so replace with `<http://temp.lincsproject.ca/datasetID/riel_birth_event/timespan>` - data is different here than there.
<http://temp.lincsproject.ca/datasetID/riel_birth_event/birth_date_riel_assignment/timespan> a crm:E52_Time-Span ;
    rdfs:label "circa 1844"@en ;
    crm:P82_at_some_time_within "c. 1844"@en ;
    crm:P82a_begin_of_the_begin "1843-01-01T00:00:00"^^xsd:dateTime ;
    crm:P82b_end_of_the_end "1845-12-31T23:59:59"^^xsd:dateTime .


<http://temp.lincsproject.ca/datasetID/riel_birth_event/birth_date_riel_assignment/motivation> a crm:E73_Information_Object ;
    rdfs:label "OCR'd census record (poor quality)"@en .

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

1. **Is it a person?** → E21_Person
2. **Is it a group/organization?** → E74_Group
3. **Is it a place?** → E53_Place
4. **Does the place change over time?** → Add E93_Presence per period
5. **Is it something that happened?** → E7_Activity or more specific event subclass
6. **Is it a birth/death?** → E67_Birth / E69_Death
7. **Is it a group forming/ending?** → E66_Formation / E68_Dissolution
8. **Is it someone joining/leaving a group?** → E85_Joining / E86_Leaving
9. **Is it a measurement/observation?** → E16_Measurement → E54_Dimension + E58_Measurement_Unit
10. **Is it a document/text?** → E33_Linguistic_Object or E73_Information_Object
11. **Is it a digital file?** → CRMdig:D1_Digital_Object
12. **Is it a classification/category?** → E55_Type
13. **Is it a name?** → E33_E41_Linguistic_Appellation with P190 for string value
14. **Is it an uncertain assertion?** → E13_Attribute_Assignment wrapping the pattern
