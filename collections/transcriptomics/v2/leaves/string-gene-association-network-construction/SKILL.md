---
name: string-gene-association-network-construction
description: Use when validating that gene modules recovered from single-cell feature
  selection (e.g., DELVE dynamic vs. static modules on CD8+ T cell LCMV data) carry
  experimental evidence of co-regulation rather than noise.
when_to_use_negative:
- The input features are not grouped into modules (e.g., a flat ranked feature list)
  — the test is defined per-module with size-matched nulls
- Curated/experimental interaction evidence is unavailable for the organism or gene
  set (e.g., poorly annotated species), making the STRING experimental-score null
  uninformative
- The goal is de novo interaction prediction rather than statistical validation of
  co-regulation in already-defined gene sets
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: DELVE
  role: Produces the dynamic and static feature modules whose gene memberships are
    validated for co-regulation by this skill
  repo: https://github.com/jranek/delve
- name: Python
  role: 'Implementation environment: fetches STRING scores, builds networkx graphs,
    and runs the permutation test'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:0acfd5c7d275eedc3fa4821f7d0d0e52fce0161e7cfc01a457a7b78bb8fc47e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve
  tools:
  - DELVE
  - Python
  - UMAP (uniform manifold approximation and projection)
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# string-gene-association-network-construction

## Summary

Builds gene association networks from experimentally-derived STRING scores for each DELVE-identified feature module and tests whether modules (dynamic vs. static) show statistically significant co-regulation via a one-sided permutation test on average network degree.

## When to use

Use when validating that gene modules recovered from single-cell feature selection (e.g., DELVE dynamic vs. static modules on CD8+ T cell LCMV data) carry experimental evidence of co-regulation rather than noise.

## When NOT to use

- The input features are not grouped into modules (e.g., a flat ranked feature list) — the test is defined per-module with size-matched nulls
- Curated/experimental interaction evidence is unavailable for the organism or gene set (e.g., poorly annotated species), making the STRING experimental-score null uninformative
- The goal is de novo interaction prediction rather than statistical validation of co-regulation in already-defined gene sets

## Inputs

- Gene-to-module assignments from DELVE feature-module identification (three dynamic modules and one static module)
- Gene symbols/identifiers for each module
- Experimentally-derived association scores fetched from the STRING database
- Preprocessed single-cell counts matrix (e.g., GSE131847 mouse splenic CD8+ T cell LCMV data, 29893 cells x 2000 genes, TPM-normalized, log+1 transformed)

## Outputs

- Per-module gene association networks (networkx graphs with nodes as genes and edges weighted by experimental evidence)
- Per-module one-sided permutation p-values against size-matched random gene sets
- CSV of feature-to-module assignments and per-module permutation p-values (cd8_module_string_permutation_results.csv)

## How to apply

After DELVE feature-module identification (k=10 nearest neighbors, num_subsamples=1000, n_clusters=5, 10 random KMeans++ initializations) assigns genes to modules, fetch experimentally-derived association scores from the STRING database for the genes in each module. Build a per-module gene association network with networkx (nodes = genes, edges weighted by experimental evidence score). For each module, run a one-sided permutation test with R = 1000 random gene sets sampled without replacement and size-matched to the module, computing the null distribution T_r of average network degree against the observed T_obs, with p-value = (N + 1)/(R + 1). The rationale is that dynamic modules built from core regulatory complexes should be enriched for co-regulated gene pairs while static (noisy) modules should not, directly testing DELVE's bottom-up module assumption.

## Related tools

- **DELVE** (Produces the dynamic and static feature modules whose gene memberships are validated for co-regulation by this skill) — https://github.com/jranek/delve
- **Python** (Implementation environment: fetches STRING scores, builds networkx graphs, and runs the permutation test)

## Examples

```
import networkx as nx; G = nx.Graph(); G.add_weighted_edges_from([(g1, g2, s) for g1, g2, s in string_module_edges]); T_obs = sum(d for _, d in G.degree()) / G.number_of_nodes()
```

## Evaluation signals

- All three dynamic modules (early activation/proliferation, effector, memory) yield statistically significant co-regulation with p-value = 0.001
- The static module yields no evidence of co-regulation with p-value = 1.0, matching the reported Fig. 5b result
- Each null distribution uses R = 1000 random gene sets sampled without replacement and size-matched to the module, with p-value = (N + 1)/(R + 1)
- Output CSV (cd8_module_string_permutation_results.csv) contains one feature-to-module assignment row per gene and one permutation p-value per module
- Per-module networkx graphs have nodes equal to the module's gene count and edge weights drawn only from STRING experimental-evidence scores

## Limitations

- STRING association scores are curated/experimental evidence rather than direct measurements; significance reflects enrichment of known interactions, not proof of novel co-regulation
- The permutation null is size-matched per module, so modules of very different sizes are tested against different null distributions and p-values are not directly comparable across modules
- Feature selection can bias interpretation of the underlying cellular trajectory; the article cautions that 'careful consideration should be made when performing feature selection for trajectory analysis', which extends to downstream module validation

## Evidence

- [methods] STRING network construction and permutation workflow: [evidence span withheld — anchor status: not_found]
- [methods] Permutation test definition: [evidence span withheld — anchor status: not_found]
- [results] Expected result reproducing Fig. 5b: [evidence span withheld — anchor status: not_found]
- [readme] DELVE module premise grounding the co-regulation test: "DELVE uses a bottom-up approach to mitigate the effect of unwanted sources of feature variation confounding inference, and instead models cell states from dynamic feature modules that constitute core"
