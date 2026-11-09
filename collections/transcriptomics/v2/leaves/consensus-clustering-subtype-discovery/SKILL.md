---
name: consensus-clustering-subtype-discovery
description: Use when a sample-by-feature landscape (e.g. DCNet-inferred 434-cell-type
  abundances for ~10,176 TCGA bulk RNA-seq samples) should be partitioned into robust
  molecular subtypes testable against clinical survival.
when_to_use_negative:
- Input is raw or unnormalized gene expression rather than a DCNet-inferred cell abundance
  landscape — consensus clustering here is defined on the inferred landscape, not
  the expression matrix
- The goal is deconvolution of cell fractions itself; use DCNet (or comparison tools
  such as CIBERSORTx, EPIC, MCP-counter, quanTIseq, TIMER, xCell) first — clustering
  is a downstream step
- Samples lack matched clinical survival data, making the IPS/NIPS survival-separation
  evaluation impossible
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: DCNet
  role: mxnet-implemented interpretable ANN that embeds 434 cell types and 9078 marker
    genes to infer the per-sample cell abundances that serve as clustering input
  repo: https://github.com/xindd/DCNet
- name: ConsensusClusterPlus
  role: R package performing the consensus clustering of the patient abundance landscape
    into the two robust subtypes
- name: mxnet
  role: Python deep-learning framework in which the DCNet model producing the abundance
    matrix was coded
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:205903dcbfcf1d5764289a2982bb341af67d5ada422863ccb1a6f8aaac680727
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - ConsensusClusterPlus
  - DCNet
  - Python
  - mxnet
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# consensus-clustering-subtype-discovery

## Summary

Robust patient stratification by running ConsensusClusterPlus (manhattan distance, PAM clustering, 1000 iterations) on DCNet-inferred cell-type abundance landscapes, yielding two TME subtypes (IPS and NIPS) from 10,176 TCGA tumor samples that differ in overall survival and cell-type composition.

## When to use

Use when a sample-by-feature landscape (e.g. DCNet-inferred 434-cell-type abundances for ~10,176 TCGA bulk RNA-seq samples) should be partitioned into robust molecular subtypes testable against clinical survival.

## When NOT to use

- Input is raw or unnormalized gene expression rather than a DCNet-inferred cell abundance landscape — consensus clustering here is defined on the inferred landscape, not the expression matrix
- The goal is deconvolution of cell fractions itself; use DCNet (or comparison tools such as CIBERSORTx, EPIC, MCP-counter, quanTIseq, TIMER, xCell) first — clustering is a downstream step
- Samples lack matched clinical survival data, making the IPS/NIPS survival-separation evaluation impossible

## Inputs

- Bulk RNA-seq expression matrix (rows = human gene symbols, columns = samples), log2(x+1)-transformed
- DCNet-inferred cell-type abundance landscape for all samples (10,176 TCGA tumor samples x 434 cell types)
- relation_matrix.pkl (zero-one genes-cells correspondence matrix) and index.pkl (input/output neuron gene mapping)
- TCGA clinical survival data from GDC (https://portal.gdc.cancer.gov/projects)

## Outputs

- Consensus matrix and subtype assignments (IPS vs NIPS) for each of the 10,176 samples
- Cell abundance landscape plot separating the two subtypes (Figure 4a analog)
- Kaplan-Meier overall-survival curves per subtype with log-rank test p-value (Figure 4b analog)
- Saved cell abundance matrix and IPS/NIPS label table

## How to apply

First infer the per-sample cell landscape with DCNet (mxnet ANN whose input/output neurons are wired to 434 cell types and marker genes via the genes-cells zero-one relation matrix and index.pkl), after applying the recommended log2(x+1) transformation to the expression matrix (columns = samples, rows = human gene symbols). Feed the resulting 10,176-sample abundance matrix to the R package ConsensusClusterPlus using manhattan distance, PAM clustering, and 1000 iterations to recover the two robust subtypes, IPS and NIPS. Merge the subtype assignments with TCGA clinical survival data, draw Kaplan-Meier overall-survival curves for the two groups, and compute the log-rank test; additionally compare cell-type composition between subtypes to confirm the separation is biologically meaningful. Save the cell abundance matrix, the IPS/NIPS labels, and the landscape plot (Figure 4a) and survival curves (Figure 4b) as outputs.

## Related tools

- **DCNet** (mxnet-implemented interpretable ANN that embeds 434 cell types and 9078 marker genes to infer the per-sample cell abundances that serve as clustering input) — https://github.com/xindd/DCNet
- **ConsensusClusterPlus** (R package performing the consensus clustering of the patient abundance landscape into the two robust subtypes)
- **mxnet** (Python deep-learning framework in which the DCNet model producing the abundance matrix was coded)

## Examples

```
Rscript -e 'library(ConsensusClusterPlus); res <- ConsensusClusterPlus(abundance_matrix, maxK=6, dist="manhattan", clusterAlg="pam", reps=1000)'
```

## Evaluation signals

- Consensus clustering with manhattan distance + PAM over 1000 iterations resolves a robust k=2 partition; subtype labels are stable across resampling runs
- Merged labels reproduce the published split sizes across the 10,176 TCGA tumor samples
- Log-rank test between IPS and NIPS Kaplan-Meier overall-survival curves yields p < .0001, with IPS patients surviving better
- Cell-type composition differs significantly between the two subtypes, not just survival
- Abundance matrix values are non-negative and the input expression matrix was log2(x+1)-transformed before DCNet inference

## Limitations

- No random seed or reproducibility controls for DCNet model training are reported; trained model weights other than the released net_M434_adam_T600_DCNetBnoise-D1.params are not stated as available
- No software version numbers are reported for mxnet, R packages, or comparison tools, and fine-tuning of the recommended dropout-0.1 DCNet on user data is not specified
- Consensus clustering parameters other than manhattan/PAM/1000 iterations are not described; the two-subtype result is specific to this pancancer TCGA cohort

## Evidence

- [methods] ConsensusClusterPlus parameters: manhattan distance, PAM, 1000 iterations: [evidence span withheld — anchor status: not_found]
