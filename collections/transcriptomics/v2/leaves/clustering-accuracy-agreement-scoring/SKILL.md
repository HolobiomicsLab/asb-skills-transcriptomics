---
name: clustering-accuracy-agreement-scoring
description: Use when you must rank pipeline configurations (e.g., integration method
  × cost-matrix similarity metric such as Cosine vs. Euclidean) by how well their
  sample-level clusters or trajectory orderings agree with true disease-status labels.
when_to_use_negative:
- 'You have no ground-truth sample labels: PILOT itself is unsupervised, and labels
  are only used here for post-hoc validation of configurations'
- You need to score cell-level annotations rather than sample-level clusters — PILOT's
  unit of analysis is the individual single-cell sample, not the cell
- The sample-level Wasserstein distance matrix has not been computed yet for the configurations
  being compared (this skill scores, it does not generate distances)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: PILOT
  role: Produces the sample-level Wasserstein distance matrix, Leiden sample clusters,
    and EIPLGraph trajectories that are scored for agreement
  repo: https://github.com/CostaLab/PILOT
- name: POT library
  role: emd2 solver used by PILOT's wasserstein_distance to compute the per-configuration
    Wasserstein distances
- name: scanpy
  role: PCA embedding of cells and Leiden clustering workflow context
- name: Leiden clustering
  role: Clusters the sample distance matrix W at the resolution chosen by best silhouette
    score before ARI/NMI scoring
- name: pandas
  role: Tabulates the performance of all integration-method × similarity-metric combinations
    into the CSV comparison table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:d085a5f88d7c554dcb4adce920daaf0fefea4411db889f87effba8b0ac33d3c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - Leiden clustering
  - PILOT
  - POT library
  - pandas
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# clustering-accuracy-agreement-scoring

## Summary

Scores unsupervised sample-level outputs of PILOT (Leiden clusters on the Wasserstein matrix W and EIPLGraph trajectories) by their agreement with known disease-status labels, using ARI/NMI for clustering and label concordance for trajectory ordering. This converts an unsupervised pipeline into a ranked comparison of configurations, e.g. integration-method × cost-matrix metric grids as in Appendix Fig. S5.

## When to use

Use when you must rank pipeline configurations (e.g., integration method × cost-matrix similarity metric such as Cosine vs. Euclidean) by how well their sample-level clusters or trajectory orderings agree with true disease-status labels.

## When NOT to use

- You have no ground-truth sample labels: PILOT itself is unsupervised, and labels are only used here for post-hoc validation of configurations
- You need to score cell-level annotations rather than sample-level clusters — PILOT's unit of analysis is the individual single-cell sample, not the cell
- The sample-level Wasserstein distance matrix has not been computed yet for the configurations being compared (this skill scores, it does not generate distances)

## Inputs

- AnnData (.h5ad) single-cell matrix with cells embedded in PCA space (e.g., GSE174188_CLUES1_adjusted.h5ad from the Lupus PBMC study)
- Sample-level Wasserstein distance matrix W produced by PILOT's wasserstein_distance per configuration
- True disease-status labels for each sample/patient (e.g., control vs. disease status)

## Outputs

- Per-configuration ARI and NMI scores for Leiden sample clusters vs. disease-status labels
- Label-concordance scores for predicted trajectory orderings
- CSV table of integration-method × similarity-metric performance reproducing the layout of Appendix Fig. S5

## How to apply

Run each PILOT configuration end-to-end: compute sample-level Wasserstein distances (emd2 from POT), then cluster the distance matrix W with Leiden, selecting the resolution from the silhouette-score curve via select_best_sil (start = 0.2). Score the resulting sample clusters against the true disease-status labels (e.g., lupus status from GSE174188) with ARI and NMI; for trajectory configurations, score the concordance of the predicted sample ordering with disease status, using a control sample as the EIPLGraph source node. Tabulate ARI/NMI/concordance for every integration-method × similarity-metric combination (Cosine, Euclidean, Manhattan, Chebyshev) in a CSV mirroring the Appendix Fig. S5 layout. Accept the table if it confirms the Cosine cost-matrix preference and the limited impact of the integration-method choice.

## Related tools

- **PILOT** (Produces the sample-level Wasserstein distance matrix, Leiden sample clusters, and EIPLGraph trajectories that are scored for agreement) — https://github.com/CostaLab/PILOT
- **POT library** (emd2 solver used by PILOT's wasserstein_distance to compute the per-configuration Wasserstein distances)
- **scanpy** (PCA embedding of cells and Leiden clustering workflow context)
- **Leiden clustering** (Clusters the sample distance matrix W at the resolution chosen by best silhouette score before ARI/NMI scoring)
- **pandas** (Tabulates the performance of all integration-method × similarity-metric combinations into the CSV comparison table)

## Examples

```
from sklearn.metrics import adjusted_rand_score, normalized_mutual_info_score; ari = adjusted_rand_score(disease_status_labels, leiden_sample_clusters); nmi = normalized_mutual_info_score(disease_status_labels, leiden_sample_clusters)
```

## Evaluation signals

- ARI and NMI are computed between Leiden sample clusters and the true disease-status labels for every configuration
- The Cosine cost-matrix configuration should rank first and be the only metric outperforming Euclidean, Manhattan, and Chebyshev in most scenarios
- Leiden resolution must be selected via the silhouette-score curve (select_best_sil, start = 0.2) rather than fixed arbitrarily
- The output CSV reproduces the integration-method × similarity-metric layout of Appendix Fig. S5
- Agreement is evaluated at the sample level (patients), consistent with PILOT's Wasserstein-distance unit of analysis

## Limitations

- PILOT's clustering and trajectory analyses are unsupervised; ground-truth labels are only used post-hoc, so agreement scores validate but do not drive the predictions
- The EIPLGraph root sample was visually selected to reflect control samples rather than chosen by an objective rule, which limits reproducibility of trajectory concordance scores
- PILOT does not explicitly correct for technical artefacts (e.g., frozen vs. fresh PBMCs, 10X-only lung samples), so agreement scores may be confounded by batch effects

## Evidence

- [abstract] PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples. This allows us to perform unsupervised analysis at the sample level: "PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples. This allows us to perform unsupervised analysis at the sample level"
- [discussion] Clustering analysis can be performed by providing W as input to a Leiden clustering algorithm (Traag et al, 2019).: "Clustering analysis can be performed by providing W as input to a Leiden clustering algorithm (Traag et al, 2019)."
- [intro] we perform a benchmarking study to compare PILOT and competing approaches on their performance in 12 public single-cell and pathomics data sets: "we perform a benchmarking study to compare PILOT and competing approaches on their performance in 12 public single-cell and pathomics data sets"
- [supplementary] Cosine similarity ranked first and was the only metric outperforming other approaches (Euclidean, Manhattan, Chebyshev) in most scenarios: [evidence span withheld — anchor status: not_found]
