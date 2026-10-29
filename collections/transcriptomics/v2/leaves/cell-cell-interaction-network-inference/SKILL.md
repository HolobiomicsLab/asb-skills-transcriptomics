---
name: cell-cell-interaction-network-inference
description: Use when you have a scRNA-seq expression matrix, cell-type labels, and
  time-point annotations and need to infer which ligand-receptor interactions connect
  which sender/receiver cell types at a given time point.
when_to_use_negative:
- Your goal is to track dynamic interaction changes across a developmental series
  or compare two conditions (e.g., AD vs control) — use developcorrelation or comparednet
  rather than the single-time-point singlenet path.
- Your interaction of interest is not in dsCellNet's default L-R gene list — the article
  notes the default list size limits detection of some interactions (e.g., MG–Neurons),
  and users must supply supplementary L-R genes.
- You need to distinguish autocrine interactions — the article notes SingleCellSignalR
  cannot distinguish autocrine interactions with default parameters, and other methods'
  behaviors differ; verify the chosen method supports your interaction topology.
edam_operation: http://edamontology.org/operation_3940
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: dsCellNet
  role: Core R package implementing localization-aware ligand-receptor network inference
    with permutation-based significance testing (singlenet, preparepairs, datainput
    functions)
  repo: https://github.com/songzh523/dsCellNet
- name: homologene
  role: Maps human/mouse gene identifiers when building the membrane/secreted ligand-receptor
    pair set
- name: igraph
  role: Dependency (>=1.2.6) for network construction and analysis within dsCellNet
- name: CellPhoneDB
  role: Source of 1,396 L-R pairs incorporated into dsCellNet's default pair list
- name: NATMI
  role: Source of 4,071 L-R pairs incorporated into dsCellNet's default pair list
- name: SingleCellSignalR
  role: Source of 3,251 L-R pairs incorporated into dsCellNet's default pair list
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:fab5ddf2196aab82e7f75c253050f218ebce4a3e0f60ec79c5d4a6e983f22575
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/songzh523/dsCellNet
  repo_url: https://github.com/songzh523/dsCellNet
  tools:
  - CellPhoneDB
  - NATMI
  - R
  - SingleCellSignalR
  - TimiGP
  - dsCellNet
  - homologene
  - igraph
  edam_operation: http://edamontology.org/operation_3940
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# cell-cell-interaction-network-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infer cell-cell communication networks from scRNA-seq by splitting ligand-receptor pairs into membrane-ligand and secreted-ligand classes, computing expression-based edge weights, and retaining only permutation-significant interactions (p < 0.05) for a single time point using the dsCellNet R package.

## When to use

Use when you have a scRNA-seq expression matrix, cell-type labels, and time-point annotations and need to infer which ligand-receptor interactions connect which sender/receiver cell types at a given time point.

## When NOT to use

- Your goal is to track dynamic interaction changes across a developmental series or compare two conditions (e.g., AD vs control) — use developcorrelation or comparednet rather than the single-time-point singlenet path.
- Your interaction of interest is not in dsCellNet's default L-R gene list — the article notes the default list size limits detection of some interactions (e.g., MG–Neurons), and users must supply supplementary L-R genes.
- You need to distinguish autocrine interactions — the article notes SingleCellSignalR cannot distinguish autocrine interactions with default parameters, and other methods' behaviors differ; verify the chosen method supports your interaction topology.

## Inputs

- scRNA-seq gene expression matrix (e.g., E15.5 mouse embryonic cerebral cortex, 11,670 single cells, GSE153164)
- Cell-type label table with a 'Type' column
- Time-point annotation table with a 'Time' column (e.g., E15.5)
- Ligand-receptor pair database path (dbloc) and species name for homologene mapping

## Outputs

- Significant cell-cell interaction edge list (e.g., lr_interactions_E15_5.csv) with sender, receiver, L-R pair, category, weight, and p-value columns
- Per-cell-type incoming/outgoing/total communication scores from importance_plot
- Cell-type color mapping (usedcolo) for downstream plots

## How to apply

Load the expression matrix, cell-type table, and time-point file with datainput(datloc, expnam, typnam, timnam, "FALSE", resloc), coercing outputs to data frames for inpexp, inptyp, and inptim. Split L-R pairs into membrane-ligand, secreted-ligand, and membrane-receptor classes with preparepairs(inpexp, species, dbloc, mainlab), mapping gene identifiers via the homologene human/mouse homolog database; dsCellNet's default L-R list draws on Ramilowski et al. 2015 (2,557 pairs), CellPhoneDB (1,396 pairs), NATMI (4,071 pairs), and SingleCellSignalR (3,251 pairs) for a total of ~5,744 interactions. Compute edge weights as W = Mean(Ligand) x Mean(Receptor) for membrane-ligand pairs and W = Sum(Ligand) x Mean(Receptor) for secreted-ligand pairs, distinguishing the two because they have different expression profiles. Run singlenet(inpexp, inptyp, "Type", inptim, "Time", memsercpros, usedcolo, "FALSE", dbloc, mainlab) restricted to the time point of interest (e.g., E15.5) to enumerate candidate sender-receiver interactions. Permute cell-type labels 100 times, recompute weights per permutation, assign each observed interaction a permutation-tail p-value, and filter to significant interactions with p < 0.05. Optionally assign a consistent color mapping with definecolo and call importance_plot(resloc, "row", "incoming"/"outgoing"/"total", "None") to rank communicating cell types.

## Related tools

- **dsCellNet** (Core R package implementing localization-aware ligand-receptor network inference with permutation-based significance testing (singlenet, preparepairs, datainput functions)) — https://github.com/songzh523/dsCellNet
- **homologene** (Maps human/mouse gene identifiers when building the membrane/secreted ligand-receptor pair set)
- **igraph** (Dependency (>=1.2.6) for network construction and analysis within dsCellNet)
- **CellPhoneDB** (Source of 1,396 L-R pairs incorporated into dsCellNet's default pair list)
- **NATMI** (Source of 4,071 L-R pairs incorporated into dsCellNet's default pair list)
- **SingleCellSignalR** (Source of 3,251 L-R pairs incorporated into dsCellNet's default pair list)

## Examples

```
out<-datainput(datloc,expnam,typnam,timnam,"FALSE",resloc); memsercpros<-preparepairs(inpexp,species,dbloc,mainlab); singlenet(inpexp,inptyp,"Type",inptim,"Time",memsercpros,usedcolo,"FALSE",dbloc,mainlab)
```

## Evaluation signals

- The output edge list (e.g., lr_interactions_E15_5.csv) contains all six required columns: sender, receiver, L-R pair, category, weight, and p-value.
- All retained rows have p < 0.05 and p-values lie in [0,1]; weights are positive finite numbers.
- Membrane-ligand weights equal Mean(Ligand) x Mean(Receptor) while secreted-ligand weights equal Sum(Ligand) x Mean(Receptor), matching the localization-specific formulas.
- Permutation count is 100 shuffles of cell-type labels, and the reported p-value equals the permutation tail probability.
- For the AD brain application, the inferred AD interaction list should contain IL8, IL18, NRP1, CX3CR1, and TLR1/2, with MG and ASCs ranked top by importance_plot scores.

## Limitations

- The size of the default L-R gene list limits detection of some interactions (e.g., MG–Neurons); users can support their L-R genes as supplementary.
- Alternative ligand-receptor genes mediating MG–Neuron interactions between AD patients are not identified, implying complex mechanisms among individual AD patients.
- The R package requires R >= 4.0.3 plus specific dependency versions (igraph >= 1.2.6, homologene >= 1.4.68, Hmisc >= 4.5.0, etc.), and is installed from a source tarball (dsCellNetv1_1.0.tar.gz).
- A single-map (stationary-condition) network cannot capture dynamic communication changes during development; temporal analysis requires the DTW/fuzzy-clustering extensions.

## Evidence

- [abstract] It identifies interactions according to protein localizations and reinforces them via dynamic time warping within each time point and between adjacent time points, respectively.: "It identifies interactions according to protein localizations and reinforces them via dynamic time warping within each time point and between adjacent time points, respectively."
- [intro] Then, we constructed a more robust interaction network, via respectively calculating the significance of membrane or secreted L-R genes.: "Then, we constructed a more robust interaction network, via respectively calculating the significance of membrane or secreted L-R genes."
- [methods] singlenet(inpexp,inptyp,"Type",inptim,"Time",memsercpros,usedcolo,"FALSE",dbloc,mainlab): "singlenet(inpexp,inptyp,"Type",inptim,"Time",memsercpros,usedcolo,"FALSE",dbloc,mainlab)"
- [results] In total, 5,744 interactions were kept for the default list of L-R pairs.: "In total, 5,744 interactions were kept for the default list of L-R pairs."
- [other] dsCellNet computes p values for interactions between two cell types by randomly shuffling cell type labels 100 times and retains only significant interactions (p < 0.05).: [evidence span withheld — anchor status: not_found]
- [readme] memsercpros<-preparepairs(inpexp,species,dbloc,mainlab): "memsercpros<-preparepairs(inpexp,species,dbloc,mainlab)"
