---
name: cluster-pruning-merging-expansion
description: Use when propagating seed labels over a directed kNN graph and the resulting
  clusters need pruning of tiny clusters, merging of similar clusters, and assignment
  of remaining unlabeled cells without a predetermined cluster count.
when_to_use_negative:
- Cell labels are already final or the input is a pre-clustered/annotated feature
  table rather than a post-propagation labeling state
- A predetermined number of cell types must be enforced — scQA's label propagation
  and expansion specifically 'obviat[es] the requirement for a predetermined number
  of cell types'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scQA
  role: Implements the CC module's pruning, merging, and cluster-expansion steps after
    seed-based label propagation, emitting labels.txt
  repo: https://github.com/LD-Lyndee/scQA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:824f06ca66b7b07d392466948bd9e20fcb28c14ac94d67ebef163f7a418c2ee9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LD-Lyndee/scQA
  tools:
  - scQA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cluster-pruning-merging-expansion

## Summary

The post-label-propagation refinement stage of the scQA Cluster Constructor (CC) module: small clusters are pruned, similar clusters merged under a misclassification guard, and unlabeled nodes are relabeled or seeded as new clusters, iterated until convergence before writing labels.txt.

## When to use

Use when propagating seed labels over a directed kNN graph and the resulting clusters need pruning of tiny clusters, merging of similar clusters, and assignment of remaining unlabeled cells without a predetermined cluster count.

## When NOT to use

- Cell labels are already final or the input is a pre-clustered/annotated feature table rather than a post-propagation labeling state
- A predetermined number of cell types must be enforced — scQA's label propagation and expansion specifically 'obviat[es] the requirement for a predetermined number of cell types'

## Inputs

- Partially labeled cell set after seed-based label propagation (labels assigned to seed-propagated clusters, remaining nodes unlabeled)
- Directed k-nearest-neighbor graph over cells from the quantitative cell-landmark matrix Q2 (Pearson correlation similarity)
- Qualitative cell-landmark matrix Q1 from the LC1 module

## Outputs

- Refined, converged set of cell clusters (per-cell cluster labels)
- labels.txt file containing the final clustering result

## How to apply

After scQA label propagation (combinatorial coefficient alpha=0.5 over the directed kNN graph built from the LC2 matrix Q2 with Pearson correlation similarity), first prune clusters that are too small, mirroring how Cluster8 with only three cells was later flagged as outlier-like. Merge cluster Ci into a larger CJ whenever S(Ci,CJ) exceeds the number of edges within Ci, and never merge a cluster that has already assimilated another, preventing misclassification chains. Then expand: relabel unlabeled node ni into cluster Cj when S(ni,Cj)=|ONiCj|-A(Cj) is positive; create a new cluster (with a fresh label) for nodes scoring negative against every existing cluster. Re-run label propagation after expansion and repeat until convergence. Finally write per-cell cluster labels to labels.txt (run scQA as './main xin_matrix.txt o', where 'o' selects the default binning=6).

## Related tools

- **scQA** (Implements the CC module's pruning, merging, and cluster-expansion steps after seed-based label propagation, emitting labels.txt) — https://github.com/LD-Lyndee/scQA

## Examples

```
git clone https://github.com/LD-Lyndee/scQA.git; cd scQA; ./make; ./main xin_matrix.txt o
```

## Evaluation signals

- No surviving cluster is suspiciously tiny after pruning — clusters reduced to a handful of cells (e.g., 3 cells) should be treated as outliers, as with Cluster8 in the Xenopus lung dataset
- A cluster that has assimilated another is never itself a merge target (misclassification-prevention invariant: S(Ci,CJ) must exceed the number of edges within Ci)
- After convergence, every cell in labels.txt is labeled: unlabeled nodes were either absorbed via positive S(ni,Cj)=|ONiCj|-A(Cj) or seeded as new clusters
- Final cluster count was not fixed in advance — it emerged from propagation, pruning, and expansion on the 20 benchmark datasets (e.g., 13 clusters expected for Baronm GSE84133)

## Limitations

- Genes may be misclassified in LC1 and LC2 due to high dimensionality, propagating errors into the clusters being pruned and merged
- Retaining only 2000 highly variable genes excluded some informative genes, which can degrade cluster quality before refinement
- Some clusters may remain biologically mixed after merging — e.g., endothelial and mesenchymal cells were incorrectly grouped into a single Cluster3
- Clusters lacking marker genes and containing only a few cells (like Cluster8) may indicate outlier cells rather than real types

## Evidence

- [methods] After the removal, we prioritize merging the smaller clusters, ensuring that similar ones are combined into a single entity.: "After the removal, we prioritize merging the smaller clusters, ensuring that similar ones are combined into a single entity."
- [methods] All unlabeled vertices are either added to existing clusters or isolated to create new ones in the cluster expansion step.: "All unlabeled vertices are either added to existing clusters or isolated to create new ones in the cluster expansion step."
- [abstract] It then clusters cells by employing an ingenious label propagation strategy, obviating the requirement for a predetermined number of cell types.: "It then clusters cells by employing an ingenious label propagation strategy, obviating the requirement for a predetermined number of cell types."
- [results] Cluster8 contained only three cells, likely being outliers during clustering: "Cluster8 contained only three cells, likely being outliers during clustering"
- [readme] The input file must be in the form of a matrix of cells * genes, without names of rows and columns.: "The input file must be in the form of a matrix of cells * genes, without names of rows and columns."
