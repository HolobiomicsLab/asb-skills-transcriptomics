---
name: convergence-score-threshold-decision
description: Use when both cell-hashing singlet labels and SNP-based genotype labels
  (e.g. from scSplit) exist for the same cells and per-cell sample identities must
  be reconciled via a convergence-score cutoff decision.
when_to_use_negative:
- Samples lack a distinct genetic background (e.g. most animal studies or same-donor
  longitudinal studies), so no reliable SNP-based genotype labels exist to correlate
  with hashing labels
- SNP-based clustering quality is poor — e.g. plasmablasts or dying cells, high true
  doublet rates, or poor transcriptome quality — since the hybrid decision is largely
  dependent on SNP clustering quality
- Only cell hashing data are available and the goal is per-hashtag cutoff calling;
  use HTOClassification/HTOcutoff instead of hybrid demultiplexing
edam_operation: http://edamontology.org/operation_3625
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: HTOreader
  role: Provides the HybridDemultiplexing function that computes convergence scores
    and applies the c_threshold decision to reconcile cell hashing and genotype labels
  repo: github.com/WilsonImmunologyLab/HTOreader
- name: Seurat
  role: Hosts the HTO assay and hashing labels in a Seurat object that is passed into
    HybridDemultiplexing
- name: scSplit
  role: Source of SNP-based genotype cluster labels used as the genetic-variant result
    correlated against cell hashing labels
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae254
    title: A hybrid demultiplexing strategy that improves performance and robustness
      of cell hashing
  source_content_hash: sha256:b89060a6b1068fb12a798489c2dac76ae147648e4b02de841d0e5944ca9c1297
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HTOreader
  - Seurat
  - scSplit
  edam_operation: http://edamontology.org/operation_3625
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# convergence-score-threshold-decision

## Summary

Decision logic from HTOreader's HybridDemultiplexing function that computes a convergence score C = N1/(N1+N2) for each cell-hashing-group/SNP-genotype-cluster pair and merges pairs only when C meets the c_threshold (e.g. 0.7), falling back to cell hashing labels with a warning when the threshold fails.

## When to use

Use when both cell-hashing singlet labels and SNP-based genotype labels (e.g. from scSplit) exist for the same cells and per-cell sample identities must be reconciled via a convergence-score cutoff decision.

## When NOT to use

- Samples lack a distinct genetic background (e.g. most animal studies or same-donor longitudinal studies), so no reliable SNP-based genotype labels exist to correlate with hashing labels
- SNP-based clustering quality is poor — e.g. plasmablasts or dying cells, high true doublet rates, or poor transcriptome quality — since the hybrid decision is largely dependent on SNP clustering quality
- Only cell hashing data are available and the goal is per-hashtag cutoff calling; use HTOClassification/HTOcutoff instead of hybrid demultiplexing

## Inputs

- Seurat object with cell-hashing singlet/doublet/negative labels from upstream hashtag demultiplexing (e.g. HTOClassification output)
- SNP-based genotype labels per cell (e.g. scSplit cluster assignments)
- pair_info mapping hashing singlet groups to genotype clusters
- c_threshold value (e.g. 0.7) and special-label names (snp_doublet_label, snp_unassign_label, cellhashing_doublet_label, cellhashing_negative_label)

## Outputs

- hybrid_demultiplexed_labels.csv with one row per cell carrying a final singlet/doublet/negative sample identity
- Warning messages for hashing/genotype pairs whose convergence score falls below the threshold

## How to apply

Load the hashing-labeled Seurat object together with the SNP-based genotype labels (e.g. scSplit labels used as ground truth for the Stoeckius-2018 dataset) and call HTOreader's HybridDemultiplexing, supplying cellhashing_label, genotype_label, hto_names, c_threshold, the special-label arguments (snp_doublet_label, snp_unassign_label, cellhashing_doublet_label, cellhashing_negative_label), and pair_info linking hashing groups to genotype clusters. For each hashing-group/genotype-cluster pair compute the convergence score C = N1/(N1+N2), where N1 and N2 are the counts of concordant and discordant cells (e.g. the 9pool-CA pairs with scores of 0.1 and 0.55). Apply the decision rule: pairs with C >= c_threshold (0.7 in the reference workflow) are accepted and merged; pairs failing the threshold keep their cell hashing labels and a warning is displayed alerting the user that the demultiplexing results may not be accurate. Resolve remaining inconsistent cells with the three-case inconsistent-label assignment rules covering snp_doublet, snp_unassign, cellhashing_doublet, and cellhashing_negative labels.

## Related tools

- **HTOreader** (Provides the HybridDemultiplexing function that computes convergence scores and applies the c_threshold decision to reconcile cell hashing and genotype labels) — github.com/WilsonImmunologyLab/HTOreader
- **Seurat** (Hosts the HTO assay and hashing labels in a Seurat object that is passed into HybridDemultiplexing)
- **scSplit** (Source of SNP-based genotype cluster labels used as the genetic-variant result correlated against cell hashing labels)

## Examples

```
HybridDemultiplexing(object, cellhashing_label, genotype_label, hto_names, c_threshold, snp_doublet_label, snp_unassign_label, cellhashing_doublet_label, cellhashing_negative_label, pair_info)
```

## Evaluation signals

- hybrid_demultiplexed_labels.csv exists with one row per cell and every cell carries a final label from the 0.7 convergence-threshold decision or the three-case inconsistent-label rules (row_count_equals and field_present checks)
- Every hashing-group/genotype-cluster pair in pair_info has a recorded convergence score C = N1/(N1+N2); pairs with C >= 0.7 are merged, pairs below retain cell hashing labels
- A warning message is emitted for every pair failing the c_threshold (e.g. the 9pool-CA pairs with scores 0.1 and 0.55)
- No cell retains an unresolved snp_doublet, snp_unassign, cellhashing_doublet, or cellhashing_negative label after the three-case rules are applied
- Concordance between final hybrid labels and SNP-based ground-truth labels improves relative to cell hashing alone on benchmark data

## Limitations

- Performance of the threshold decision is largely dependent on the quality of SNP-based clustering; low-quality genotype clusters propagate into hybrid labels
- SNP-based demultiplexing is not applicable when samples lack distinct genetic backgrounds, limiting the hybrid strategy to such datasets
- A below-threshold pair silently degrades to hashing-only labels (with a warning), so demultiplexing results may not be accurate for those cells

## Evidence

- [methods] convergence score C = N1/(N1+N2): "This approach correlates the results of cell hashing and genetic variant clustering, enabling precise and efficient cell identity determination without additional experimental costs or efforts."
- [methods] hybrid decision rule: "HybridDemultiplexing(object, cellhashing_label, genotype_label, hto_names, c_threshold, snp_doublet_label, snp_unassign_label, cellhashing_doublet_label, cellhashing_negative_label, pair_info)"
- [intro] self-validating rationale: "this hybrid approach is inherently self-validating; potential errors that arise from one method can be easily identified by the other"
- [discussion] dependence on SNP clustering: "the performance of the hybrid strategy is largely dependent on the quality of SNP-based clustering"
- [readme] README function context: "There are two main functions, `HTOClassification` for Hashtag demultiplexing, `PlotHTO` for plotting distributions of hashtags"
