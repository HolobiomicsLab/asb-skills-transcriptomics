---
name: gene-community-modularity-detection
description: Use when you have a fused gene co-expression network (e.g., Gephi-format
  .gexf from the GeneNetFusion pipeline) and need to partition genes into communities
  for biomarker extraction.
when_to_use_negative:
- Input is raw TCGA gene expression data and no fused network has been built yet —
  run Pipeline.ipynb (preprocessing, creation_matrices, fusion_networks) first
- The goal is a differential-expression baseline signature — the article uses DESeq2-identified
  DE genes as a comparison, not as the community source
- You already have extracted IC gene sets and want classification performance — that
  is the downstream LASSO + Random Forest step, not community detection
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0605
tools:
- name: Gephi
  role: Runs the modularity community-detection method on the fused, disparity-filter-pruned
    co-expression networks to extract IC gene communities
  repo: https://gephi.org
- name: Jupyter Notebook
  role: Pipeline.ipynb builds and fuses the co-expression networks up to the point
    where the graph is handed off to Gephi for community extraction
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
- name: Python
  role: Preprocessing.py, creation_matrices.py, fusion_networks.py and extraction_classification.py
    implement the network construction steps feeding and following the Gephi modularity
    step
  repo: https://github.com/DEIB-GECO/GeneNetFusion
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:2fe69df51db162f4c1a6601c4f50e70b20494eb73bd03754895104365bd9f6e4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Gephi
  - Jupyter Notebook
  - Python
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gene-community-modularity-detection

## Summary

Extracts communities of co-expressed candidate biomarker genes (IC genes) by applying the modularity community-detection method in Gephi to fused, disparity-filter-pruned gene co-expression networks built from TCGA expression data for KIRC, LIHC, and PRAD.

## When to use

Use when you have a fused gene co-expression network (e.g., Gephi-format .gexf from the GeneNetFusion pipeline) and need to partition genes into communities for biomarker extraction.

## When NOT to use

- Input is raw TCGA gene expression data and no fused network has been built yet — run Pipeline.ipynb (preprocessing, creation_matrices, fusion_networks) first
- The goal is a differential-expression baseline signature — the article uses DESeq2-identified DE genes as a comparison, not as the community source
- You already have extracted IC gene sets and want classification performance — that is the downstream LASSO + Random Forest step, not community detection

## Inputs

- Fused co-expression network per cancer type (99th-percentile disparity-filter pruned), in Gephi-compatible format from the repository's Graphs folder
- TCGA gene expression matrices from the Datasets_created folder (retrieved via GMQL/PyGMQL)
- Jupyter notebook Pipeline.ipynb executed up to the 'After extraction of communities with Gephi' part

## Outputs

- Gene community lists saved in the Extracted folder as 'IC_'+str(tumor)+'.csv' (e.g., IC_LIHC.csv, IC_KIRC.csv, IC_PRAD.csv)
- IC gene symbols per cancer type, subsequently consumed by LASSO selection and Random Forest normal/cancer classification

## How to apply

First ensure the fused network is pruned: edges in the Euclidean distance networks are kept only above the 99th percentile of edge weights, Pearson's correlation edges pass a permutation-test threshold, and a disparity filter extracts the backbone (retaining on average ~4.9% of Merged-network edges). Load the resulting fused graph (from the repository's Graphs folder, in Gephi-compatible format) into Gephi and run the modularity method to partition genes into communities. Execute Pipeline.ipynb up to the 'After extraction of communities with Gephi' stage, then export the community gene lists to the Extracted folder as 'IC_'+str(tumor)+'.csv'. The rationale is that the fused normal+cancer network per cancer type captures cancer-type-specific structure, so its modularity communities concentrate biologically interesting genes for downstream LASSO selection and Random Forest classification.

## Related tools

- **Gephi** (Runs the modularity community-detection method on the fused, disparity-filter-pruned co-expression networks to extract IC gene communities) — https://gephi.org
- **Jupyter Notebook** (Pipeline.ipynb builds and fuses the co-expression networks up to the point where the graph is handed off to Gephi for community extraction) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
- **Python** (Preprocessing.py, creation_matrices.py, fusion_networks.py and extraction_classification.py implement the network construction steps feeding and following the Gephi modularity step) — https://github.com/DEIB-GECO/GeneNetFusion

## Examples

```
jupyter notebook Pipeline.ipynb  # run until 'After extraction of communities with Gephi', then run modularity in Gephi on Graphs/LIHC fused network and save Extracted/IC_LIHC.csv
```

## Evaluation signals

- Community output files named 'IC_'+str(tumor)+'.csv' exist in the Extracted folder for the target tumor types (LIHC, KIRC, PRAD)
- The input network is the fused network after disparity filtering (on average ~4.9% of Merged-network edges retained), not a raw or Merged network
- Extracted IC genes for LIHC match the gene symbols in the repository's Results folder for the LIHC fused network
- Downstream sanity check: LASSO-selected genes from these communities yield valid Accuracy, AUC and F1 values (0–1) in auc_acc_f1.csv after running extraction_classification.py
- Modularity was run in Gephi on the graph loaded from the Graphs folder, which is provided in the format required by Gephi

## Limitations

- Community extraction is a manual Gephi GUI step: the README instructs users to stop Pipeline.ipynb and use Gephi interactively before resuming, so full automation requires replicating the modularity settings
- The repository itself only ships the LIHC fused network in Graphs folder; KIRC and PRAD networks must be regenerated from TCGA data via GMQL/PyGMQL
- Modularity communities are only meaningful on the pruned fused network — applying it to unpruned or single-type (non-fused) networks departs from the validated workflow

## Evidence

- [methods] gene communities in the fused network are extracted using the modularity method implemented in Gephi: "gene communities in the fused network are extracted using the modularity method implemented in Gephi39"
- [readme] Use Gephi in order to extract the relevant communities and save the genes in the folder Extracted with the name 'IC_'+str(tumor)+'.csv': "Use [Gephi](https://gephi.org) in order to extract the relevant communities and save the genes in the folder [Extracted](https://github.com/DEIB-GECO/GeneNetFusion/tree/master/Extracted) with the"
- [results] these communities are found using the modularity method implemented in the Gephi software: "these communities are found using the modularity method implemented in the Gephi software"
- [results] The relevant edges selected with the disparity filter in the final (Fused) networks are on average about 4.9% of the edges in the Merged ones: "The relevant edges selected with the disparity filter in the final (Fused) networks are on average about 4.9% of the edges in the Merged ones"
- [readme] Execute the jupyter notebook Pipeline.ipynb until the part 'After extraction of communities with Gephi': "Execute the jupyter notebook [Pipeline.ipynb](https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb) until the part 'After extraction of communities with Gephi'"
