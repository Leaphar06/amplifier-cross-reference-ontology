---
template:
  id: https://leaphar06.github.io/amplifier-cross-reference-ontology/method/product-lines
  name: "TI Product Line"
  rank: 2
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
# TI Product Line

Record which product line and sub-line each TI amplifier belongs to, so a part can be traced back to the family it's positioned in.

```table-editor
---
target: ${target}
columns: { this: { label: "TI Amplifier" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix amplifier: <https://leaphar06.github.io/amplifier-cross-reference-ontology/vocabulary#> .

amplifier:ProductLineShape
    a sh:NodeShape ;
    sh:targetClass amplifier:ProductLine ;
    sh:property [
        sh:path amplifier:productLineName ;
        sh:name "Product Line Name" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
    ] ;
    sh:property [
        sh:path amplifier:hasSubLine ;
        sh:name "Sub-Lines" ;
        sh:class amplifier:ProductLine ;
    ] ;
    .

amplifier:HighSpeedAmplifierShape
    a sh:NodeShape ;
    sh:targetClass amplifier:HighSpeedAmplifier ;
    sh:property [
        sh:path amplifier:partNumber ;
        sh:name "Part Number" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
    ] ;
    sh:property [
        sh:path amplifier:belongsToLine ;
        sh:name "Product Line" ;
        sh:class amplifier:ProductLine ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
    ] ;
    .
```