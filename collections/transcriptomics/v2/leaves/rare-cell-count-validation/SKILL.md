---
name: rare-cell-count-validation
description: Use when reproducing FiRE rare-cell calls on ~68k-cell scRNA-seq data
  and you must verify both the expected rare-cell count (4238) and cluster-level specificity
  of top-scoring cells.
when_to_use_negative:
- Your goal is distinguishing technical outliers (e.g., doublets) from genuinely rare
  cell types — FiRE does not discriminate between an outlier and cells representing
  minor cell types
- You only need binary rare-cell predictions from pairwise clustering methods such
  as RaceID or GiniClust rather than a graded rareness score per profile
- Your dataset is small enough that clustering alone can chart minor cell types in
  a single pass; FiRE's speed advantage targets voluminous data of tens of thousands
  of profiles
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: FiRE
  role: Assigns a rareness score to every individual expression profile; scores are
    IQR-thresholded and top 0.25% extracted for validation
  repo: https://github.com/princethewinner/FiRE
- name: dropClust
  role: Clusters the FiRE-detected rare cells to provide cluster labels against which
    the top 0.25% scores are cross-referenced (megakaryocyte specificity check)
- name: Python
  role: Environment for loading the ~68k expression matrix with pandas, running the
    FiRE module, computing the IQR cutoff, and counting rare cells
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:8c4a4e8541a7022705f34c127b5d635b8332d399b1d971f45e120fbe8fdb7ca4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/princethewinner/FiRE
  tools:
  - FiRE
  - Python
  - dropClust
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# rare-cell-count-validation

## Summary

Validates FiRE-based rare-cell detection on the ~68k PBMC scRNA-seq dataset (Zheng et al., GSE94820) by checking that IQR-thresholded rare-cell counts equal 4238 and that the top 0.25% highest FiRE scores map exclusively to the megakaryocyte cluster.

## When to use

Use when reproducing FiRE rare-cell calls on ~68k-cell scRNA-seq data and you must verify both the expected rare-cell count (4238) and cluster-level specificity of top-scoring cells.

## When NOT to use

- Your goal is distinguishing technical outliers (e.g., doublets) from genuinely rare cell types — FiRE does not discriminate between an outlier and cells representing minor cell types
- You only need binary rare-cell predictions from pairwise clustering methods such as RaceID or GiniClust rather than a graded rareness score per profile
- Your dataset is small enough that clustering alone can chart minor cell types in a single pass; FiRE's speed advantage targets voluminous data of tens of thousands of profiles

## Inputs

- Single-cell expression matrix of ~68k human blood cells (Zheng et al.), cells x genes, loadable with pandas/numpy from GEO accession GSE94820
- dropClust-derived cluster labels for the ~68k profiles (for cross-referencing top-scoring cells)

## Outputs

- Binary rare/non-rare prediction vector (4238 cells flagged rare via IQR thresholding)
- Per-cell FiRE rareness score array [nCells]
- Validation report confirming top 0.25% highest scores correspond exclusively to the megakaryocyte cluster (~0.3% of cells)

## How to apply

Download the ~68k human blood cell expression profiles (GSE94820) and preprocess with the ranger_preprocess routine (keeping 1000 variable genes, log2 transformed). Fit the FiRE model (L=100 estimators, M=50 sampled features, H=1017881 hash bins, seed=5489), score every cell, and dichotomize rare vs non-rare using the IQR cutoff (score >= Q3 + 1.5*IQR), which should yield 4238 rare cells in ~31 s. Rationale: FiRE scores all profiles individually without an intermediate clustering step, so a second check is needed to confirm biological specificity. Cross-reference the top 0.25% highest-scoring cells against dropClust-derived cluster labels to confirm they correspond solely to the smallest unambiguously annotated cluster (megakaryocytes, ~0.3% of cells), as depicted in Supplementary Figure 1 and Fig. 2a. Pass criteria: rare-cell count equals 4238 AND top 0.25% scores map exclusively to the megakaryocyte cluster.

## Related tools

- **FiRE** (Assigns a rareness score to every individual expression profile; scores are IQR-thresholded and top 0.25% extracted for validation) — https://github.com/princethewinner/FiRE
- **dropClust** (Clusters the FiRE-detected rare cells to provide cluster labels against which the top 0.25% scores are cross-referenced (megakaryocyte specificity check))
- **Python** (Environment for loading the ~68k expression matrix with pandas, running the FiRE module, computing the IQR cutoff, and counting rare cells)

## Examples

```
model = FiRE.FiRE(L=100, M=50, H=1017881, seed=5489); model.fit(preprocessedData); score = np.array(model.score(preprocessedData)); th = np.percentile(score, 75) + 1.5*stats.iqr(score); indIqr = np.where(score >= th)[0]
```

## Evaluation signals

- The IQR-thresholded rare-cell set contains exactly 4238 cells
- FiRE scoring completes in seconds (~31 s reported for ~68k profiles) — a runtime far longer suggests an implementation issue
- The top 0.25% highest FiRE scores map exclusively to the megakaryocyte cluster (~0.3% of cells) per dropClust labels
- FiRE scores are continuous values per cell (not binary), with higher values denoting rarer cells; binary calls only exist after applying score >= Q3 + 1.5*IQR
- Reproduction matches the pattern in Supplementary Figure 1 and Fig. 2a of the article

## Limitations

- FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types; outlier cells can get submerged into minor cell clusters when clustering FiRE-detected rare cells with dropClust
- Doublet rate varies between 1% and 30%, a range that does not necessarily comply with the notion of cell rareness, limiting FiRE's application to doublet detection
- The full jurkat demo dataset may require a large amount of RAM to load and preprocess

## Evidence

- [abstract] abstract: "an algorithm that, in a matter of seconds, assigns a rareness score to every individual expression profile under study"
- [discussion] discussion_runtime: "FiRE took ~31 s to analyze a scRNA-seq dataset containing ~68 k expression profiles."
- [discussion] discussion_dropclust: "For clustering FiRE detected rare cells, in all our analyses, we adhered to dropClust24"
- [discussion] discussion_outlier_limit: "FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types."
- [readme] readme_score_direction: [evidence span withheld — anchor status: external_doc]
- [readme] readme_iqr_threshold: [evidence span withheld — anchor status: external_doc]
