---
name: cell-type-label-permutation-testing
description: Use when constructing a cell–cell communication network from scRNA-seq
  data and you need to decide which sender–receiver L–R interactions are statistically
  significant rather than keeping all expression-product edges.
when_to_use_negative:
- The downstream goal is temporal dynamics (DTW correlation across adjacent time points
  or fuzzy clustering of interaction clusters) rather than per-time-point network
  inference — those are handled by developcorrelation()/comparednet(), not the permutation
  test
- You lack discrete cell-type labels (e.g., only clusters without annotations), since
  the null distribution requires shuffling valid 'Type' labels
- Only two or very few cell types are present, making a 100-permutation label shuffle
  unable to produce a meaningful null distribution
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: dsCellNet
  role: R package providing singlenet() and preparepairs() that compute membrane/secreted
    L–R edge weights and run the 100-permutation cell-type-label significance test
  repo: github.com/songzh523/dsCellNet
- name: homologene
  role: Maps human L–R pair gene identifiers to mouse orthologs when building the
    multi-species L–R database used by the inference step
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:97f8083e9e83a37edf8d3fee858cbdf04af92075d8ca7532ccde64d679e894a1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/songzh523/dsCellNet
  tools:
  - R
  - dsCellNet
  - homologene
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cell-type-label-permutation-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assess significance of candidate ligand–receptor cell–cell interactions by permuting cell-type labels (100 permutations in dsCellNet) and retaining only interactions whose observed weight exceeds the permutation null (p < 0.05). This filters false-positive interactions when constructing the per-time-point communication network.

## When to use

Use when constructing a cell–cell communication network from scRNA-seq data and you need to decide which sender–receiver L–R interactions are statistically significant rather than keeping all expression-product edges.

## When NOT to use

- The downstream goal is temporal dynamics (DTW correlation across adjacent time points or fuzzy clustering of interaction clusters) rather than per-time-point network inference — those are handled by developcorrelation()/comparednet(), not the permutation test
- You lack discrete cell-type labels (e.g., only clusters without annotations), since the null distribution requires shuffling valid 'Type' labels
- Only two or very few cell types are present, making a 100-permutation label shuffle unable to produce a meaningful null distribution

## Inputs

- scRNA-seq gene expression matrix (e.g., 11,670 E15.5 mouse embryonic cerebral cortex cells) loaded via datainput(datloc, expnam, typnam, ...)
- Cell-type label table (inptyp with a 'Type' column)
- Time-point label table (inptim with a 'Time' column)
- Membrane/secreted L–R pair classification object from preparepairs(inpexp, species, dbloc, mainlab), mapped via homologene human/mouse homologs

## Outputs

- Edge list CSV (e.g., lr_interactions_E15_5.csv) with columns: sender, receiver, L–R pair, category, weight, p-value
- Set of significant cell–cell interactions (p < 0.05) for the single time point

## How to apply

After loading the expression matrix and cell-type label table with datainput() and splitting L–R pairs into membrane-ligand, secreted-ligand and membrane-receptor classes via preparepairs(), run singlenet() restricted to the time point of interest (e.g., E15.5) to enumerate candidate sender–receiver edges, where W = Mean(Ligand) x Mean(Receptor) for membrane-ligand pairs and W = Sum(Ligand) x Mean(Receptor) for secreted-ligand pairs. Then randomly shuffle the cell-type labels 100 times, recomputing the weights for each permutation, and assign each observed interaction a p-value equal to the permutation tail probability. Retain only significant interactions (p < 0.05) and write the filtered edge list (sender, receiver, L–R pair, category, weight, p-value) to the output CSV. Rationale: computing significance separately for membrane vs. secreted ligands accounts for their different expression profiles and produces a more robust interaction network than expression-based cutoffs alone.

## Related tools

- **dsCellNet** (R package providing singlenet() and preparepairs() that compute membrane/secreted L–R edge weights and run the 100-permutation cell-type-label significance test) — github.com/songzh523/dsCellNet
- **homologene** (Maps human L–R pair gene identifiers to mouse orthologs when building the multi-species L–R database used by the inference step)

## Examples

```
singlenet(inpexp,inptyp,"Type",inptim,"Time",memsercpros,usedcolo,"FALSE",dbloc,mainlab)
```

## Evaluation signals

- All p-values in the output edge list lie in [0, 1]
- Every retained row has p < 0.05 (significance filter applied exactly as specified)
- All weights are positive finite numbers, consistent with product-of-means formulations
- Output file contains all six required columns: sender, receiver, L–R pair, category, weight, p-value
- Membrane-ligand edges use Mean(Ligand) x Mean(Receptor) while secreted-ligand edges use Sum(Ligand) x Mean(Receptor), matching the category column

## Limitations

- The default L–R gene list size (5,744 interactions) limits detection of some interactions (e.g., MG–Neurons); users can supply supplementary L–R genes
- Published methods (and by extension this class of approaches) cannot fully distinguish membrane-ligand vs. secreted-ligand mediated receptor interactions given their different expression profiles
- Significance depends on 100 label permutations, so the smallest attainable p-value is bounded by the permutation count

## Evidence
