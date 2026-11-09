---
name: permutation-based-interaction-significance-testing
description: Use when inferring cell-cell communication from clustered scRNA-seq data
  and you need to call which ligand-receptor pairs are statistically significant rather
  than merely enriched or co-expressed.
when_to_use_negative:
- Input data has not been clustered into cell groups — CellChat requires cell clustering
  as a pre-requisite, so unclustered or continuous-only state data cannot be permuted
  by group labels
- The goal is a published/curated ligand-receptor list without statistical testing
  — browsing CellChatDB directly suffices and no permutation null is needed
- Only single ligand/single receptor gene pairs with no consideration of multimeric
  complexes are required — CellChatDB's complex-aware modeling would be unnecessary
  overhead
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CellChat
  role: R package implementing mass-action communication probability estimation with
    the permutation significance test (computeCommunProb, filterCommunication, computeCommunProbPathway)
  repo: https://github.com/sqjin/CellChat
- name: CellChatDB
  role: Manually curated ligand-receptor database (2,021 validated mouse interactions)
    supplying the candidate interaction space that is permuted and tested
- name: SingleCellSignalR
  role: Alternative cell-cell communication inference tool referenced as prior art;
    different ligand-receptor database confounds cross-method comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21246-9
    title: Inference and analysis of cell-cell communication using CellChat
  source_content_hash: sha256:743cb70b64af0da16eaea9e5ce2041178cc6ff9334ac5a719708652cfff105bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sqjin/CellChat
  tools:
  - CellChat
  - CellChatDB
  - SingleCellSignalR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# permutation-based-interaction-significance-testing

## Summary

Statistical procedure from CellChat that identifies significant ligand-receptor interactions between cell groups by randomly permuting group labels (default nboot=100) and retaining interactions whose communication probability exceeds the null (p<0.05), before aggregating them into signaling pathways.

## When to use

Use when inferring cell-cell communication from clustered scRNA-seq data and you need to call which ligand-receptor pairs are statistically significant rather than merely enriched or co-expressed.

## When NOT to use

- Input data has not been clustered into cell groups — CellChat requires cell clustering as a pre-requisite, so unclustered or continuous-only state data cannot be permuted by group labels
- The goal is a published/curated ligand-receptor list without statistical testing — browsing CellChatDB directly suffices and no permutation null is needed
- Only single ligand/single receptor gene pairs with no consideration of multimeric complexes are required — CellChatDB's complex-aware modeling would be unnecessary overhead

## Inputs

- Preprocessed scRNA-seq expression matrix with cell-group annotations (e.g. 10X-derived mouse skin wound data, GSE113854, 21,557 cells after RBC exclusion)
- A CellChat object constructed with createCellChat using the mouse CellChatDB in label-based mode
- Over-expressed signaling genes from identifyOverExpressedGenes (Wilcoxon rank sum test, significance level 0.05)

## Outputs

- Table of significant ligand-receptor pairs with communication probabilities (60 pairs expected on the day-12 wound dataset)
- Pathway-level aggregated communication probabilities (25 signaling pathways expected)
- Null distributions from the 100 label permutations used to assign p-values

## How to apply

Model each ligand-receptor mediated signaling interaction using the law of mass action: compute communication probability with computeCommunProb via a Hill function over the ensemble average (trimean of quartiles) expression of signaling genes. Assess significance with a permutation test that randomly permutes cell group labels (default M=100 permutations, nboot=100) and re-computes the communication probability to build a null distribution; keep interactions with p<0.05. Filter the retained pairs with filterCommunication, then aggregate pair-level probabilities into pathway-level communication with computeCommunProbPathway. As a concrete acceptance check on the day-12 mouse skin wound dataset (GSE113854, 21,557 cells after red blood cell exclusion), the run is accepted only if the network yields 60 significant ligand-receptor pairs across 25 signaling pathways (TGFbeta, ncWNT, TNF, SPP1, PTN, PDGF, CXCL, CCL, MIF).

## Related tools

- **CellChat** (R package implementing mass-action communication probability estimation with the permutation significance test (computeCommunProb, filterCommunication, computeCommunProbPathway)) — https://github.com/sqjin/CellChat
- **CellChatDB** (Manually curated ligand-receptor database (2,021 validated mouse interactions) supplying the candidate interaction space that is permuted and tested)
- **SingleCellSignalR** (Alternative cell-cell communication inference tool referenced as prior art; different ligand-receptor database confounds cross-method comparison)

## Examples

```
cellchat <- createCellChat(data, meta = meta, group.by = "labels"); cellchat@DB <- CellChatDB; cellchat <- identifyOverExpressedGenes(cellchat); cellchat <- computeCommunProb(cellchat, type = "trimean", nboot = 100, population.size = TRUE)
```

## Evaluation signals

- The permutation test uses the default nboot=100 shuffles of cell group labels; only interactions with p<0.05 are retained
- On the day-12 mouse skin wound data, the network must reproduce exactly 60 significant ligand-receptor pairs
- The same run must aggregate to 25 signaling pathways including TGFbeta, ncWNT, TNF, and SPP1
- Downstream NMF pattern recognition (identifyCommunicationPatterns, thresholding W/H elements <0.5 to zero) must return 5 outgoing and 5 incoming communication patterns
- Interactions were pre-filtered by over-expressed signaling genes identified with the Wilcoxon rank sum test at significance level 0.05

## Limitations

- No ground truth exists for systematic evaluation of predicted cell-cell communication networks; overlap-based comparisons across methods are limited because similar methods tend to generate similar results regardless of accuracy
- The communication probability only represents interaction strength and is not exactly a probability; mRNA levels are used to approximate protein levels, so comprehensive ligand-receptor modeling remains challenging without single-cell proteomics
- Hill-function parameters Kh and n lack explicit biological connections and are treated only as a nonlinear approximation of ligand-receptor interactions
- Different ligand-receptor databases used by each method could contribute to the variety of predicted interactions, confounding cross-method performance comparison

## Evidence

- [discussion] permutation null justification: "The significant interactions between two cell groups are identified using a permutation test by randomly permuting the group labels of cells"
- [discussion] mass action model: "We modeled ligand-receptor mediated signaling interactions using the law of mass action."
- [discussion] DE pre-filter: "we first identified differentially expressed signaling genes across all cell groups within a given scRNA-seq dataset, using the Wilcoxon rank sum test"
- [intro] CellChatDB size: "CellChatDB contains 2,021 validated molecular interactions, including 60% of paracrine/autocrine signaling interactions"
- [discussion] communication probability caveat: "The communication probability here only represents the interaction strength and is not exactly a probability."
