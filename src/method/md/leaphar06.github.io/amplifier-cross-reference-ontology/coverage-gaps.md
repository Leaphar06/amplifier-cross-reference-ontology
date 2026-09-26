---
template:
  id: coverage-gaps
  name: Coverage Gap
  rank: 4
expose:
  - kind: compose
    params:
      ontology:
        defaultValue: ${context.ontology}
---

# Coverage Gap

Records which failed parameter matches represent a coverage gap for a
given criterion. Kept as a simple hand-authored table for now; the
computed rollup off `ParameterMatch` described in Step 4 is a later
stretch, not required for the minimum.

```turtle
amplifier:CoverageGapShape
    a sh:NodeShape ;
    sh:targetClass amplifier:CoverageGap ;
    sh:property [
        sh:path amplifier:derivedFrom ;
        sh:name "Failed Match" ;
        sh:class amplifier:ParameterMatch ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        oml:localReference true ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path amplifier:gapCount ;
        sh:name "Gap Count" ;
        sh:datatype xsd:integer ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    .
```

```table-editor
shape: amplifier:CoverageGapShape
ontology: ${ontology}
```
