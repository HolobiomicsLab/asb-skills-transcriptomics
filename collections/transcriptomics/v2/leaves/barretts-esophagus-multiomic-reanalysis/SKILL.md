---
name: barretts-esophagus-multiomic-reanalysis
description: Use when single-cell DNA copy-number and RNA expression tables from Barrett's
  esophagus biopsies (NDBE, LGD, HGD, EAC) need cross-modality cell-to-clone assignment
  and per-gene phylogenetic signal (K*) analysis to test whether genomic mutations
  drive transcriptomic heterogeneity.
when_to_use_negative:
- 'The sample is biologically homogeneous (e.g. a non-progressing or healthy homogeneous
  biopsy): the article notes ''matching cells is distinctively challenging in homogeneous
  samples'', a biologically driven issue for MaCroDNA.'
- 'Probabilistic confidence measures per assignment are required: MaCroDNA does not
  provide them, unlike statistical methods such as clonealign.'
- Ground truth cell correspondence is available and accuracy can be measured directly
  — the BE dataset lacks ground truth, which is why resampling and stability analyses
  were needed instead.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3170
tools:
- name: MaCroDNA
  role: Correlation-based method performing cell2clone assignment between scRNA-seq
    gene expression and scDNA-seq copy number via maximum weighted bipartite matching;
    source under src/MaCroDNA/macrodna.py
  repo: https://github.com/NakhlehLab/MaCroDNA
- name: Gurobi Optimizer
  role: Solves the binary integer program (maximum weighted bipartite matching) to
    optimality with tolerance 1.00e-04 during cell2clone_assignment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:502929de89683ad923504f30dd9b69a5d662edacbade622b6cfbf7e4460aaed3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NakhlehLab/MaCroDNA
  tools:
  - Gurobi Optimizer
  - MaCroDNA
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# barretts-esophagus-multiomic-reanalysis

## Summary

Reanalysis of Barrett's esophagus progression biopsies using MaCroDNA to assign scRNA-seq cells to scDNA-seq clones and compute per-gene K* phylogenetic signal, showing that genomic mutations of large effect size increasingly contribute to differential expression as BE progresses to dysplasia and esophageal adenocarcinoma.

## When to use

Use when single-cell DNA copy-number and RNA expression tables from Barrett's esophagus biopsies (NDBE, LGD, HGD, EAC) need cross-modality cell-to-clone assignment and per-gene phylogenetic signal (K*) analysis to test whether genomic mutations drive transcriptomic heterogeneity.

## When NOT to use

- The sample is biologically homogeneous (e.g. a non-progressing or healthy homogeneous biopsy): the article notes 'matching cells is distinctively challenging in homogeneous samples', a biologically driven issue for MaCroDNA.
- Probabilistic confidence measures per assignment are required: MaCroDNA does not provide them, unlike statistical methods such as clonealign.
- Ground truth cell correspondence is available and accuracy can be measured directly — the BE dataset lacks ground truth, which is why resampling and stability analyses were needed instead.

## Inputs

- pandas dataframe dna_df: single-cell DNA copy-number values, genes as row indices, cells as column indices
- pandas dataframe rna_df: single-cell RNA-seq gene expression values, genes as row indices, cells as column indices (gene set may differ from dna_df)
- pandas dataframe dna_label: two columns 'clone' and 'cell' linking each DNA cell to its clone id
- BE biopsy single-cell multiomic tables stratified by progression stage (NDBE, LGD, HGD, EAC)

## Outputs

- cell2clone_assignments.csv: dataframe indexed by RNA cell ids with 'predict_cell' (matched DNA cell) and 'predict_clone' (predicted clone id) columns
- per-gene K* phylogenetic signal values per biopsy, identifying genes with K* > 1 (e.g. ERBB2, COSMIC genes)
- list of genes with statistically significant phylogenetic signal in HGD and EAC biopsies

## How to apply

Load the biopsy's scDNA copy-number table (dna_df) and scRNA expression table (rna_df) as pandas dataframes with genes as row indices and cells as columns, plus the clone-label table (dna_label) with 'clone' and 'cell' columns; gene sets in the two modalities may differ because MaCroDNA selects shared genes itself. Instantiate MaCroDNA(rna_df, dna_df, dna_label) and run macrodna.cell2clone_assignment(), which maximizes the sum of per-gene Pearson correlation coefficients between RNA expression and DNA copy number via maximum weighted bipartite matching solved as a binary integer program with Gurobi (MIP, tolerance 1.00e-04). Save the returned cell2clone dataframe — indexed by RNA cell ids with 'predict_cell' and 'predict_clone' columns — as cell2clone_assignments.csv. Then compute per-gene K* phylogenetic signal on the assigned cells for each biopsy: HGD and EAC biopsies should yield hundreds of genes with inferred K* > 1, while lower-grade or healthy tissue yields far fewer; ERBB2 had the highest inferred K* of statistically significant COSMIC genes in two biopsies.

## Related tools

- **MaCroDNA** (Correlation-based method performing cell2clone assignment between scRNA-seq gene expression and scDNA-seq copy number via maximum weighted bipartite matching; source under src/MaCroDNA/macrodna.py) — https://github.com/NakhlehLab/MaCroDNA
- **Gurobi Optimizer** (Solves the binary integer program (maximum weighted bipartite matching) to optimality with tolerance 1.00e-04 during cell2clone_assignment)

## Examples

```
from MaCroDNA import MaCroDNA; macrodna = MaCroDNA(rna_df, dna_df, dna_label); cell2clone = macrodna.cell2clone_assignment(); cell2clone.to_csv('cell2clone_assignments.csv')
```

## Evaluation signals

- The Gurobi MIP reports 'Optimal solution found' with a zero optimality gap (tolerance 1.00e-04) and 'IsMIP: 1'
- The number of associations in the correspondence matrix equals the expected count of assigned RNA cells (in the installation test, 4 RNA cells yield 4.0 associations)
- The installation test (test_macrodna.tiny_test()) ends with 'Test Success' and exit code 0
- Every RNA cell in rna_df receives a non-null 'predict_cell' and 'predict_clone' entry in the cell2clone output
- Per-biopsy K* results show hundreds of genes with inferred K* > 1 in every HGD and EAC biopsy and far fewer in lower-grade or healthy tissue, with ERBB2 among the significant COSMIC genes

## Limitations

- No ground truth information about the correspondence between cells exists for the BE dataset, so performance is assessed only via resampling and stability analyses
- MaCroDNA does not provide probabilistic confidence measures for the assignments, unlike statistical methods such as clonealign
- Matching cells is distinctively challenging in homogeneous samples — a biologically driven issue when applying MaCroDNA to such cases
- Only gurobipy must be installed manually (free academic license); MaCroDNA requires Python >= 3.7 plus numpy, pandas, scipy, and math

## Evidence

- [results] Applying MaCroDNA to BE biopsies and computing per-gene K* phylogenetic signal identified hundreds of genes with inferred K* > 1 in every HGD and EAC biopsy: [evidence span withheld — anchor status: not_found]
- [discussion] MaCroDNA finds correspondence by maximizing Pearson correlations: "Our method, named MaCroDNA, finds the correspondence between the scRNA-seq and scDNA-seq cells by maximizing the sum of the Pearson correlation coefficients"
- [readme] README usage: instantiate MaCroDNA with three dataframes and run cell2clone_assignment: "macrodna = MaCroDNA(rna_df, dna_df, dna_label)
> cell2clone = macrodna.cell2clone_assignment()"
- [readme] README installation test expected output confirms MIP optimality and association count: "Optimal solution found (tolerance 1.00e-04)
Best objective 2.830007718086e+00, best bound 2.830007718086e+00, gap 0.0000%"
- [methods] Article methods: MaCroDNA source location: "MaCroDNA is under the directory `src/MaCroDNA/macrodna.py`"
