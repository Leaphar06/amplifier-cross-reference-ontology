---
template:
  id: product-lines
  name: TI Product Line
  rank: 2
expose:
  - kind: compose
    params:
      ontology:
        defaultValue: ${context.ontology}
---

# TI Product Line

Records which product line and sub-line each TI amplifier belongs to.
`hasSubLine` runs parent to child, the same direction as `hasPort` in
Fire Force, so this stays a flat table rather than risk an untested
`dash:composite` plus `sh:inversePath` combination.

```table-editor
shape: amplifier:HighSpeedAmplifierShape
ontology: ${ontology}
```

```turtle
amplifier:ProductLineShape
    a sh:NodeShape ;
    sh:targetClass amplifier:ProductLine ;
    sh:property [
        sh:path amplifier:productLineName ;
        sh:name "Product Line Name" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path amplifier:hasSubLine ;
        sh:name "Sub-Lines" ;
        sh:class amplifier:ProductLine ;
        oml:localReference true ;
        sh:order 2 ;
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
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path amplifier:belongsToLine ;
        sh:name "Product Line" ;
        sh:class amplifier:ProductLine ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        oml:localReference true ;
        sh:order 2 ;
    ] ;
    .
```
