---
name: lincs-sparql
description: Build and explain SPARQL queries for the LINCS triplestore (fuseki.lincsproject.ca). Covers Historical Canadian Persons, Indian Affairs Agents, and Cabinet Conclusions datasets. Also supports querying local CIDOC-CRM data.
user-invocable: true
allowed-tools: Read, Grep, Glob, WebFetch
argument-hint: [what-to-query]
---

# LINCS SPARQL Query Builder

Build SPARQL queries for the LINCS endpoint or local CIDOC-CRM data.

## Query Goal: $ARGUMENTS

## LINCS SPARQL Endpoint

**URL**: `https://fuseki.lincsproject.ca/lincs/sparql`
**Method**: POST
**Content-Type**: `application/sparql-query`

## Named Graphs

```sparql
# Historical Canadian Persons
GRAPH <http://graph.lincsproject.ca/hist-canada/hist-cdns>

# Historical Indian Affairs Agents
GRAPH <http://graph.lincsproject.ca/hist-canada/ind-affairs>

# Cabinet Conclusions
GRAPH <http://graph.lincsproject.ca/hist-canada/cabinet-conclusions>
```

## Standard Prefixes

```sparql
PREFIX crm: <http://www.cidoc-crm.org/cidoc-crm/>
PREFIX crmdig: <http://www.ics.forth.gr/isl/CRMdig/>
PREFIX oa: <http://www.w3.org/ns/oa#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX geonames: <http://sws.geonames.org/>
PREFIX viaf: <http://viaf.org/viaf/>
PREFIX wikidata: <http://www.wikidata.org/entity/>
PREFIX lincs: <http://id.lincsproject.ca/>
PREFIX biography: <http://id.lincsproject.ca/biography/>
PREFIX event: <http://id.lincsproject.ca/event/>
PREFIX identity: <http://id.lincsproject.ca/identity/>
PREFIX occupation: <http://id.lincsproject.ca/occupation/>
PREFIX aat: <http://vocab.getty.edu/aat/>
PREFIX lexvo: <http://lexvo.org/id/iso639-3/>
PREFIX base: <http://example.org/chgis/>
# NOTE: Replace base: with actual project URI for production queries
```

## Query Templates

### 1. Explore Dataset Classes
```sparql
SELECT ?class (COUNT(?s) AS ?count)
WHERE {
  GRAPH <http://graph.lincsproject.ca/hist-canada/hist-cdns> {
    ?s a ?class .
  }
}
GROUP BY ?class
ORDER BY DESC(?count)
```

### 2. Find a Person by Name and additional information about their birth if available
```sparql
SELECT ?person ?name ?birthDate ?birthPlace ?birthPlaceLabel
WHERE {
  GRAPH <http://graph.lincsproject.ca/hist-canada/hist-cdns> {
    ?person a crm:E21_Person ;
            crm:P1_is_identified_by ?nameNode .
    ?nameNode crm:P190_has_symbolic_content ?name .
    FILTER(CONTAINS(LCASE(?name), "macdonald"))

    OPTIONAL {
      ?birth crm:P98_brought_into_life ?person ;
             crm:P4_has_time-span ?bts .
      ?bts crm:P82_at_some_time_within ?birthDate .
    }
    OPTIONAL {
      ?birth crm:P7_took_place_at ?birthPlace .
      ?birthPlace rdfs:label ?birthPlaceLabel .
    }
  }
}
LIMIT 20
```

### 3. Person's Full Biography (Events, Occupations, Relationships)
```sparql

# Replace viaf:XXXXXXXXX with valid person URI
SELECT ?person ?name ?eventType ?eventLabel ?date ?place ?placeLabel
WHERE {
  GRAPH <http://graph.lincsproject.ca/hist-canada/hist-cdns> {
    ?person a crm:E21_Person ;
            rdfs:label ?name .
    FILTER(?person = viaf:XXXXXXXXX)

    {
      # Birth
      ?event crm:P98_brought_into_life ?person .
      BIND("Birth" AS ?eventType)
    } UNION {
      # Death
      ?event crm:P100_was_death_of ?person .
      BIND("Death" AS ?eventType)
    } UNION {
      # Occupation
      ?event a crm:E7_Activity ;
             crm:P14_carried_out_by ?person ;
             crm:P2_has_type event:OccupationEvent .
      BIND("Occupation" AS ?eventType)
    } UNION {
      # Marriage
      ?event a crm:E85_Joining ;
             crm:P143_joined ?person .
      BIND("Marriage" AS ?eventType)
    }

    ?event rdfs:label ?eventLabel .

    OPTIONAL {
      ?event crm:P4_has_time-span ?ts .
      ?ts crm:P82_at_some_time_within ?date .
    }
    OPTIONAL {
      ?event crm:P7_took_place_at ?place .
      ?place rdfs:label ?placeLabel .
    }
  }
}
ORDER BY ?date
```

### 4. People Associated with a Place (via GeoNames)
```sparql
SELECT ?person ?name ?eventType ?date
WHERE {
  GRAPH <http://graph.lincsproject.ca/hist-canada/hist-cdns> {
    ?event crm:P7_took_place_at geonames:6094817/ .  # Ottawa

    {
      ?event crm:P98_brought_into_life ?person .
      BIND("Born" AS ?eventType)
    } UNION {
      ?event crm:P100_was_death_of ?person .
      BIND("Died" AS ?eventType)
    } UNION {
      ?event crm:P14_carried_out_by ?person ;
             crm:P2_has_type event:OccupationEvent .
      BIND("Worked" AS ?eventType)
    }

    ?person rdfs:label ?name .
    OPTIONAL {
      ?event crm:P4_has_time-span ?ts .
      ?ts crm:P82_at_some_time_within ?date .
    }
  }
}
ORDER BY ?date
```

### 5. Indian Affairs Agents by Agency
```sparql
SELECT ?agent ?agentName ?agency ?agencyName ?occLabel ?startDate ?place ?placeLabel
WHERE {
  GRAPH <http://graph.lincsproject.ca/hist-canada/ind-affairs> {
    ?agency a crm:E74_Group ;
            rdfs:label ?agencyName ;
            crm:P107_has_current_or_former_member ?agent .

    ?agent a crm:E21_Person ;
           rdfs:label ?agentName .

    OPTIONAL {
      ?occ a crm:E7_Activity ;
           crm:P14_carried_out_by ?agent ;
           crm:P2_has_type event:OccupationEvent ;
           rdfs:label ?occLabel .
      OPTIONAL {
        ?occ crm:P4_has_time-span ?ts .
        ?ts crm:P82_at_some_time_within ?startDate .
      }
      OPTIONAL {
        ?occ crm:P7_took_place_at ?place .
        ?place rdfs:label ?placeLabel .
      }
    }
  }
}
ORDER BY ?agencyName ?startDate
```

### 6. Cabinet Meeting Topics Over Time
```sparql
# NOTE: P82a values are xsd:dateTime (e.g., "1944-06-06T00:00:00").
# GROUP BY raw ?date would group by exact timestamp, not year.
# Use YEAR() to bin by calendar year for historical analysis.
SELECT ?year ?topic ?topicLabel (COUNT(?meeting) AS ?meetings)
WHERE {
  GRAPH <http://graph.lincsproject.ca/hist-canada/cabinet-conclusions> {
    ?meeting a crm:E7_Activity ;
             crm:P21_had_general_purpose ?topic ;
             crm:P4_has_time-span ?ts .
    ?topic rdfs:label ?topicLabel .
    ?ts crm:P82a_begin_of_the_begin ?date .
    BIND(YEAR(?date) AS ?year)
  }
}
GROUP BY ?year ?topic ?topicLabel
ORDER BY ?year
```
**Temporal aggregation helpers** — use these SPARQL functions when grouping historical dates:
| Granularity | Expression | Example Output |
|-------------|-----------|----------------|
| Year | `BIND(YEAR(?date) AS ?year)` | `1944` |
| Decade | `BIND(FLOOR(YEAR(?date)/10)*10 AS ?decade)` | `1940` |
| Month | `BIND(CONCAT(STR(YEAR(?date)), "-", STR(MONTH(?date))) AS ?ym)` | `"1944-6"` |
| Quarter | `BIND(CONCAT(STR(YEAR(?date)), "-Q", STR(CEIL(MONTH(?date)/3.0))) AS ?q)` | `"1944-Q2"` |

### 7. Cross-Dataset: People Appearing in Multiple Datasets
```sparql
SELECT ?person 
       (SAMPLE(?name) AS ?personName) 
       (COUNT(DISTINCT ?g) AS ?datasetCount)
       (GROUP_CONCAT(DISTINCT ?label; separator=", ") AS ?foundIn)
WHERE {
  # Define the graphs and their readable labels upfront
  VALUES (?g ?label) {
    (<http://graph.lincsproject.ca/hist-canada/hist-cdns> "Historical Canadians")
    (<http://graph.lincsproject.ca/hist-canada/ind-affairs> "Indian Affairs")
    (<http://graph.lincsproject.ca/hist-canada/cabinet-conclusions> "Cabinet Conclusions")
  }
  
  # Execute a single graph pattern match against the defined list
  GRAPH ?g {
    ?person a crm:E21_Person ; 
            rdfs:label ?name .
  }
}
GROUP BY ?person
HAVING (?datasetCount > 1)
ORDER BY DESC(?datasetCount) ?personName
```
### 7.1. Cross-Dataset: All People Across Datasets
```sparql
SELECT ?person ?name ?dataset
WHERE {
  {
    GRAPH <http://graph.lincsproject.ca/hist-canada/hist-cdns> {
      ?person a crm:E21_Person ; rdfs:label ?name .
      BIND("Historical Canadians" AS ?dataset)
    }
  } UNION {
    GRAPH <http://graph.lincsproject.ca/hist-canada/ind-affairs> {
      ?person a crm:E21_Person ; rdfs:label ?name .
      BIND("Indian Affairs" AS ?dataset)
    }
  } UNION {
    GRAPH <http://graph.lincsproject.ca/hist-canada/cabinet-conclusions> {
      ?person a crm:E21_Person ; rdfs:label ?name .
      BIND("Cabinet Conclusions" AS ?dataset)
    }
  }
}
ORDER BY ?person
```

### 8. Census Measurement Query (Local CIDOC-CRM Data)
For querying your Neo4j CIDOC-CRM census data if serialized to RDF:

```sparql
PREFIX temp: <http://temp.lincsproject.ca/hgis/>

SELECT ?place ?placeName ?year ?variable ?value ?unit
WHERE {
  ?measurement a crm:E16_Measurement ;
               crm:P39_measured ?presence ;
               crm:P2_has_type ?varType ;
               crm:P40_observed_dimension ?dim ;
               crm:P4_has_time-span ?ts .

  ?presence crm:P166_was_a_presence_of ?place .
  ?place rdfs:label ?placeName .

  ?varType rdfs:label ?variable .

  ?dim crm:P90_has_value ?value ;
       crm:P91_has_unit ?unitNode .
  ?unitNode rdfs:label ?unit .

  ?ts crm:P82_at_some_time_within ?year .
}
ORDER BY ?placeName ?year ?variable
```

### 9. Population Timeline for a Place
```sparql
PREFIX base: <http://example.org/chgis/>

SELECT ?year ?population
WHERE {
  ?measurement a crm:E16_Measurement ;
               crm:P39_measured ?presence ;
               crm:P2_has_type base:VAR_POP_XX_N ;
               crm:P40_observed_dimension ?dim ;
               crm:P4_has_time-span ?ts .

  ?presence crm:P166_was_a_presence_of ?place .
  ?place rdfs:label "Westmeath"@en .

  ?dim crm:P90_has_value ?population .
  ?ts crm:P82a_begin_of_the_begin ?year .
}
ORDER BY ?year
```

### 10. Bridging Query: LINCS People ↔ Local Census Places

**Architecture note**: LINCS data lives on Fuseki (`fuseki.lincsproject.ca`). Your census data is local. A plain query combining both will FAIL unless both datasets are in the same triplestore. There are two approaches:

#### 10a. Co-located (both datasets loaded in same triplestore)
Use when you've loaded your census RDF into the same Fuseki/GraphDB instance as LINCS data:

Load into a graph with a URI that starts with `http://graph.lincsproject.ca/hist-canada/`

```sparql
SELECT ?person ?name ?localPlace ?localPlaceName ?censusYear ?population
WHERE {
  # LINCS biographical data (remote graph)
  GRAPH <http://graph.lincsproject.ca/hist-canada/hist-cdns> {
    ?birth crm:P98_brought_into_life ?person ;
           crm:P7_took_place_at ?geoPlace .
    ?person rdfs:label ?name .
  }

  GRAPH <http://graph.lincsproject.ca/hist-canada/my-graph> {

  # Local census data (default graph or named local graph)
  ?localPlace owl:sameAs ?geoPlace ;
              rdfs:label ?localPlaceName .
  ?measurement crm:P39_measured ?presence ;
               crm:P2_has_type base:VAR_POP_XX_N ;
               crm:P40_observed_dimension ?dim ;
               crm:P4_has_time-span ?ts .
  ?presence crm:P166_was_a_presence_of ?localPlace .
  ?dim crm:P90_has_value ?population .
  ?ts crm:P82_at_some_time_within ?censusYear .
  }
}
ORDER BY ?name ?censusYear
```

#### 10b. Federated (distributed across endpoints)
Use SPARQL `SERVICE` keyword when data lives in separate triplestores. Run this from your LOCAL triplestore, reaching out to LINCS:

```sparql
# Execute from your local SPARQL endpoint
SELECT ?person ?name ?localPlace ?localPlaceName ?censusYear ?population
WHERE {
  # Remote: fetch biographical data from LINCS Fuseki
  SERVICE <https://fuseki.lincsproject.ca/lincs/sparql> {
    GRAPH <http://graph.lincsproject.ca/hist-canada/hist-cdns> {
      ?birth crm:P98_brought_into_life ?person ;
             crm:P7_took_place_at ?geoPlace .
      ?person rdfs:label ?name .
    }
  }

  # Local: match to census data via shared GeoNames URIs
  ?localPlace owl:sameAs ?geoPlace ;
              rdfs:label ?localPlaceName .
  ?measurement crm:P39_measured ?presence ;
               crm:P2_has_type base:VAR_POP_XX_N ;
               crm:P40_observed_dimension ?dim ;
               crm:P4_has_time-span ?ts .
  ?presence crm:P166_was_a_presence_of ?localPlace .
  ?dim crm:P90_has_value ?population .
  ?ts crm:P82_at_some_time_within ?censusYear .
}
ORDER BY ?name ?censusYear
LIMIT 100  # Always LIMIT federated queries during development
```

**Federated query caveats:**
- The remote endpoint must support SPARQL 1.1 Federation (`SERVICE`)
- Performance depends on network latency and remote endpoint load
- Always use `LIMIT` during development to avoid timeouts
- Put the most selective patterns in the `SERVICE` block to minimize data transfer
- If the remote endpoint times out, pre-fetch the LINCS results into a local named graph instead

## Query Building Guidelines

When building queries:
1. Always specify the GRAPH for LINCS data
2. Use OPTIONAL for properties that may not exist on all entities
3. Use UNION to combine different event types
4. Use BIND to add readable labels for event categories
5. For performance, put the most selective triple pattern first
6. Use LIMIT during exploration, remove for production
7. Cross-dataset queries use UNION across GRAPH blocks
8. The bridging pattern (template 10) requires owl:sameAs links between your data and GeoNames URIs
9. For federated queries, use `SERVICE` keyword and always add `LIMIT`

### String Matching Performance

**WARNING**: `FILTER(CONTAINS(LCASE(?name), "macdonald"))` triggers a full table scan — every name string in the graph is lowercased and checked. On large endpoints this causes timeouts.

**Best practices** (in order of preference):
1. **Exact match** (fastest): `?nameNode crm:P190_has_symbolic_content "John A. Macdonald"^^xsd:string`
2. **URI match** (fast): `FILTER(?person = viaf:29539039)` — if you know the authority URI
3. **Starts-with** (moderate): `FILTER(STRSTARTS(LCASE(?name), "macdonald"))` — cheaper than CONTAINS
4. **Contains** (slow): `FILTER(CONTAINS(LCASE(?name), "macdonald"))` — use only with tight `LIMIT`
5. **Regex** (slowest): `FILTER(REGEX(?name, "mac.*donald", "i"))` — avoid on remote endpoints

When using fuzzy matching, always pair with `LIMIT` and add other selective triple patterns first to narrow the search space.

### Temporal Aggregation

LINCS dates are `xsd:dateTime` (e.g., `"1944-06-06T00:00:00"`). Grouping by raw datetime gives per-timestamp bins, not useful periods. Always extract the temporal component you need:
- **By year**: `BIND(YEAR(?date) AS ?year)` before `GROUP BY ?year`
- **By decade**: `BIND(FLOOR(YEAR(?date)/10)*10 AS ?decade)`
- See Template 6 for the full set of temporal helpers.
