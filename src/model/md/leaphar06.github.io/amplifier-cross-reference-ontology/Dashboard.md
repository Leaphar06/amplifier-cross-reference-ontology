---
ontology: https://leaphar06.github.io/amplifier-cross-reference-ontology/description-bundle
---

# Cross-Reference Analysis Dashboard

This dashboard answers the three questions this model was built for (see README.md) and checks each of the four description patterns for missing relationships. Every value on the page is computed live from the model. The prose explains what each result means, not what today's numbers are, so it stays true as parts and matches are added.

## 1. Q1: Replacements for a competitor part

The view below is a method-owned template, run here for LT1360. ReplacementsAD8001.md runs the same template for a different part.

```compose
template: https://leaphar06.github.io/amplifier-cross-reference-ontology/method/replacements
competitor: "https://leaphar06.github.io/amplifier-cross-reference-ontology/catalog#LT1360"
```

## 2. Q2: Which parameters block each vendor's parts?

**Question.** Across each competitor vendor's catalog, which criteria block TI's candidates, and on which parts?

```matrix
---
rowColumnLabel: "Vendor / Part  vs.  Criterion"
stylesheet:
  - selector: cell [Number(value) > 0]
    style:
      background-color: mistyrose
---
PREFIX amplifier: <https://leaphar06.github.io/amplifier-cross-reference-ontology/vocabulary#>

# Rows: every competitor part. Columns: all 7 criteria.
# Cell: how many TI candidates are blocked by that criterion for that part.
SELECT ?row ?column (COALESCE(?n, 0) AS ?value)
WHERE {
  ?comp a amplifier:CompetitorAmplifier ;
        amplifier:belongsToVendor ?v .
  OPTIONAL { ?v amplifier:vendorName ?vn }
  OPTIONAL { ?comp amplifier:partNumber ?pn }
  BIND(CONCAT(COALESCE(?vn, REPLACE(STR(?v), "^.*#", "")), " / ",
              COALESCE(?pn, REPLACE(STR(?comp), "^.*#", ""))) AS ?row)
  VALUES (?ct ?column ?order) {
    (amplifier:GBWCriterion "GBW" 1)
    (amplifier:SupplyVoltageCriterion "Supply Voltage" 2)
    (amplifier:PackageCriterion "Package" 3)
    (amplifier:SlewRateCriterion "Slew Rate" 4)
    (amplifier:InputBiasCurrentCriterion "Input Bias Current" 5)
    (amplifier:NoiseDensityCriterion "Noise Density" 6)
    (amplifier:TempGradeCriterion "Temp Grade" 7)
  }
  OPTIONAL {
    SELECT ?comp ?ct (COUNT(DISTINCT ?pm) AS ?n)
    WHERE {
      ?pm amplifier:comparesTo ?comp ;
          amplifier:matches false ;
          amplifier:evaluatesCriterion ?crit .
      ?crit a ?ct .
    }
    GROUP BY ?comp ?ct
  }
}
ORDER BY ?row ?order
```

```chart
---
type: bar
data:
  labels: Criterion
  datasets:
    - label: Failed matches
      data: Blocks
options:
  plugins:
    title:
      display: true
      text: Failed matches by criterion
    legend:
      display: false
  scales:
    y:
      beginAtZero: true
---
PREFIX amplifier: <https://leaphar06.github.io/amplifier-cross-reference-ontology/vocabulary#>

# One row = one criterion. The full list comes first, so a criterion
# that never blocks shows an explicit 0 instead of disappearing.
SELECT ?Criterion (COALESCE(?n, 0) AS ?Blocks)
WHERE {
  VALUES (?ct ?Criterion ?order) {
    (amplifier:GBWCriterion "GBW" 1)
    (amplifier:SupplyVoltageCriterion "Supply Voltage" 2)
    (amplifier:PackageCriterion "Package" 3)
    (amplifier:SlewRateCriterion "Slew Rate" 4)
    (amplifier:InputBiasCurrentCriterion "Input Bias Current" 5)
    (amplifier:NoiseDensityCriterion "Noise Density" 6)
    (amplifier:TempGradeCriterion "Temp Grade" 7)
  }
  OPTIONAL {
    SELECT ?ct (COUNT(DISTINCT ?pm) AS ?n)
    WHERE {
      ?pm amplifier:matches false ;
          amplifier:evaluatesCriterion ?crit .
      ?crit a ?ct .
    }
    GROUP BY ?ct
  }
}
ORDER BY ?order
```

**Interpretation.** Each cell counts the TI candidates blocked by that criterion for that competitor part. The query lists every part and every criterion before counting, so a 0 is an explicit cell rather than a missing one. A 0 means no failure is *recorded*. It can also mean the criterion was never compared, which section 5 checks separately. The chart totals the same failures across the whole catalog, so the criteria worth roadmap attention stand out first.

## 3. Q2: Systematic blockers (scripted analysis)

**Question.** Is any criterion blocking *every* part from a vendor, which would point to a portfolio gap rather than a one-off mismatch?

SPARQL reduces the model to one row per failed match. Python groups the rows by vendor, computes each criterion's share of that vendor's parts, and flags a criterion as systematic when it blocks every part of a vendor with at least two parts.

```python
# 1. fetch: one row per failed match per competitor part
result = await query("""
PREFIX amplifier: <https://leaphar06.github.io/amplifier-cross-reference-ontology/vocabulary#>

# One row per failed match per competitor part, plus one row for any part with no failures.
SELECT ?vendor ?part ?label
WHERE {
  ?comp a amplifier:CompetitorAmplifier ;
        amplifier:belongsToVendor ?v .
  OPTIONAL { ?v amplifier:vendorName ?vn }
  OPTIONAL { ?comp amplifier:partNumber ?pn }
  BIND(COALESCE(?vn, REPLACE(STR(?v), "^.*#", "")) AS ?vendor)
  BIND(COALESCE(?pn, REPLACE(STR(?comp), "^.*#", "")) AS ?part)
  OPTIONAL {
    VALUES (?ct ?label ?order) {
    (amplifier:GBWCriterion "GBW" 1)
    (amplifier:SupplyVoltageCriterion "Supply Voltage" 2)
    (amplifier:PackageCriterion "Package" 3)
    (amplifier:SlewRateCriterion "Slew Rate" 4)
    (amplifier:InputBiasCurrentCriterion "Input Bias Current" 5)
    (amplifier:NoiseDensityCriterion "Noise Density" 6)
    (amplifier:TempGradeCriterion "Temp Grade" 7)
    }
    ?pm amplifier:comparesTo ?comp ;
        amplifier:matches false ;
        amplifier:evaluatesCriterion ?crit .
    ?crit a ?ct .
  }
}
""")
rows = result["rows"]

# 2. compute: for each vendor, which criteria block which of its parts
vendors = {}    # vendor -> set of all its competitor parts
blocked = {}    # (vendor, criterion) -> set of parts that criterion blocks
for r in rows:
    vendor, part, label = r.get("vendor"), r.get("part"), r.get("label")
    vendors.setdefault(vendor, set()).add(part)
    if label:
        blocked.setdefault((vendor, label), set()).add(part)

findings = []
for (vendor, label), parts in blocked.items():
    total = len(vendors[vendor])
    share = len(parts) / total if total else 0.0       # guarded division
    systematic = total >= 2 and len(parts) == total     # blocks every part of this vendor
    findings.append((vendor, label, len(parts), total, share, systematic))
findings.sort(key=lambda f: (f[0], -f[4], f[1]))

# 3. render
body = "".join(
    f"<tr><td>{v}</td><td>{c}</td><td>{n} of {t}</td><td>{s:.0%}</td>"
    f"<td>{'<b>Yes</b>' if sysm else 'No'}</td></tr>"
    for v, c, n, t, s, sysm in findings
)
systematic = [f"{c} ({v})" for v, c, n, t, s, sysm in findings if sysm]
summary = ("<p><b>Systematic blockers:</b> " + ", ".join(systematic) + ".</p>") if systematic \
          else "<p><b>No criterion blocks every part of any vendor.</b></p>"
display(
    summary +
    '<table class="oml-md-table"><thead><tr><th>Vendor</th><th>Blocking criterion</th>'
    '<th>Parts blocked</th><th>Share</th><th>Systematic</th></tr></thead>'
    f"<tbody>{body}</tbody></table>"
)
```

**Interpretation.** A criterion that blocks every part from one vendor suggests that vendor competes on a parameter TI's portfolio does not reach, which is a roadmap input rather than a datasheet nit. With only two parts per vendor today, "systematic" is a weak signal. The rule is written so the same analysis becomes meaningful as the catalog grows toward the ~50 parts in the original workbook. The shares are computed at render time and never written back into the model.

## 4. Q3: Is every close-enough call tied to a policy?

**Question.** For each pass or fail, does the model record which ThresholdPolicy governed the decision?

```table
PREFIX amplifier: <https://leaphar06.github.io/amplifier-cross-reference-ontology/vocabulary#>

# One row = one result type (Pass or Fail).
SELECT ?Result ?Matches ?WithPolicy (?Matches - ?WithPolicy AS ?WithoutPolicy)
WHERE {
  {
    SELECT ?Result (COUNT(DISTINCT ?pm) AS ?Matches) (COUNT(DISTINCT ?governed) AS ?WithPolicy)
    WHERE {
      ?pm a amplifier:ParameterMatch ;
          amplifier:matches ?ok .
      BIND(IF(?ok, "Pass", "Fail") AS ?Result)
      OPTIONAL {
        ?pm amplifier:governedBy ?policy .
        BIND(?pm AS ?governed)
      }
    }
    GROUP BY ?Result
  }
}
ORDER BY ?Result
```

**Interpretation.** The ParameterMatch shape requires a policy only on failed matches, so the Fail row should always show 0 without a policy. The Pass row is the real question for Q3: a "close enough, so it passes" call is the ambiguous case, and the model only records a policy on a pass when someone chooses to. A ThresholdPolicy also records only a name, not who owns it or what tolerance it sets, so the model can say *which* policy was cited but not *whose* judgment it represents. See ANALYSIS.md for the diagnosed gap.

## 5. Gap detection: one check per pattern

Each check is a closed-world query over this page's scope. A row means the relationship is not *recorded*, not that it cannot exist.

### Competitor Catalog: parts never evaluated

```text
PREFIX amplifier: <https://leaphar06.github.io/amplifier-cross-reference-ontology/vocabulary#>

SELECT ?Finding
WHERE {
  { SELECT (COUNT(DISTINCT ?c) AS ?total) WHERE { ?c a amplifier:CompetitorAmplifier } }
  { SELECT (COUNT(DISTINCT ?c) AS ?unevaluated)
    WHERE { ?c a amplifier:CompetitorAmplifier .
            FILTER NOT EXISTS { ?pm amplifier:comparesTo ?c } } }
  BIND(CONCAT(STR(?unevaluated), " of ", STR(?total),
       " competitor parts have no ParameterMatch recorded.") AS ?Finding)
}
```

```table
PREFIX amplifier: <https://leaphar06.github.io/amplifier-cross-reference-ontology/vocabulary#>

# One row = one competitor part with no ParameterMatch recorded against it.
SELECT ?CompetitorPart ?Vendor
WHERE {
  ?comp a amplifier:CompetitorAmplifier .
  OPTIONAL { ?comp amplifier:partNumber ?pn }
  OPTIONAL { ?comp amplifier:belongsToVendor ?v . OPTIONAL { ?v amplifier:vendorName ?vn } }
  FILTER NOT EXISTS { ?pm amplifier:comparesTo ?comp }
  BIND(COALESCE(?pn, REPLACE(STR(?comp), "^.*#", "")) AS ?CompetitorPart)
  BIND(COALESCE(?vn, "unknown") AS ?Vendor)
}
```

An empty table means every catalog part has at least one comparison. The check stays in place to catch the next competitor part added to the catalog without any evaluation, which Q1 would otherwise report as "0 TI parts evaluated."

### TI Product Line: amplifiers never evaluated

```table
PREFIX amplifier: <https://leaphar06.github.io/amplifier-cross-reference-ontology/vocabulary#>

# One row = one TI amplifier that has never been evaluated against any competitor part.
SELECT ?TIPart ?ProductLine
WHERE {
  ?ti a amplifier:HighSpeedAmplifier .
  OPTIONAL { ?ti amplifier:partNumber ?pn }
  OPTIONAL { ?ti amplifier:belongsToLine ?pl . OPTIONAL { ?pl amplifier:productLineName ?pln } }
  FILTER NOT EXISTS { ?ti amplifier:hasParameterMatch ?pm }
  BIND(COALESCE(?pn, REPLACE(STR(?ti), "^.*#", "")) AS ?TIPart)
  BIND(COALESCE(?pln, "none") AS ?ProductLine)
}
```

An empty table means every TI amplifier has been compared against at least one competitor part. A new TI part added through the Product Lines editor would appear here until someone evaluates it.

### Parameter Matches: pairs with fewer than 7 criteria

```table
PREFIX amplifier: <https://leaphar06.github.io/amplifier-cross-reference-ontology/vocabulary#>

# One row = one TI/competitor pair with fewer than 7 criteria recorded.
SELECT ?TIPart ?CompetitorPart ?Recorded
WHERE {
  {
    SELECT ?ti ?comp (COUNT(DISTINCT ?pm) AS ?n)
    WHERE {
      ?ti amplifier:hasParameterMatch ?pm .
      ?pm amplifier:comparesTo ?comp .
    }
    GROUP BY ?ti ?comp
  }
  FILTER (?n < 7)
  OPTIONAL { ?ti amplifier:partNumber ?tpn }
  OPTIONAL { ?comp amplifier:partNumber ?cpn }
  BIND(COALESCE(?tpn, REPLACE(STR(?ti), "^.*#", "")) AS ?TIPart)
  BIND(COALESCE(?cpn, REPLACE(STR(?comp), "^.*#", "")) AS ?CompetitorPart)
  BIND(CONCAT(STR(?n), " of 7") AS ?Recorded)
}
ORDER BY ?TIPart
```

The vocabulary declares `HighSpeedAmplifier restricts hasParameterMatch to min 7`, and `oml reason` still passes with these rows present. Under the open world assumption, the reasoner assumes the missing comparisons exist somewhere. This query counts what is actually recorded. (The restriction is also per TI part, not per TI/competitor pair, so a part compared against two competitors could satisfy it while each pair is incomplete.)

### Parameter Matches: passing matches with no policy

```table
PREFIX amplifier: <https://leaphar06.github.io/amplifier-cross-reference-ontology/vocabulary#>

# One row = one passing match with no ThresholdPolicy recorded.
SELECT ?Match ?TIPart ?CompetitorPart ?Criterion
WHERE {
  VALUES (?ct ?Criterion ?order) {
    (amplifier:GBWCriterion "GBW" 1)
    (amplifier:SupplyVoltageCriterion "Supply Voltage" 2)
    (amplifier:PackageCriterion "Package" 3)
    (amplifier:SlewRateCriterion "Slew Rate" 4)
    (amplifier:InputBiasCurrentCriterion "Input Bias Current" 5)
    (amplifier:NoiseDensityCriterion "Noise Density" 6)
    (amplifier:TempGradeCriterion "Temp Grade" 7)
  }
  ?pm amplifier:matches true ;
      amplifier:comparesTo ?comp ;
      amplifier:evaluatesCriterion ?crit .
  ?crit a ?ct .
  ?ti amplifier:hasParameterMatch ?pm .
  FILTER NOT EXISTS { ?pm amplifier:governedBy ?policy }
  OPTIONAL { ?ti amplifier:partNumber ?tpn }
  OPTIONAL { ?comp amplifier:partNumber ?cpn }
  BIND(REPLACE(STR(?pm), "^.*#", "") AS ?Match)
  BIND(COALESCE(?tpn, REPLACE(STR(?ti), "^.*#", "")) AS ?TIPart)
  BIND(COALESCE(?cpn, REPLACE(STR(?comp), "^.*#", "")) AS ?CompetitorPart)
}
ORDER BY ?TIPart ?order
```

These are the calls Q3 cannot audit. Each row is a pass that may have been a "close enough" judgment, with nothing recording which threshold allowed it.

### Coverage Gaps: failed matches with no CoverageGap

```text
PREFIX amplifier: <https://leaphar06.github.io/amplifier-cross-reference-ontology/vocabulary#>

SELECT ?Finding
WHERE {
  { SELECT (COUNT(DISTINCT ?pm) AS ?failed)
    WHERE { ?pm a amplifier:ParameterMatch ; amplifier:matches false } }
  { SELECT (COUNT(DISTINCT ?pm) AS ?unrecorded)
    WHERE { ?pm a amplifier:ParameterMatch ; amplifier:matches false .
            FILTER NOT EXISTS { ?gap amplifier:derivedFrom ?pm } } }
  BIND(CONCAT(STR(?unrecorded), " of ", STR(?failed),
       " failed matches have no CoverageGap recorded.") AS ?Finding)
}
```

```table
PREFIX amplifier: <https://leaphar06.github.io/amplifier-cross-reference-ontology/vocabulary#>

# One row = one failed match that no CoverageGap is derived from.
SELECT ?Match ?TIPart ?CompetitorPart ?Criterion
WHERE {
  VALUES (?ct ?Criterion ?order) {
    (amplifier:GBWCriterion "GBW" 1)
    (amplifier:SupplyVoltageCriterion "Supply Voltage" 2)
    (amplifier:PackageCriterion "Package" 3)
    (amplifier:SlewRateCriterion "Slew Rate" 4)
    (amplifier:InputBiasCurrentCriterion "Input Bias Current" 5)
    (amplifier:NoiseDensityCriterion "Noise Density" 6)
    (amplifier:TempGradeCriterion "Temp Grade" 7)
  }
  ?pm amplifier:matches false ;
      amplifier:comparesTo ?comp ;
      amplifier:evaluatesCriterion ?crit .
  ?crit a ?ct .
  OPTIONAL { ?ti amplifier:hasParameterMatch ?pm . OPTIONAL { ?ti amplifier:partNumber ?tpn } }
  OPTIONAL { ?comp amplifier:partNumber ?cpn }
  FILTER NOT EXISTS { ?gap amplifier:derivedFrom ?pm }
  BIND(REPLACE(STR(?pm), "^.*#", "") AS ?Match)
  BIND(COALESCE(?tpn, "unassigned") AS ?TIPart)
  BIND(COALESCE(?cpn, REPLACE(STR(?comp), "^.*#", "")) AS ?CompetitorPart)
}
ORDER BY ?CompetitorPart ?order
```

CoverageGap instances are entered by hand, and `gapCount` is typed rather than counted. Every row here is a failure the Coverage Gaps pattern does not yet reflect. Sections 2 and 3 derive the same information directly from ParameterMatch, which suggests the coverage rollup should be computed by query rather than maintained by hand.