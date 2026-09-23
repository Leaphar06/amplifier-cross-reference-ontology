# amplifier-cross-reference-ontology

Ontology-based model for evaluating TI amplifier replacements for competitor parts. It classifies matches as full, partial, or no match and captures the engineering criteria behind those decisions.

## What this system does

This model answers three questions for TI's High-Speed Amplifier (HSAMPS) product line team (at my company):

1. Given a competitor amplifier part number, which TI part(s) are full ("drop-in") replacements, and for partial matches, which specific parameter(s) prevent a full match?
2. Across a competitor vendor's catalog, where does TI's amplifier portfolio have partial-only or no-match coverage, and which parameters are driving those gaps?
3. When a parameter is "close enough" but does not clearly meet the requirements, how is that decision made, and whose judgment determines whether the match is full or partial?

This is the symbolic layer of the project. The ontology defines the matching criteria, thresholds, and classification logic so the decisions are consistent and repeatable.

A separate generative layer, not part of this repo, could later handle mapping differently worded competitor specifications and explaining the match or no-match results to sales engineers.

## Where things live

```text
src/
├── method/oml/leaphar06.github.io/amplifier-cross-reference-ontology/
│   ├── vocabulary.oml          # Concepts, properties, relations, the defined concept
│   │                           # (FullMatchReplacement), and the PartialMatchDerivation rule
│   └── vocabulary-bundle.oml   # Vocabulary bundle - closes the vocabulary for reasoning
│                               # and enables disjointness between the match classifications
└── model/oml/leaphar06.github.io/amplifier-cross-reference-ontology/
    ├── description.oml         # Instance data: TI parts, competitor parts, vendors,
    │                           # product lines, and per-parameter match results
    └── description-bundle.oml  # Description bundle - packages the description for
                                # reasoning against the closed vocabulary bundle
```

The seven matching criteria are:

* Gain-bandwidth product (GBW)
* Supply voltage range
* Package/pinout
* Slew rate
* Input bias current
* Noise density
* Temperature grade

## How to build/verify it

From the repo root, run the OML tools to check the vocabulary and description files:

```bash
oml lint
oml reason
```

A successful run should report something with no errors.

## Status

The current model includes:

* Vocabulary and description bundles
* Property characteristics
* The `FullMatchReplacement` defined concept
* Disjointness between the three match classifications, verified through bundle closure
* The `PartialMatchDerivation` rule

Both `oml lint` and `oml reason` pass cleanly.
