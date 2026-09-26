---
template:
  id: parameter-matches
  name: Parameter Matches
  rank: 3
expose:
  - kind: compose
    params:
      ontology:
        defaultValue: ${context.ontology}
---

# Parameter Matches

A `ParameterMatch` is a single comparison: one competitor part, evaluated
against one of the seven criteria, for one TI amplifier. This is where
"full match" vs "partial match" actually gets decided, `matches` records
the verdict, and `governedBy` records whose threshold policy made the call
when the verdict isn't a clean measured pass/fail.

Two rules enforce that a failed match can't go unexplained: a `matches
false` row must name the policy that governed it, and the same TI/competitor
pair can't be evaluated twice against the same criterion.

```turtle
amplifier:ParameterMatchShape
    a sh:NodeShape ;
    sh:targetClass amplifier:ParameterMatch ;
    sh:sparql [
        sh:message "A failed match (matches = false) must state which ThresholdPolicy governed the decision." ;
        sh:select """
            PREFIX amplifier: <https://leaphar06.github.io/amplifier-cross-reference-ontology/vocabulary#>
            SELECT $this WHERE {
                $this amplifier:matches false .
                FILTER NOT EXISTS { $this amplifier:governedBy ?policy }
            }
        """ ;
    ] ;
    sh:sparql [
        sh:message "This HighSpeedAmplifier already has a ParameterMatch evaluating this criterion against this competitor part, duplicate evaluation." ;
        sh:select """
            PREFIX amplifier: <https://leaphar06.github.io/amplifier-cross-reference-ontology/vocabulary#>
            SELECT $this WHERE {
                $this amplifier:comparesTo ?comp ;
                      amplifier:evaluatesCriterion ?crit .
                ?owner amplifier:hasParameterMatch $this ;
                       amplifier:hasParameterMatch ?other .
                ?other amplifier:comparesTo ?comp ;
                       amplifier:evaluatesCriterion ?crit .
                FILTER (?other != $this)
            }
        """ ;
    ] ;
    sh:property [
        sh:path [ sh:inversePath amplifier:hasParameterMatch ] ;
        sh:name "TI Amplifier" ;
        sh:class amplifier:HighSpeedAmplifier ;
        sh:maxCount 1 ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path amplifier:comparesTo ;
        sh:name "Competitor Part" ;
        sh:class amplifier:CompetitorAmplifier ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        oml:localReference true ;
        sh:order 2 ;
    ] ;
    sh:property [
        sh:path amplifier:evaluatesCriterion ;
        sh:name "Criterion" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        oml:localReference true ;
        sh:order 3 ;
    ] ;
    sh:property [
        sh:path amplifier:matches ;
        sh:name "Matches" ;
        sh:datatype xsd:boolean ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 4 ;
    ] ;
    sh:property [
        sh:path amplifier:governedBy ;
        sh:name "Policy" ;
        sh:class amplifier:ThresholdPolicy ;
        sh:maxCount 1 ;
        oml:localReference true ;
        sh:order 5 ;
    ] ;
    .
```

```table-editor
shape: amplifier:ParameterMatchShape
ontology: ${ontology}
```
