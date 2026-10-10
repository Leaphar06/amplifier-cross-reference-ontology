# ANALYSIS.md

Each methodology question from Deliverable 1 ends in either an answer or a diagnosed gap. Evidence lives in `Dashboard.md` (model folder), which computes every value live. The findings below describe what the evidence showed at the time of writing.

## Q1. Which TI parts are drop-in replacements for a competitor part, and which parameters block the rest?

**Status:** Answered by query.

**Evidence:** The method-owned template `replacements.md`, run for LT1360 in Dashboard §1 and for AD8001 in `ReplacementsAD8001.md`.

**Finding:** No TI part is a full replacement for any of the four competitor parts. All four pairs are partial:

| TI part | Competitor | Recorded | Blocked by |
|---|---|---|---|
| OPA847 | LT1360 | 3 of 7 | Package |
| OPA842 | AD8000 | 2 of 7 | Input Bias Current |
| OPA843 | LT1220 | 7 of 7 | Package, Noise Density |
| LMH6624 | AD8001 | 7 of 7 | Supply Voltage, Input Bias Current |

The outcome is derived by a closed-world query, not by the reasoner. The vocabulary's own classification cannot produce it (see "Reasoning finding" below). The query uses three outcomes rather than two, because a pair with no failures but fewer than 7 recorded criteria is incomplete, not a drop-in.

## Q2. Across a vendor's catalog, where is TI partial-only or no-match, and which parameters drive it?

**Status:** Answered by query and script, with one diagnosed pattern gap.

**Evidence:** Dashboard §2 (vendor/part × criterion matrix and failure chart) and §3 (Python systematic-blocker analysis).

**Finding:** Package blocks both Linear Technology parts, and Input Bias Current blocks both Analog Devices parts. These are the only criteria that block every part of a vendor. Supply Voltage and Noise Density each block one part. Across the catalog, Package and Input Bias Current account for 4 of the 6 failed matches.

**Diagnosed gap (pattern):** *"We cannot report no-match coverage because the methodology has no way to record that a competitor part was reviewed and no viable TI replacement exists. `NoMatchReplacement` is a kind of TI part, but 'no replacement' is a fact about the competitor part."* An orphan query can only say no candidate is recorded, which is not the same as no candidate exists.

## Q3. When a parameter is "close enough," how is that decided, and whose judgment governs?

**Status:** Partly answered, with two diagnosed gaps and one limitation outside the model.

**Evidence:** Dashboard §4 (policy coverage by result) and §5 (passing matches with no policy).

**Finding:** All 6 failed matches cite `policy1`, as the ParameterMatch shape requires. Only 2 of 13 passing matches cite any policy.

**Diagnosed gap (pattern):** *"We cannot audit passing 'close enough' calls because the pattern requires a ThresholdPolicy only on failed matches."* The ambiguous decision Q3 asks about is most often a pass.

**Diagnosed gap (vocabulary):** *"We cannot report whose judgment governed a call because a ThresholdPolicy records only a name, not an owner, a tolerance, or the values that were compared."* Extending ThresholdPolicy with an owning stakeholder and a tolerance, and recording each part's measured values, would let the pass/fail verdict be derived instead of typed in.

**Outside the model:** Which stakeholder's threshold *should* govern is a governance decision for the product line, not something a query can decide. Once it is decided, it can be recorded as a ThresholdPolicy owned by that stakeholder and enforced with a conformance query.

## Gap detection by pattern

| Pattern | Check | Result |
|---|---|---|
| Competitor Catalog | Competitor parts with no ParameterMatch | 0 of 4 (clean; catches the next unevaluated part) |
| TI Product Line | TI amplifiers never evaluated | None (clean; catches the next new TI part) |
| Parameter Matches | TI/competitor pairs with fewer than 7 criteria | OPA847/LT1360 (3 of 7), OPA842/AD8000 (2 of 7) |
| Parameter Matches | Passing matches with no policy | 11 of 13 passes |
| Coverage Gaps | Failed matches with no CoverageGap | 5 of 6 failures |

## Reasoning finding: the model reasons clean but cannot classify

`oml reason` passes, yet three checks the vocabulary appears to make never take effect:

1. **`min 7` on HighSpeedAmplifier is never violated.** OPA847 and OPA842 have fewer than 7 matches, and the reasoner assumes the rest exist (open world). The pair-count query in §5 catches it.
2. **`FullParameterMatch < ParameterMatch` is never inferred.** With `<`, matching on `matches true` is a necessary condition only, so no ParameterMatch is classified as a FullParameterMatch. As a result, the `PartialMatchDerivation` rule, which requires `FullParameterMatch(m1)`, cannot fire for any part.
3. **`FullMatchReplacement` uses `restricts all`.** Under the open world assumption, the reasoner can never prove a part has no other, unrecorded failing match, so it cannot infer FullMatchReplacement either.

This is why replacement classification lives in the analysis layer: the question "is every recorded criterion a pass?" is a closed-world question, and a query answers it directly.

## Next increments

- Derive CoverageGap from ParameterMatch by query and retire the hand-entered `gapCount`.
- Extend ThresholdPolicy with an owner and tolerance, and require a policy on every ParameterMatch, not only failures.
- Add a way to record "reviewed, no viable replacement" on the competitor part.
- Decide whether to change `FullParameterMatch` from `<` to `=` so the PartialMatchDerivation rule can fire, then confirm with `oml reason`.