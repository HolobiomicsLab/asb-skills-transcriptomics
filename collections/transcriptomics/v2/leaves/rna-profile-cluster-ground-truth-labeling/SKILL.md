---
name: rna-profile-cluster-ground-truth-labeling
description: Use when benchmarking HTO-based demultiplexing methods on a cell hashing
  dataset where HTO counts alone cannot be trusted as truth, and RNA profiles are
  available to establish droplet identities independently.
when_to_use_negative:
- The RNA library is missing or too shallow to cluster droplets — no independent reference
  exists for labeling.
- You need absolute, experimentally verified sample identities (e.g. from FACS sorting),
  not cluster-inferred approximations.
- The dataset has no cell hashing / HTO channel and you are not evaluating demultiplexing
  methods at all.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3934
tools:
- name: demuxmix
  role: R/Bioconductor package whose benchmark evaluation consumes the RNA-cluster
    ground-truth labels to compute precisionSSD and sensitivitySSD
  repo: https://github.com/huklein/demuxmix
- name: R
  role: Implementation environment for the Bioconductor demuxmix package and the benchmarking
    scripts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad481
    title: 'demuxmix: demultiplexing oligonucleotide-barcoded single-cell RNA sequencing
      data with regression mixture models'
  source_content_hash: sha256:baafbf2aa9e301d7972f2617098042fd52ace237171b098a6dfef26293c22f01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huklein/demuxmix
  tools:
  - R
  - demuxmix
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3934
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# rna-profile-cluster-ground-truth-labeling

## Summary

Derives sample-of-origin ground-truth labels for each droplet from clustering of RNA expression profiles, providing an RNA-independent reference against which HTO-based demultiplexing methods are benchmarked.

## When to use

Use when benchmarking HTO-based demultiplexing methods on a cell hashing dataset where HTO counts alone cannot be trusted as truth, and RNA profiles are available to establish droplet identities independently.

## When NOT to use

- The RNA library is missing or too shallow to cluster droplets — no independent reference exists for labeling.
- You need absolute, experimentally verified sample identities (e.g. from FACS sorting), not cluster-inferred approximations.
- The dataset has no cell hashing / HTO channel and you are not evaluating demultiplexing methods at all.

## Inputs

- RNA count matrix (genes x droplets) for the pooled cell hashing experiment (e.g. GSM3501447 from GEO)
- HTO count matrix (HTOs x droplets) (e.g. GSM3501446 from GEO)
- Attenuation factor s applied to HTO counts (s = 1 down to 0.25)
- Simulated doublet set (pairs of artificially combined droplets)

## Outputs

- Per-droplet ground-truth sample-of-origin labels derived from RNA-profile clusters
- Ground-truth singlet/doublet class assignments for all droplets including the 350 simulated doublets
- Confusion-ready labels supporting precisionSSD and sensitivitySSD computation per demultiplexing method

## How to apply

Cluster the droplets using their RNA count matrix (genes x droplets) rather than the HTO counts, so that the resulting cluster labels reflect transcriptional identity rather than the hashtag oligonucleotide signal under evaluation. Assign each droplet a sample-of-origin label based on its RNA cluster membership, treating cells from the same biological sample as one ground-truth class and label the artificially generated doublets (pairs of combined droplets) as doublets. In the demuxmix benchmark, this RNA-cluster labeling was applied to a cell line mixture experiment (GSM3501446/GSM3501447) with HTO signal attenuated from s = 1 down to s = 0.25 and 350 simulated doublets, so that demultiplexing performance could be scored at each attenuation level. The rationale is that RNA profiles are independent of the HTO sequencing channel being degraded, giving a stable reference even as the HTO signal weakens. Use these labels to compute per-method average precision for singlet/doublet classification (precisionSSD) and sensitivity (sensitivitySSD), tabulated per method and attenuation factor.

## Related tools

- **demuxmix** (R/Bioconductor package whose benchmark evaluation consumes the RNA-cluster ground-truth labels to compute precisionSSD and sensitivitySSD) — https://github.com/huklein/demuxmix
- **R** (Implementation environment for the Bioconductor demuxmix package and the benchmarking scripts)

## Examples

```
Rscript -e 'library(demuxmix); hto <- readRDS("GSM3501446_hto.rds"); rna.genes <- colSums(readRDS("GSM3501447_rna.rds") > 0); res <- demuxmix(hto, numGenes = rna.genes); dmmClass(res)'
```

## Evaluation signals

- Spot-check that droplets assigned to the same RNA cluster receive consistent sample-of-origin labels.
- Verify that every artificially combined doublet pair is labeled as a doublet and all uncombined droplets as singlets in the ground truth.
- Benchmark sanity: demuxmix should attain approximately 85% average precisionSSD while competing methods fall in the 67%-81% range against these labels.
- Confirm labels are derived from RNA clustering, not from the attenuated HTO counts (s down to 0.25), so the reference does not degrade with the HTO signal.
- Check that the total labeled droplet count equals original droplets plus the 350 simulated doublets.

## Limitations

- RNA-cluster labels are an inferred, not experimentally verified, ground truth — cluster errors propagate into the benchmark.
- The reference depends on sufficient RNA depth; it was validated on a cell line mixture dataset, which may not generalize to more heterogeneous samples.
- Ground-truth labels only remain valid for the fixed attenuation scenario they were derived under (s = 1 to 0.25) plus the specific 350 simulated doublets.

## Evidence

- [results] Simulated cell line mixture benchmark metrics with RNA cluster labels as ground truth: [evidence span withheld — anchor status: not_found]
- [intro] HTO and RNA matrices as paired inputs: "sequenced together with the RNA molecules of the cells resulting in an (HTOs x droplets) count matrix in addition to the"
- [readme] Package purpose in README: "demuxmix is a package for demultiplexing single-cell sequencing experiments of pooled cells labeled with barcode oligonucleotides"
- [intro] Classification includes multiplet detection: "The package implements methods to fit regression mixture models for a probabilistic classification of cells, including multiplet detection"
