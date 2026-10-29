---
name: benchmark-ground-truth-label-chaining
description: Use when a cell hashing Seurat object and published SNP-based (scSplit)
  ground-truth labels are both available and you need to correlate hashing singlet
  groups with genotype clusters to produce validated per-cell sample identity labels.
when_to_use_negative:
- Samples lack a distinct genetic background (most animal studies or same-donor longitudinal
  studies), so SNP-based ground-truth labels cannot be produced for chaining
- Only cell hashing data is available with no SNP/genotype labels — use HTOreader's
  standalone HTOClassification cutoff calling instead
- Input is raw HTO counts that have not yet been demultiplexed by the upstream hashtag
  step (HTOClassification/PlotHTO cutoff checks)
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: HTOreader
  role: Provides the HybridDemultiplexing function that correlates cell hashing and
    genotype labels and applies the convergence-threshold decision; installed via
    devtools from GitHub
  repo: https://github.com/WilsonImmunologyLab/HTOreader
- name: scSplit
  role: Source of the published ground-truth genotype labels used as the SNP-based
    result for benchmark chaining
- name: Seurat
  role: Holds the HTO assay and cell hashing labels as the Seurat object input to
    HybridDemultiplexing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae254
    title: A hybrid demultiplexing strategy that improves performance and robustness
      of cell hashing
  source_content_hash: sha256:803d00c38499ccdf4331b2d7299a01ad627d8dd392b6913919f57d34b0884abb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WilsonImmunologyLab/HTOreader
  tools:
  - HTOreader
  - Seurat
  - scSplit
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# benchmark-ground-truth-label-chaining

## Summary

Chains published ground-truth genotype labels (e.g. scSplit labels from the Stoeckius-2018 dataset) with HTOreader's HybridDemultiplexing function to assign final cell sample identities, using a convergence-score threshold (c_threshold = 0.7, C = N1/(N1+N2)) to decide between merged hashing/genotype labels and cell-hashing fallback labels.

## When to use

Use when a cell hashing Seurat object and published SNP-based (scSplit) ground-truth labels are both available and you need to correlate hashing singlet groups with genotype clusters to produce validated per-cell sample identity labels.

## When NOT to use

- Samples lack a distinct genetic background (most animal studies or same-donor longitudinal studies), so SNP-based ground-truth labels cannot be produced for chaining
- Only cell hashing data is available with no SNP/genotype labels — use HTOreader's standalone HTOClassification cutoff calling instead
- Input is raw HTO counts that have not yet been demultiplexed by the upstream hashtag step (HTOClassification/PlotHTO cutoff checks)

## Inputs

- Hashing-labeled Seurat object (HTO assay with cell hashing singlet/doublet/negative labels from HTOClassification)
- Published scSplit ground-truth genotype labels for the Stoeckius-2018 dataset (used as the SNP-based result)
- HTO names (hto_names) and pair_info linking hashing singlet groups to genotype clusters
- c_threshold (default 0.7) and label-name arguments: snp_doublet_label, snp_unassign_label, cellhashing_doublet_label, cellhashing_negative_label

## Outputs

- hybrid_demultiplexed_labels.csv with one row per cell carrying a final identity label (singlet/doublet/negative designation) from the 0.7 convergence-threshold decision or the three-case inconsistent-label rules
- Warning messages for hashing-group/genotype-cluster pairs whose convergence score falls below c_threshold

## How to apply

Load the hashing-labeled Seurat object (from upstream HTOClassification/hashtag demultiplexing) together with the published scSplit ground-truth genotype labels, treating the scSplit labels as the SNP-based result. Invoke HTOreader's HybridDemultiplexing with cellhashing_label, genotype_label, hto_names, c_threshold, the snp_doublet_label / snp_unassign_label / cellhashing_doublet_label / cellhashing_negative_label arguments, and pair_info to correlate hashing singlet groups with genotype clusters. For each hashing-group/genotype-cluster pair compute the convergence score C = N1/(N1+N2) from concordant and discordant cell counts; pairs with C >= 0.7 are accepted and merged. For pairs falling below the 0.7 threshold (e.g. the 9pool-CA scores of 0.1 and 0.55), fall back to the cell hashing labels and emit a warning that demultiplexing results may not be accurate. Resolve remaining disagreements with the three-case inconsistent-label assignment rules (reclassifying snp_doublet, snp_unassign, cellhashing_doublet, and cellhashing_negative labels), then write per-cell identities including singlet/doublet/negative designations to hybrid_demultiplexed_labels.csv.

## Related tools

- **HTOreader** (Provides the HybridDemultiplexing function that correlates cell hashing and genotype labels and applies the convergence-threshold decision; installed via devtools from GitHub) — https://github.com/WilsonImmunologyLab/HTOreader
- **scSplit** (Source of the published ground-truth genotype labels used as the SNP-based result for benchmark chaining)
- **Seurat** (Holds the HTO assay and cell hashing labels as the Seurat object input to HybridDemultiplexing)

## Examples

```
hybrid <- HybridDemultiplexing(object, cellhashing_label, genotype_label, hto_names, c_threshold = 0.7, snp_doublet_label, snp_unassign_label, cellhashing_doublet_label, cellhashing_negative_label, pair_info)
```

## Evaluation signals

- hybrid_demultiplexed_labels.csv exists with exactly one row per input cell (row_count_equals check)
- Every cell carries a final label produced by either the 0.7 convergence-threshold decision or one of the three-case inconsistent-label rules (field_present check)
- Each hashing-group/genotype-cluster pair has a computed convergence score C = N1/(N1+N2); pairs with C >= 0.7 are merged, pairs below (e.g. 0.1 and 0.55 in 9pool-CA) fall back to cell hashing labels
- A warning is displayed for every pair whose convergence score falls below c_threshold
- Cells labeled snp_doublet, snp_unassign, cellhashing_doublet, or cellhashing_negative are reclassified by the three-case inconsistent-label rules rather than passed through unchanged

## Limitations

- The performance of the hybrid strategy is largely dependent on the quality of SNP-based clustering, which can degrade for cell populations with abnormal gene expression profiles (e.g. plasmablasts or dying cells), high true doublet rates, and poor transcriptome quality
- SNP-based demultiplexing only works with samples that have a distinct genetic background, so this chaining approach is not applicable in most animal studies or longitudinal studies with samples from the same donors
- When the convergence score falls below the threshold, final labels rely on cell hashing alone and results may not be accurate — the warning must not be ignored

## Evidence

- [methods] HybridDemultiplexing function signature: "- `HybridDemultiplexing(object, cellhashing_label, genotype_label, hto_names, c_threshold, snp_doublet_label, snp_unassign_label, cellhashing_doublet_label, cellhashing_negative_label, pair_info)`"
- [intro] Self-validating nature of hybrid approach: "this hybrid approach is inherently self-validating; potential errors that arise from one method can be easily identified by the other"
- [abstract] Correlating hashing and genetic clustering results: "This approach correlates the results of cell hashing and genetic variant clustering, enabling precise and efficient cell identity determination"
- [discussion] Hybrid strategy definition: "we developed a hybrid demultiplexing strategy that combines cell-hashing-based methods with SNP-based methods"
- [readme] HTOreader installation from GitHub: "`devtools::install_github("WilsonImmunologyLab/HTOreader")`"
- [readme] HTO assay loading requirement: "The `pbmc.hashtag` in the following example is a Seurat object. Please load your "HTO" assay using the Seurat package"
