---
name: cell-hashing-snp-label-correlation
description: Use when a dataset has both cell hashing (HTO) demultiplexing labels
  and SNP/genotype-based demultiplexing labels (e.g. scSplit) and per-cell sample
  identities must be reconciled into a final assignment.
when_to_use_negative:
- Samples lack a distinct genetic background (most animal studies or same-donor longitudinal
  studies), so SNP-based demultiplexing cannot provide genotype labels to correlate
- Only cell hashing data is available with no SNP-based demultiplexing result — use
  HTOClassification cutoff calling instead
- SNP-based clustering quality is poor (e.g. plasmablast or dying-cell populations
  with abnormal expression, high true doublet rates), since hybrid performance is
  largely dependent on SNP clustering quality
edam_operation: http://edamontology.org/operation_3741
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: HTOreader
  role: Provides the HybridDemultiplexing function that computes convergence scores
    and applies the c_threshold decision and three-case inconsistent-label rules
  repo: https://github.com/WilsonImmunologyLab/HTOreader
- name: Seurat
  role: Hosts the Seurat object with the 'HTO' assay and hashing labels used as cellhashing_label
    input
- name: scSplit
  role: Source of the SNP-based genotype labels treated as genotype_label input in
    the benchmark workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae254
    title: A hybrid demultiplexing strategy that improves performance and robustness
      of cell hashing
  source_content_hash: sha256:e53c44c58c201128b46b0cb3e9b3fce5a2a6371d87d61bce95ef8c5cccc45e85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WilsonImmunologyLab/HTOreader
  tools:
  - HTOreader
  - Seurat
  - scSplit
  edam_operation: http://edamontology.org/operation_3741
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cell-hashing-snp-label-correlation

## Summary

Correlates cell hashing singlet assignments with SNP-based genotype cluster labels via HTOreader's HybridDemultiplexing function, computing a per-pair convergence score C = N1/(N1+N2) and resolving disagreeing cells with a 0.7 c_threshold decision and three-case inconsistent-label rules.

## When to use

Use when a dataset has both cell hashing (HTO) demultiplexing labels and SNP/genotype-based demultiplexing labels (e.g. scSplit) and per-cell sample identities must be reconciled into a final assignment.

## When NOT to use

- Samples lack a distinct genetic background (most animal studies or same-donor longitudinal studies), so SNP-based demultiplexing cannot provide genotype labels to correlate
- Only cell hashing data is available with no SNP-based demultiplexing result — use HTOClassification cutoff calling instead
- SNP-based clustering quality is poor (e.g. plasmablast or dying-cell populations with abnormal expression, high true doublet rates), since hybrid performance is largely dependent on SNP clustering quality

## Inputs

- Seurat object with HTO assay and cell hashing singlet/doublet/negative labels (e.g. from HTOClassification with method='CLR')
- SNP-based genotype cluster labels per cell (e.g. scSplit ground-truth labels for the Stoeckius-2018 dataset)
- hto_names mapping and pair_info correlating hashing singlet groups with genotype clusters
- Convergence threshold c_threshold (default 0.7) and special-label strings (snp_doublet_label, snp_unassign_label, cellhashing_doublet_label, cellhashing_negative_label)

## Outputs

- hybrid_demultiplexed_labels.csv with one row per cell carrying a final sample identity plus singlet/doublet/negative designation
- Warning messages for hashing-group/genotype-cluster pairs whose convergence score falls below c_threshold

## How to apply

Load the hashing-labeled Seurat object (produced by HTOClassification/HTOIdAssign) together with genotype cluster labels (e.g. scSplit output), and call HybridDemultiplexing supplying cellhashing_label, genotype_label, hto_names, c_threshold, the special-label arguments (snp_doublet_label, snp_unassign_label, cellhashing_doublet_label, cellhashing_negative_label) and pair_info to correlate hashing singlet groups with genotype clusters. For each hashing-group/genotype-cluster pair compute the convergence score C = N1/(N1+N2) from concordant and discordant cell counts. Accept and merge pairs with C >= 0.7 (the c_threshold); for pairs failing the threshold (e.g. 9pool-CA scores of 0.1 and 0.55), fall back to cell hashing labels and issue a warning that results may be inaccurate. Then apply the three-case inconsistent-label assignment rules to cells whose hashing and genotype labels disagree, reclassifying snp_doublet, snp_unassign, cellhashing_doublet, and cellhashing_negative cells, and write final per-cell identities including singlet/doublet/negative designations to hybrid_demultiplexed_labels.csv. The hybrid approach is inherently self-validating because errors from one method can be identified by the other.

## Related tools

- **HTOreader** (Provides the HybridDemultiplexing function that computes convergence scores and applies the c_threshold decision and three-case inconsistent-label rules) — https://github.com/WilsonImmunologyLab/HTOreader
- **Seurat** (Hosts the Seurat object with the 'HTO' assay and hashing labels used as cellhashing_label input)
- **scSplit** (Source of the SNP-based genotype labels treated as genotype_label input in the benchmark workflow)

## Examples

```
devtools::install_github("WilsonImmunologyLab/HTOreader"); hybrid_labels <- HybridDemultiplexing(object = pbmc.hashtag, cellhashing_label = 'HTOid', genotype_label = 'scSplit_id', c_threshold = 0.7)
```

## Evaluation signals

- hybrid_demultiplexed_labels.csv exists with one row per cell (row_count_equals check)
- Every cell carries a final label produced by the 0.7 convergence-threshold decision or one of the three-case inconsistent-label rules (field_present check)
- For each hashing-group/genotype-cluster pair, the convergence score C = N1/(N1+N2) can be recomputed and pairs with C >= 0.7 are merged while pairs below threshold retain hashing labels
- Cells flagged snp_doublet, snp_unassign, cellhashing_doublet, or cellhashing_negative are reclassified under the three-case rules rather than left with raw labels
- A warning is emitted for low-convergence pairs (e.g. 9pool-CA scores of 0.1 and 0.55)

## Limitations

- The hybrid strategy's performance is largely dependent on the quality of SNP-based clustering
- SNP-based clustering can produce low-quality results due to abnormal gene expression profiles (e.g. plasmablasts or dying cells), high true doublet rates, and poor transcriptome quality
- SNP-based demultiplexing only works with samples that have a distinct genetic background, so it is not applicable in most animal studies or same-donor longitudinal studies
- Compatibility of the hybrid strategy with other barcode-based methods has not yet been examined

## Evidence

- [methods] HybridDemultiplexing function signature: "`HybridDemultiplexing(object, cellhashing_label, genotype_label, hto_names, c_threshold, snp_doublet_label, snp_unassign_label, cellhashing_doublet_label, cellhashing_negative_label, pair_info)`"
- [abstract] Hybrid approach correlating hashing and SNP clustering: "This approach correlates the results of cell hashing and genetic variant clustering, enabling precise and efficient cell identity determination without additional experimental costs or efforts."
- [intro] Self-validating nature of hybrid demultiplexing: "this hybrid approach is inherently self-validating; potential errors that arise from one method can be easily identified by the other, making hybrid demultiplexing more reliable than other unimodal"
- [discussion] SNP dependency limitation: "the performance of the hybrid strategy is largely dependent on the quality of SNP-based clustering"
- [discussion] SNP genetic background constraint: "SNP-based demultiplexing only works with samples that have a distinct genetic background. As a result, it is not applicable in most animal studies or longitudinal studies with samples from the same"
