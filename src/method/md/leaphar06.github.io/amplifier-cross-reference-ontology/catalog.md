---
template:
  id: catalog
  name: Competitor Catalog
  rank: 1
expose:
  - kind: compose
    params:
      ontology:
        defaultValue: ${context.ontology}
---

# Competitor Catalog

Catalogs each competitor amplifier under its vendor. This is the raw
material for question 1: given a competitor part number, which TI parts
replace it. `belongsToVendor` runs child to parent, so vendor and
amplifier compose cleanly as a tree, no inverse path needed.

```tree-editor
shape: amplifier:CompetitorAmplifierShape
ontology: ${ontology}
```

```turtle
amplifier:CompetitorVendorShape
    a sh:NodeShape ;
    sh:targetClass amplifier:CompetitorVendor ;
    sh:property [
        sh:path amplifier:vendorName ;
        sh:name "Vendor Name" ;
        sh:maxCount 1 ;
        sh:order 1 ;
    ] ;
    .

amplifier:CompetitorAmplifierShape
    a sh:NodeShape ;
    sh:targetClass amplifier:CompetitorAmplifier ;
    sh:property [
        sh:path amplifier:belongsToVendor ;
        sh:name "Vendor" ;
        sh:class amplifier:CompetitorVendor ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        dash:composite true ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path amplifier:partNumber ;
        sh:name "Part Number" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    .
```
