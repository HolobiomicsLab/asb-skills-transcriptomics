---
name: regularization-parameter-sweep-analysis
description: Use when you must attribute performance contributions of individual regularization
  penalties in a regularized regression model, or determine how the network regularization
  parameter η affects enrichment performance under varying noise conditions.
when_to_use_negative:
- The goal is a single biological enrichment analysis on real differential expression
  data rather than a controlled ablation of regularization terms — use a plain pareg
  fit instead.
- No ground-truth active pathways (true positives) are available, so precision-recall
  AUC cannot be computed against known positives.
- The model under study does not expose distinct regularization terms (e.g. an unregularized
  ORA/GSEA method) that could be toggled independently.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: pareg
  role: Fits the regularized generalized linear model under four regularization configurations
    and provides compute_term_similarities, similarity_sample, and network regularization
    via term_network/network_param
  repo: https://github.com/cbg-ethz/pareg
- name: R
  role: Language in which pareg is distributed on Bioconductor and the sweep/evaluation
    scripts are executed
  repo: https://bioconductor.org/packages/release/bioc/html/pareg.html
- name: Snakemake
  role: Workflow engine for reproducible execution of the ablation sweep pipelines
  repo: https://github.com/cbg-ethz/pareg
- name: tidyverse
  role: Data manipulation and summarization of the PR-AUC results across configurations
    and η values
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad522
    title: Coherent pathway enrichment estimation by modeling inter-pathway dependencies
      using regularized regression
  source_content_hash: sha256:cfa7f56f6319f9061b804c6f01732659ab879b22216108c6062ead2d0a983e29
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cbg-ethz/pareg
  tools:
  - R
  - Snakemake
  - pareg
  - tidyverse
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# regularization-parameter-sweep-analysis

## Summary

Systematically sweeps pareg's regularization configurations (none, LASSO-only, network-only, network+LASSO) and the network regularization parameter η over a grid (e.g. 0.25 and 0.75), scoring each fit by precision-recall AUC against known active pathways to quantify how each penalty term contributes to performance at different noise levels.

## When to use

Use when you must attribute performance contributions of individual regularization penalties in a regularized regression model, or determine how the network regularization parameter η affects enrichment performance under varying noise conditions.

## When NOT to use

- The goal is a single biological enrichment analysis on real differential expression data rather than a controlled ablation of regularization terms — use a plain pareg fit instead.
- No ground-truth active pathways (true positives) are available, so precision-recall AUC cannot be computed against known positives.
- The model under study does not expose distinct regularization terms (e.g. an unregularized ORA/GSEA method) that could be toggled independently.

## Inputs

- Synthetic pathway database (data frame of pathway/gene memberships with 2 pathway groups of 10 pathways each)
- Pairwise pathway similarity matrix computed with compute_term_similarities(df_terms, similarity_function = jaccard)
- Simulated gene p-values (Beta(0.1, 1) for differentially expressed genes, Uniform for background genes)
- Ground-truth set of 5 active pathways selected with similarity_sample(mat_similarities, 5)

## Outputs

- PR-AUC table per pareg variant (pareg_noterms, pareg_lasso, pareg_network, pareg_network_lasso) and per network_param value (η = 0.25, 0.75)
- Relative performance ranking of regularization variants reproducing the Supplementary Fig S1 pattern

## How to apply

Generate a synthetic pathway database with pronounced pathway clustering (2 groups of 10 pathways, 15 group genes plus 50 background genes per group, each pathway containing 10 group and 10 background genes), compute the pairwise similarity matrix with pareg's compute_term_similarities using the Jaccard similarity_function, and confirm group structure with a ComplexHeatmap heatmap. Sample 5 active pathways as true positives via similarity_sample(mat_similarities, 5), define de_genes as the union of their genes, and draw p-values from Beta(0.1, 1) for de_genes and a Uniform distribution for all other genes. Fit pareg in the four regularization configurations (pareg_noterms, pareg_lasso, pareg_network, pareg_network_lasso), passing term_network = mat_similarities and sweeping network_param (η) over {0.25, 0.75}. Compute PR-AUC for each variant and η from the fitted enrichment scores using the selected active terms as ground-truth positives; the rationale is that the joint network+LASSO penalty should rank highest at both noise levels, with LASSO contributing more at low noise and network fusion more at high noise.

## Related tools

- **pareg** (Fits the regularized generalized linear model under four regularization configurations and provides compute_term_similarities, similarity_sample, and network regularization via term_network/network_param) — https://github.com/cbg-ethz/pareg
- **R** (Language in which pareg is distributed on Bioconductor and the sweep/evaluation scripts are executed) — https://bioconductor.org/packages/release/bioc/html/pareg.html
- **Snakemake** (Workflow engine for reproducible execution of the ablation sweep pipelines) — https://github.com/cbg-ethz/pareg
- **tidyverse** (Data manipulation and summarization of the PR-AUC results across configurations and η values)

## Examples

```
fit <- pareg(df_genes, df_terms, term_network = mat_similarities, network_param = 0.75); active_terms <- similarity_sample(mat_similarities, 5)
```

## Evaluation signals

- pareg_network_lasso must achieve the highest PR-AUC among all four variants at BOTH η = 0.25 and η = 0.75, matching the Supplementary Fig S1 ordering.
- ComplexHeatmap of the Jaccard similarity matrix must show pronounced clustering of the two synthetic pathway groups before any fitting.
- The p-value distribution check: de_genes p-values drawn from Beta(0.1, 1), all other genes from Uniform.
- PR-AUC is computed with the 5 similarity-sampled active terms as ground-truth positives for every configuration, ensuring comparability across the sweep.
- Relative ordering between single-penalty variants should shift with η: LASSO relatively stronger at low noise, network fusion relatively stronger at high noise.

## Limitations

- Results are demonstrated on a synthetic pathway database with designed group structure; transferability to real pathway databases and real noise regimes must be verified separately.
- Only two network_param values (η = 0.25, 0.75) are swept in the benchmark, so the full response surface of η is not characterized.
- The evaluation depends on simulated p-values (Beta/Uniform), which may not capture realistic differential expression signal structure.

## Evidence

- [supplementary] In both noise settings (η=0.25 and η=0.75), the pareg variant featuring both regularization terms achieves the best performance: [evidence span withheld — anchor status: not_found]
- [abstract] We show that pareg is more robust to noise than competing methods.: "We show that pareg is more robust to noise than competing methods."
- [methods] The p-values of genes considered to be differentially expressed are sampled from a Beta distribution centered at $0$.: "The p-values of genes considered to be differentially expressed are sampled from a Beta distribution centered at $0$. The p-values for all other genes are drawn from a Uniform distribution."
- [methods] we generate a synthetic pathway database with a pronounced clustering of pathways: "we generate a synthetic pathway database with a pronounced clustering of pathways"
- [readme] Pathway enrichment computations using a regularized regression approach to incorporate inter-pathway relations in the statistical model.: "Pathway enrichment computations using a regularized regression approach to incorporate inter-pathway relations in the statistical model."
