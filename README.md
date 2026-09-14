# amplifier-cross-reference-ontology
Ontology-based model for evaluating amplifier replacements for competitor parts. It classifies matches as full, partial, or no match and captures the engineering criteria behind those decisions.

## What this system does

This model answers three questions for High-Speed Amplifier products:

1. Given a competitor amplifier part number, which TI part(s) are full ("drop-in") replacements, and for partial matches, which specific parameter(s) block the full match?**
2. Across a given competitor vendor's catalog, where does TI's amplifier portfolio have systematic partial-only or no-match coverage, and which parameters are driving those gaps?**
3.How is an ambiguous "close enough" threshold on a parameter resolved, and whose judgment governs a full vs. partial call?**

It's the symbolic layer of a neurosymbolic design: this ontology fixes the matching criteria, thresholds, and classification logic deterministically. A separate generative layer (not part of this repo) would later handle natural-language mapping of oddly-worded competitor specs and explaining match/no-match results to sales engineers.

A separate generative layer, not part of this repo, could later handle mapping differently worded competitor specifications and explaining the results to sales engineers.
## Where things live
https://github.com/Leaphar06/amplifier-cross-reference-ontology/tree/main/src/method/oml/www.modelware.io/sierra 
