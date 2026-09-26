---
template:
  id: https://leaphar06.github.io/amplifier-cross-reference-ontology/method/coverage-gaps
  name: "Coverage Gap"
  rank: 4
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
    - id: target
      type: iri
      required: true
---
# Coverage Gap

Records which failed parameter matches represent a coverage gap for a given criterion. Kept as a simple hand-authored table for now; the computed rollup off `ParameterMatch` described in Step 4 is a later stretch, not required for the minimum.

```table-editor
---
target: ${target}
columns: { this: { label: "Coverage Gap" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix amplifier: <https://leaphar06.github.io/amplifier-cross-reference-ontology/vocabulary#> .

amplifier:CoverageGapShape
    a sh:NodeShape ;
    sh:targetClass amplifier:CoverageGap ;
    sh:property [
        sh:path amplifier:derivedFrom ;
        sh:name "Failed Match" ;
        sh:class amplifier:ParameterMatch ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
    ] ;
    sh:property [
        sh:path amplifier:gapCount ;
        sh:name "Gap Count" ;
        sh:datatype xsd:integer ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
    ] ;
    .
```