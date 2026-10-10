---
template:
  id: https://leaphar06.github.io/amplifier-cross-reference-ontology/method/replacements
  name: "Replacements for a Competitor Part"
  rank: 5
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
    - id: competitor
      type: iri
      required: true
---
**Question.** For this competitor part, which TI parts are full (drop-in) replacements, and for partial matches, which parameters block a full match?

```text
PREFIX amplifier: <https://leaphar06.github.io/amplifier-cross-reference-ontology/vocabulary#>

# One row: a one-sentence summary for the chosen competitor part.
SELECT ?Finding
WHERE {
  {
    SELECT (COUNT(DISTINCT ?ti) AS ?evaluated)
    WHERE {
      ?ti amplifier:hasParameterMatch ?pm .
      ?pm amplifier:comparesTo <${competitor}> .
    }
  }
  OPTIONAL { <${competitor}> amplifier:partNumber ?pn }
  BIND(COALESCE(?pn, REPLACE(STR(<${competitor}>), "^.*#", "")) AS ?part)
  BIND(CONCAT(STR(?evaluated), " TI part(s) have been evaluated against ", ?part, ".") AS ?Finding)
}
```

```table
PREFIX amplifier: <https://leaphar06.github.io/amplifier-cross-reference-ontology/vocabulary#>

# One row = one TI part evaluated against the chosen competitor part.
SELECT ?TIPart ?Outcome ?Evaluated ?BlockedBy ?NotEvaluated
WHERE {
  # How many of the 7 criteria were recorded for this pair
  {
    SELECT ?ti (COUNT(DISTINCT ?pm) AS ?n)
    WHERE {
      ?ti amplifier:hasParameterMatch ?pm .
      ?pm amplifier:comparesTo <${competitor}> .
    }
    GROUP BY ?ti
  }
  # Criteria that failed (block a drop-in)
  OPTIONAL {
    SELECT ?ti (GROUP_CONCAT(DISTINCT ?label; SEPARATOR=", ") AS ?blocked)
    WHERE {
      VALUES (?ct ?label ?order) {
    (amplifier:GBWCriterion "GBW" 1)
    (amplifier:SupplyVoltageCriterion "Supply Voltage" 2)
    (amplifier:PackageCriterion "Package" 3)
    (amplifier:SlewRateCriterion "Slew Rate" 4)
    (amplifier:InputBiasCurrentCriterion "Input Bias Current" 5)
    (amplifier:NoiseDensityCriterion "Noise Density" 6)
    (amplifier:TempGradeCriterion "Temp Grade" 7)
      }
      ?ti amplifier:hasParameterMatch ?pm .
      ?pm amplifier:comparesTo <${competitor}> ;
          amplifier:matches false ;
          amplifier:evaluatesCriterion ?crit .
      ?crit a ?ct .
    }
    GROUP BY ?ti
  }
  # Criteria never compared for this pair
  OPTIONAL {
    SELECT ?ti (GROUP_CONCAT(DISTINCT ?label; SEPARATOR=", ") AS ?missing)
    WHERE {
      VALUES (?ct ?label ?order) {
    (amplifier:GBWCriterion "GBW" 1)
    (amplifier:SupplyVoltageCriterion "Supply Voltage" 2)
    (amplifier:PackageCriterion "Package" 3)
    (amplifier:SlewRateCriterion "Slew Rate" 4)
    (amplifier:InputBiasCurrentCriterion "Input Bias Current" 5)
    (amplifier:NoiseDensityCriterion "Noise Density" 6)
    (amplifier:TempGradeCriterion "Temp Grade" 7)
      }
      ?ti amplifier:hasParameterMatch ?any .
      ?any amplifier:comparesTo <${competitor}> .
      FILTER NOT EXISTS {
        ?ti amplifier:hasParameterMatch ?pm .
        ?pm amplifier:comparesTo <${competitor}> ;
            amplifier:evaluatesCriterion ?crit .
        ?crit a ?ct .
      }
    }
    GROUP BY ?ti
  }
  OPTIONAL { ?ti amplifier:partNumber ?pn }
  BIND(COALESCE(?pn, REPLACE(STR(?ti), "^.*#", "")) AS ?TIPart)
  BIND(IF(BOUND(?blocked), "Partial", IF(?n < 7, "Incomplete", "Full")) AS ?Outcome)
  BIND(CONCAT(STR(?n), " of 7") AS ?Evaluated)
  BIND(COALESCE(?blocked, "none") AS ?BlockedBy)
  BIND(COALESCE(?missing, "none") AS ?NotEvaluated)
}
ORDER BY ?Outcome ?TIPart
```

**Interpretation.** A pair is **Full** only when all 7 criteria are recorded and none fail. **Partial** means at least one criterion fails, and Blocked By names it. **Incomplete** means nothing has failed yet, but fewer than 7 criteria were compared, so the pair cannot be called a drop-in. Not Evaluated lists what is missing. This is a closed-world reading of the recorded matches: "none" in Blocked By means no failure is recorded, not that the part has been proven equivalent. The outcome is derived here at view time rather than asserted, so it updates whenever a ParameterMatch changes.