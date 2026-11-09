---
name: graphml-network-topology-analysis
description: Use when the analysis requires supplying pathway topologies (KEGG-derived
  or custom networks from non-KEGG sources) in GRAPHML format to scBONITA's pathway-analysis
  stage, especially when scoring node importance from in-silico knock-in/knock-out
  perturbations of learned Boolean rules.
when_to_use_negative:
- When the question is gene-level differential expression or conventional over-representation
  analysis (ORA) rather than topology-aware pathway scoring — the article found conventional
  ORA inefficacious at identifying coherent molecular processes, but scBONITA PA answers
  a different question
- When the network source cannot be exported to GRAPHML format and is not among the
  KEGG pathways scBONITA downloads (e.g. hsa00010, hsa00020)
- When only a single-condition dataset exists with no contrast pairs, since pathway
  modulation scores require condition comparisons defined in the contrast file
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: scBONITA
  role: Pathway-analysis engine that ingests GRAPHML network topologies, simulates
    knock-in/knock-out perturbations, and computes node importance and pathway modulation
    scores
  repo: https://github.com/Thakar-Lab/scBONITA
- name: KEGG
  role: Source of pathway topologies (e.g. hsa00010, hsa00020) downloaded for rule
    inference and pathway analysis
- name: GRAPHML
  role: Network file format used to supply pathway topologies, especially networks
    from non-KEGG sources
- name: Python 3.6
  role: Runtime for pathwayAnalysis.py and the scBONITA pipeline (requires Python
    3.6 or newer)
  repo: https://github.com/Thakar-Lab/scBONITA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41540-022-00246-5
    title: Executable models of immune signaling pathways in HIV-associated atherosclerosis
  source_content_hash: sha256:c68859ce558c5c328f2d20254d215b5b24c94da4f0e3d23067ce73eba02041d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Thakar-Lab/scBONITA
  tools:
  - GRAPHML
  - KEGG
  - Python
  - Python 3.6
  - scBONITA
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# graphml-network-topology-analysis

## Summary

Load and analyze pathway network topologies stored as GRAPHML files within the scBONITA pipeline, so that Boolean rule inference, perturbation-based node importance scoring, and pathway modulation scores are computed on the correct topology (e.g. KEGG pathways like hsa00010 or non-KEGG networks saved in GRAPHML format).

## When to use

Use when the analysis requires supplying pathway topologies (KEGG-derived or custom networks from non-KEGG sources) in GRAPHML format to scBONITA's pathway-analysis stage, especially when scoring node importance from in-silico knock-in/knock-out perturbations of learned Boolean rules.

## When NOT to use

- When the question is gene-level differential expression or conventional over-representation analysis (ORA) rather than topology-aware pathway scoring — the article found conventional ORA inefficacious at identifying coherent molecular processes, but scBONITA PA answers a different question
- When the network source cannot be exported to GRAPHML format and is not among the KEGG pathways scBONITA downloads (e.g. hsa00010, hsa00020)
- When only a single-condition dataset exists with no contrast pairs, since pathway modulation scores require condition comparisons defined in the contrast file

## Inputs

- Pathway network in GRAPHML format (e.g. 'hsa00010'), either KEGG-downloaded or from non-KEGG sources
- Processed scRNA-seq training matrix (rows = genes/features, columns = cells, first column labeled 'Genes'), e.g. 'data/trainingData.csv'
- Binary cell-by-condition metadata file (rows = cells, columns = condition variables with 0/1 entries), e.g. 'data/conditions.txt'
- Contrast file where each line lists the pair of condition columns to compare, e.g. 'data/contrast.txt'

## Outputs

- Node importance scores per network node (scaled to [0,1], weighted by equivalent rule set size)
- Pathway modulation scores combining node importance with gene-expression fold-change per pathway-contrast combination
- Combined results written to 'pathway_analysis_results.csv' with an entry for every pathway-contrast pair analyzed

## How to apply

Obtain or export the pathway network as a GRAPHML file; for KEGG pathways, scBONITA downloads pathways such as hsa00010 (glycolysis) and hsa00020 directly, while custom networks from non-KEGG sources must be saved in GRAPHML format. Pass the GRAPHML paths to the pathway-analysis script via the --pathwayList argument, alongside the processed scRNA-seq training matrix (rows as genes, first column labeled 'Genes'), the binary cell-by-condition metadata file, and the contrast file listing condition pairs. scBONITA then uses the GRAPHML topology to restrict Boolean rule inference and to simulate in-silico knock-in/knock-out perturbations, weighting each node's importance score by the size of the equivalent rule set (ERS) to account for uncertainty in rule determination. Node importance scores are scaled to the [0,1] range and combined with fold-change in gene expression between contrasted conditions to yield pathway modulation scores for each pathway-condition pair.

## Related tools

- **scBONITA** (Pathway-analysis engine that ingests GRAPHML network topologies, simulates knock-in/knock-out perturbations, and computes node importance and pathway modulation scores) — https://github.com/Thakar-Lab/scBONITA
- **KEGG** (Source of pathway topologies (e.g. hsa00010, hsa00020) downloaded for rule inference and pathway analysis)
- **GRAPHML** (Network file format used to supply pathway topologies, especially networks from non-KEGG sources)
- **Python 3.6** (Runtime for pathwayAnalysis.py and the scBONITA pipeline (requires Python 3.6 or newer)) — https://github.com/Thakar-Lab/scBONITA

## Examples

```
python3.6 pathwayAnalysis.py --dataFile "data/trainingData.csv" --conditions "data/conditions.txt" --contrast "data/contrast.txt" --conditions_separator "comma" --contrast_separator "comma" --pathwayList "hsa00010"
```

## Evaluation signals

- The output file 'pathway_analysis_results.csv' exists and contains a pathway modulation score entry for each pathway-contrast combination analyzed
- Every node in the GRAPHML network has an importance score scaled within the [0,1] range
- The set of pathways analyzed matches the --pathwayList argument (e.g. 'hsa00010' appears in results when passed)
- Each contrast in the contrast file has a corresponding pathway modulation score, with scores weighted by fold-change between the compared conditions
- Rule inference on the loaded topology completed without topology/node-mismatch errors between the GRAPHML file and the training data gene names

## Limitations

- scBONITA's pathway analysis requires contrast pairs defined in the conditions metadata; pathways cannot be scored without a valid contrast file
- Rule inference and attractor analysis require substantial RAM (in excess of 10G for large scRNA-seq datasets); the README does not recommend running them on a desktop computer
- scBONITA inference is expected to improve when pure cell populations are sequenced; mixed PBMC subpopulations can reduce specificity of pathway analysis
- Large scRNA-seq datasets require the C file's NODE and CELL constants to be modified to reflect dataset size before compilation

## Evidence

- [readme] Paths to GRAPHML files that should be used for scBONITA analysis. Usually networks from non-KEGG sources, saved in GRAPHML format.: "Paths to GRAPHML files that should be used for scBONITA analysis. Usually networks from non-KEGG sources, saved in GRAPHML format."
- [methods] scBONITA performs node knock-out and knock-in simulations and calculates each node's importance score as the difference between network states after these simulations, weighted by the size of the equivalent rule set (ERS).: [evidence span withheld — anchor status: not_found]
- [methods] It downloads the KEGG pathways hsa00010 and hsa00020 for rule inference.: "It downloads the KEGG pathways hsa00010 and hsa00020 for rule inference."
- [abstract] scBONITA infers executable dynamic pathway models and performs a perturbation analysis to identify high impact genes.: "scBONITA infers executable dynamic pathway models and performs a perturbation analysis to identify high impact genes"
