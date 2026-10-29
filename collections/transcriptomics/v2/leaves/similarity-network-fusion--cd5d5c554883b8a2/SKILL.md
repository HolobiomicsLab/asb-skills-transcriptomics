---
name: similarity-network-fusion
description: Use when multiple gene co-expression networks per condition (e.g., Euclidean-distance
  and Pearson-correlation networks) must be combined into a single fused network per
  cancer type using SNF.
when_to_use_negative:
- Input consists of a single co-expression network per cancer type — SNF requires
  at least two condition-specific networks (normal and cancer) to fuse
- Raw edge weights have not yet been thresholded/normalized (Euclidean edges must
  pass 99th percentile thresholding and Pearson edges the permutation-based p-value
  filtering before fusion)
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: Implements the SNF fusion step in fusion_networks.py as part of the pipeline
  repo: https://github.com/DEIB-GECO/GeneNetFusion
- name: Jupyter Notebook
  role: Pipeline.ipynb contains the complete pipeline including the fused co-expression
    network construction
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
- name: GMQL
  role: Retrieves TCGA gene expression data used to build the networks that are fused
  repo: http://gmql.eu
- name: PyGMQL
  role: Python interface for the GMQL extraction of TCGA gene expression datasets
  repo: https://pygmql.readthedocs.io/en/latest/
- name: Gephi
  role: Extracts gene communities from the fused network via its modularity method
    after SNF fusion
  repo: https://gephi.org
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:db27d61de99168ad713815cca15877a0b20791c4956ea11256d7d305a8d324fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/GeneNetFusion
  tools:
  - GMQL
  - Gephi
  - Jupyter Notebook
  - PyGMQL
  - Python
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# similarity-network-fusion

## Summary

Fuses condition-specific (normal vs cancer) gene co-expression networks for KIRC, LIHC and PRAD using the SNF algorithm with KNN-based kernel matrices, iteratively updating W matrices until the relative difference between iterations is <10^-5, yielding one cancer-type-specific fused adjacency matrix.

## When to use

Use when multiple gene co-expression networks per condition (e.g., Euclidean-distance and Pearson-correlation networks) must be combined into a single fused network per cancer type using SNF.

## When NOT to use

- Input consists of a single co-expression network per cancer type — SNF requires at least two condition-specific networks (normal and cancer) to fuse
- Raw edge weights have not yet been thresholded/normalized (Euclidean edges must pass 99th percentile thresholding and Pearson edges the permutation-based p-value filtering before fusion)

## Inputs

- Normalized normal-condition gene co-expression network (sum of scaled Euclidean-distance and Pearson adjacency matrices)
- Normalized cancer-condition gene co-expression network (sum of scaled Euclidean-distance and Pearson adjacency matrices)
- TCGA RNA-Seq gene expression data for KIRC, LIHC and PRAD pre-processed with GMQL/PyGMQL (protein-coding and long non-coding genes with mean zFPKM > -3.0)

## Outputs

- Fused-network adjacency matrix per cancer type as a weighted adjacency CSV (e.g., fused_network_KIRC.csv)
- Fused network in Gephi-importable format stored in the Graphs folder

## How to apply

First build two condition-specific sample networks per cancer type: normalize the logarithm of each Euclidean-distance adjacency matrix by its maximum logarithmic value and take absolute values of the Pearson adjacency matrices, then sum the two per condition to obtain one normal and one cancer network. Fuse the normal and cancer sample networks with the SNF algorithm (fusion_networks.py), computing K nearest neighbors (KNN)-based kernel matrices to measure local affinity in each network, and iteratively updating the W matrices until the relative difference between consecutive iterations falls below 10^-5. Output the final fused adjacency matrix as the average of the two converged matrices, saved e.g. as fused_network_KIRC.csv. Extract gene communities from the fused network afterwards using the modularity method implemented in Gephi, after optionally pruning with a disparity filter.

## Related tools

- **Python** (Implements the SNF fusion step in fusion_networks.py as part of the pipeline) — https://github.com/DEIB-GECO/GeneNetFusion
- **Jupyter Notebook** (Pipeline.ipynb contains the complete pipeline including the fused co-expression network construction) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
- **GMQL** (Retrieves TCGA gene expression data used to build the networks that are fused) — http://gmql.eu
- **PyGMQL** (Python interface for the GMQL extraction of TCGA gene expression datasets) — https://pygmql.readthedocs.io/en/latest/
- **Gephi** (Extracts gene communities from the fused network via its modularity method after SNF fusion) — https://gephi.org

## Examples

```
pip install -r requirements.txt && jupyter notebook Pipeline.ipynb  # run through fusion_networks.py stage to produce fused_network_<tumor>.csv
```

## Evaluation signals

- The fusion script (fusion_networks.py) runs successfully and produces fused-network adjacency matrix files (e.g., fused_network_KIRC.csv) that exist and are formatted as weighted adjacency matrices
- SNF iteration terminated because the relative difference between consecutive W matrices is <10^-5
- Fused matrix values lie in [0,1], being the average of the two converged KNN-kernel W matrices
- The fused network loads into Gephi and gene communities can be extracted with its modularity method
- Disparity filtering of the fused network retains on average about 4.9% of the edges in the Merged networks

## Limitations

- No software implementation or tool version is specified for the SNF algorithm in the discussion, so reproduction depends on the repository's fusion_networks.py
- The fused-network step depends on upstream thresholding choices (99th percentile for Euclidean edges, permutation test plus p-value percentile for Pearson edges) being reproduced exactly
- Community extraction is not fully automated: the README instructs users to run Gephi manually and save genes as 'IC_'+str(tumor)+'.csv'

## Evidence

- [abstract] Fusing normal and cancer gene co-expression networks for each cancer type: "fusing normal and cancer networks for each cancer type, we can have better insights on the overall structure of the cancer-type-specific network"
- [methods] KNN-based local affinity in SNF: "the use of the K nearest neighbors (KNN)33 to measure local affinity in each network"
- [results] Upstream thresholding of the networks to fuse: "edges in the Euclidean distance networks for the normal and cancer condition are those after their 99th percentile thresholding"
- [results] Disparity filter on fused networks: "The relevant edges selected with the disparity filter in the final (Fused) networks are on average about 4.9% of the edges in the Merged ones"
- [readme] Repository structure and fusion script: [evidence span withheld — anchor status: not_found]
