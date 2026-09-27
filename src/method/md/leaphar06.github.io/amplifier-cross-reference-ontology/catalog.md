---
template:
  id: https://leaphar06.github.io/amplifier-cross-reference-ontology/method/catalog
  name: "Competitor Catalog"
  rank: 1
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
# Competitor Catalog

Catalog competitor amplifiers under their vendor, so a competitor part number can be traced back to the manufacturer it belongs to.

```table-editor
---
target: ${target}
columns: { this: { label: "Competitor Amplifier" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix amplifier: <https://leaphar06.github.io/amplifier-cross-reference-ontology/vocabulary#> .

amplifier:CompetitorAmplifierShape
    a sh:NodeShape ;
    sh:targetClass amplifier:CompetitorAmplifier ;
    sh:property [
        sh:path amplifier:belongsToVendor ;
        sh:name "Vendor" ;
        sh:class amplifier:CompetitorVendor ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
    ] ;
    sh:property [
        sh:path amplifier:partNumber ;
        sh:name "Part Number" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
    ] ;
    .
```

# Competitor Vendors
List the vendors that compete against TI in this space.
```table-editor
---
target: ${target}
columns: { this: { label: "Vendor" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix amplifier: <https://leaphar06.github.io/amplifier-cross-reference-ontology/vocabulary#> .

amplifier:CompetitorVendorShape
    a sh:NodeShape ;
    sh:targetClass amplifier:CompetitorVendor ;
    sh:property [
        sh:path amplifier:vendorName ;
        sh:name "Vendor Name" ;
        sh:maxCount 1 ;
    ] ;
    .
```