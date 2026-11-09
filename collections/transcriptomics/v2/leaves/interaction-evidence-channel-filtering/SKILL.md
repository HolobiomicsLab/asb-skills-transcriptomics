---
name: interaction-evidence-channel-filtering
description: Use when building or filtering a protein–protein interaction database
  for intercellular communication analysis and you must separate STRING interactions
  by evidence channel (experiments > 0, database > 0, predicted low-evidence sources)
  with credibility/action annotations.
when_to_use_negative:
- When you need a pre-curated ligand–receptor complex resource (CellPhoneDB-style)
  rather than user-defined filtering of genome-wide STRING evidence channels
- When the filtering question is about subcellular plausibility of a pair (COMPARTMENTS
  localization cut-off) rather than evidence provenance — that is a separate localization
  filter with a user-adjustable cut-off
- When predicting downstream signal transduction inside receiver cells (NicheNet-style),
  which InterCellDB's interaction evidence channels do not model
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: InterCellDB
  role: R package whose interaction sub-libraries (exp/know/pred) store the partitioned
    STRING pairs and expose them as user-defined gene-pair filters by type, direction
    and effect
  repo: https://github.com/ZJUDBlab/InterCellDB
- name: STRING
  role: Source of all protein–protein interactions and their evidence-channel labels
    ('experiments > 0', 'database > 0'), low-evidence prediction scores, action modes,
    and action effects
- name: R
  role: Runtime environment in which InterCellDB data calculation and visualization
    are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/advs.202200045
    title: 'InterCellDB: A User‐Defined Database for Inferring Intercellular Networks'
  source_content_hash: sha256:08919d4ec59688176f9373f6a14c0ef91717332a715163881a10da710b62152d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZJUDBlab/InterCellDB
  tools:
  - COMPARTMENTS
  - InterCellDB
  - R
  - STRING
  - UniProt
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# interaction-evidence-channel-filtering

## Summary

Partitions genome-wide STRING v11 protein–protein interactions into three evidence-channel sub-libraries — experiment validated (InterCellDB.exp), pathway curated (InterCellDB.know), and predicted (InterCellDB.pred) — and annotates every pair with a credibility score, action mode, and action effect to form the user-filterable interaction backbone of InterCellDB.

## When to use

Use when building or filtering a protein–protein interaction database for intercellular communication analysis and you must separate STRING interactions by evidence channel (experiments > 0, database > 0, predicted low-evidence sources) with credibility/action annotations.

## When NOT to use

- When you need a pre-curated ligand–receptor complex resource (CellPhoneDB-style) rather than user-defined filtering of genome-wide STRING evidence channels
- When the filtering question is about subcellular plausibility of a pair (COMPARTMENTS localization cut-off) rather than evidence provenance — that is a separate localization filter with a user-adjustable cut-off
- When predicting downstream signal transduction inside receiver cells (NicheNet-style), which InterCellDB's interaction evidence channels do not model

## Inputs

- STRING v11 protein–protein interaction records with per-evidence-source labels and combined scores
- Gene-coded protein database for human and mouse built from Ensembl and NCBI genomes database references (snapshot dated March 20, 2020; 18990 human genes, 20938 mouse genes)

## Outputs

- InterCellDB.exp sub-library (experiment validated pairs, STRING label 'experiments > 0')
- InterCellDB.know sub-library (pathway curated pairs, STRING label 'database > 0')
- InterCellDB.pred sub-library (predicted pairs, score > 0 in neighborhood/fusion/cooccurence/homology/coexpression/textmining or transferred variants)
- Per-pair annotations: credibility score (highest ≥ 900, high ≥ 700, medium ≥ 400, low < 400), action mode (activation, binding, catalysis, expression, inhibition, ptmod, reaction, other), action effect (positive, negative, unspecified, undirected)

## How to apply

Query STRING version 11 (https://version-11-0.string-db.org) for all protein–protein interactions and assign each pair to a channel: pairs labelled 'experiments > 0' go to the experiment validated sub-library, pairs labelled 'database > 0' go to the pathway curated sub-library, and pairs with score > 0 in at least one low-evidence source (neighborhood, fusion, cooccurence, homology, coexpression, textmining, or their transferred variants) go to the predicted sub-library. Attach to every pair a credibility score binned into four confidence levels (highest ≥ 900, high ≥ 700, medium ≥ 400, low < 400), an action mode (activation, binding, catalysis, expression, inhibition, ptmod, reaction, other), and an action effect (positive, negative, unspecified, undirected) inherited from STRING. Intersect the resulting sub-libraries with the gene-coded protein database (built from Ensembl and NCBI references, snapshot dated March 20, 2020) so only pairs among the 18990 human or 20938 mouse annotated genes remain queryable. Downstream, these channels act as user-defined filters: e.g., setting action mode to 'binding' ensures factors released from y-axis cells can bind receptors in x-axis cells, and combining 'activation' with 'positive' effect extracts directional stimulatory pairs.

## Related tools

- **InterCellDB** (R package whose interaction sub-libraries (exp/know/pred) store the partitioned STRING pairs and expose them as user-defined gene-pair filters by type, direction and effect) — https://github.com/ZJUDBlab/InterCellDB
- **STRING** (Source of all protein–protein interactions and their evidence-channel labels ('experiments > 0', 'database > 0'), low-evidence prediction scores, action modes, and action effects)
- **R** (Runtime environment in which InterCellDB data calculation and visualization are executed)

## Examples

```
devtools::install_github("ZJUDBlab/InterCellDB")
```

## Evaluation signals

- Every pair labelled 'experiments > 0' in STRING appears in InterCellDB.exp and nowhere else; 'database > 0' pairs appear only in InterCellDB.know
- Predicted sub-library contains exactly pairs with score > 0 in at least one of neighborhood, fusion, cooccurence, homology, coexpression, textmining or their transferred variants
- Credibility score bins are monotone and exhaustive: highest ≥ 900, high ≥ 700, medium ≥ 400, low < 400, with no pair left un-binned
- Every retained pair maps to proteins in the gene-coded database, which must total 18990 human genes and 20938 mouse genes
- Action mode and action effect values are drawn only from the closed vocabularies (activation, binding, catalysis, expression, inhibition, ptmod, reaction, other) and (positive, negative, unspecified, undirected)

## Limitations

- Interaction data are a static STRING version 11 snapshot (query at https://version-11-0.string-db.org, gene/protein references dated March 20, 2020), so newer interactions are absent
- The predicted channel aggregates low-evidence sources such as coexpression and textmining, which do not necessarily imply direct physical interaction
- Credibility-score cut-offs (≥ 900/700/400) are tied to the STRING combined-score scale and are not recalibrated by InterCellDB

## Evidence

- [methods] The first subset contained those protein interactions that were validated by experiments. These protein interactions were labelled with "experiments > 0" in the STRING database.: [evidence span withheld — anchor status: not_found]
- [methods] The second subset included those protein interactions that were extracted from pathways and labelled with "database > 0" in the STRING database.: [evidence span withheld — anchor status: not_found]
- [methods] We named these three subsets as experiment validated, pathway curated, and predicted databases.: "We named these three subsets as experiment validated, pathway curated, and predicted databases."
- [readme] Gene pair (inferred from protein-protein interactions) subset selection by type, direction and effect: "Gene pair (inferred from protein-protein interactions) subset selection by type, direction and effect"
- [results] The gene database contained 18 990 human genes and 20 938 mouse genes with a variety of annotations: "The gene database contained 18 990 human genes and 20 938 mouse genes with a variety of annotations"
