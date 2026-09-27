# METHOD.md

## Why these four patterns
The description data naturally breaks into four areas: `Catalog` for competitor vendors and parts, `Product Lines` for TI’s amplifier families, `Parameter Matches` for the actual comparison results, and `Coverage Gaps` for unresolved failures. I split them this way because it keeps each type of information focused and easier to review on its own. It also follows the same general approach used in Fire Force, where things like `Concerns`, `Objectives`, and `Missions` are separated into their own sub-ontologies instead of being put into one large file.

## Description layout decision
I put each concern into its own OML description ontology: `catalog.oml`, `product-lines.oml`, `parameter-matches.oml`, and `coverage-gaps.oml`. These are brought together through `description-bundle.oml` instead of putting everything into one flat file.

There are two different ways I handle references between the files. When a new instance needs to point to something defined in another ontology, I use extends to make that reference available. For example, `parameter-matches.oml` extends `catalog.oml` so it can reference catalog:LT1360. When I need to add a property to an instance that is actually defined somewhere else, I use ref instance. For example, `parameter-matches.oml` can add `hasParameterMatch` to a HighSpeedAmplifier instance that is defined in `product-lines.oml`.

The .md compose files, such as `Catalog.md` and `ProductLines.md`, point to the bundle with ontology: and then select the specific sub-ontology with target:. This follows the same bundle and target approach used in the Fire Force examples like `Stakeholders.md` and `Dashboard.md`.

## Why sh:sparql for the business rules
I used two `sh:sparql` constraints on `ParameterMatchShape` because these rules go beyond what a basic `sh:property `cardinality check can handle.

The first rule says that if matches is false, the comparison needs to identify the `ThresholdPolicy` that was used to make that decision. In other words, a failed match should have an explanation behind it.

The second rule prevents the same TI amplifier from having two `ParameterMatch` rows for the same competitor part and the same comparison criterion. Each comparison should only be recorded once.

Both rules use `SELECT` queries to find the rows that violate the rule, with `$this` identifying the instance that should be flagged.

## Why Product Line is flat while Competitor Catalog is a tree
The two structures ended up being different because their relationships run in different directions.

For the competitor catalog, `belongsToVendor` goes from the competitor part to its vendor. That makes it straightforward to organize the data as a tree, with the vendor at the top and its amplifier parts underneath it.

`hasSubLine` works in the opposite direction. It goes from a parent `ProductLine` to its child line, similar to how `hasPort` works in Fire Force. Making that into a tree would require `dash:composite` together with `sh:inversePath`. I didn't find that combination used in either this project or the Fire Force reference, so I decided not to introduce an untested pattern this close to submission. For now, Product Lines stays as a flat table.

## What dogfooding verified
Running `oml lint` and `oml reason` against the split `.oml` files caught a couple of issues that weren't obvious at first.

One was that the ontology namespace has to match the file location. Since these files sit directly in the folder, the namespace couldn't include a `/description/` prefix. I also found that a `description` ontology can `uses` a vocabulary, but when it references another description ontology, it needs to extends it rather than `uses` it.

The live previews of the .md templates exposed a different problem. I initially treated the table-editor/tree-editor configuration and the SHACL Turtle as separate blocks. They actually belong to one fence, with the ---/YAML/--- configuration followed directly by the SHACL Turtle. There isn't another closing fence between them.

I also compared my templates against actual Fire Force templates such as `requirements.md`, `concerns.md`, and `stakeholders.md`. That helped confirm the correct front-matter structure, with `params` as a sibling of `expose` under template:. It also helped me remove several things I had assumed were part of the Sierra format, including `oml:localReference`, `sh:order`, and `dash:composite`, because I couldn't find them in the real templates.

## Open questions and next steps
One thing I haven't done yet is generate the coverage-gap rollup directly from `ParameterMatch`. Right now, that would be a stretch goal rather than something required for this submission.

I also changed `catalog.md` to use `table-editor` instead of the `tree-editor` I originally planned. I couldn't find a real Fire Force example showing enough of the internal `tree-editor` structure to confidently reproduce it, so I didn't want to guess. If I find a real example later, that's something I would revisit.

There also isn't a dashboard or diagram view yet for the amplifier cross-reference data. That would be the equivalent of something like Fire Force's `Dashboard.md`, but it's outside the current scope.