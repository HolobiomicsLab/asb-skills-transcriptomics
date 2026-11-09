---
name: gene-regulatory-network-resource-handling
description: Use when a task needs a prior-knowledge GRN (DoRothEA or OmniPath) as
  input for TF-activity inference or perturbation benchmarking with decoupleR/decoupleRBench,
  and the network's sign and weight annotations must be preserved.
when_to_use_negative:
- The analysis must infer regulatory relationships de novo from expression data rather
  than relying on a curated prior-knowledge GRN.
- The chosen resource lacks signed/weighted TF–target edges while the selected decoupleR
  method requires mode of regulation and interaction weights.
- The task involves phospho-proteomic perturbation benchmarking (run_php_bench.R path)
  rather than transcriptomic GRN resource handling.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: decoupleR
  role: R/Bioconductor package whose decouple wrapper consumes the GRN resource to
    infer TF activities within a unified framework
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: decoupler
  role: Faster, memory-efficient Python implementation of the same enrichment methods
    for GRN-based scoring
  repo: https://github.com/saezlab/decoupler-py
- name: OmniPath
  role: Meta-resource of prior knowledge (100+ databases) leveraged by decoupleR as
    a network source
- name: decoupleRBench
  role: Benchmark package that pairs the GRN with perturbation count tables/metadata
    via decouple to evaluate method performance
  repo: https://github.com/saezlab/decoupleRBench
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbac016
    title: 'decoupleR: ensemble of computational methods to infer biological activities
      from omics data'
  source_content_hash: sha256:397910e43199334bd55b982a0e69c19c3f046ea63c8299a85877b5414cc63476
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OmniPath
  - R
  - decoupleR
  - decoupleRBench
  - decoupler
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# gene-regulatory-network-resource-handling

## Summary

Retrieves and prepares prior-knowledge gene regulatory networks (e.g. DoRothEA, OmniPath) with signed and weighted TF–target interactions so they can be flexibly paired with decoupleR inference methods to compute TF activity scores.

## When to use

Use when a task needs a prior-knowledge GRN (DoRothEA or OmniPath) as input for TF-activity inference or perturbation benchmarking with decoupleR/decoupleRBench, and the network's sign and weight annotations must be preserved.

## When NOT to use

- The analysis must infer regulatory relationships de novo from expression data rather than relying on a curated prior-knowledge GRN.
- The chosen resource lacks signed/weighted TF–target edges while the selected decoupleR method requires mode of regulation and interaction weights.
- The task involves phospho-proteomic perturbation benchmarking (run_php_bench.R path) rather than transcriptomic GRN resource handling.

## Inputs

- DoRothEA gene regulatory network (signed, weighted TF–target interactions)
- OmniPath meta-resource of prior-knowledge databases
- Perturbation experiment data from Zenodo record 5645208 (Holland et al. 2020)
- Input tibble with user-specified benchmark settings, count table, and metadata table

## Outputs

- Per-method TF activity scores for each regulator and experiment
- Benchmark performance table (AUROC/AUPRC, e.g. rna_bench_performance.csv matching Supplementary Table S3)

## How to apply

Retrieve the network and benchmark data via the manuscript repository's data script, which pulls the Holland et al. 2020 perturbation collection from Zenodo record 5645208; the OmniPath meta-resource (over 100 databases) can also be leveraged directly through decoupleR. Keep the GRN as a signed, weighted TF–target resource because decoupleR incorporates methods that leverage mode of regulation and interaction weights, which other frameworks lack. The benchmark pipeline then needs an input tibble with user-specified settings plus a count table and corresponding metadata table, and methods are applied through the decouple wrapper function. Because decoupleR allows flexibly testing any method with any resource, the same network can be reused across the 11 methods (AUCell, fast GSEA, GSVA, ORA, ULM, VIPER, mlm, consensus) without reformatting per method.

## Related tools

- **decoupleR** (R/Bioconductor package whose decouple wrapper consumes the GRN resource to infer TF activities within a unified framework) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **decoupler** (Faster, memory-efficient Python implementation of the same enrichment methods for GRN-based scoring) — https://github.com/saezlab/decoupler-py
- **OmniPath** (Meta-resource of prior knowledge (100+ databases) leveraged by decoupleR as a network source)
- **decoupleRBench** (Benchmark package that pairs the GRN with perturbation count tables/metadata via decouple to evaluate method performance) — https://github.com/saezlab/decoupleRBench

## Examples

```
devtools::install_github('saezlab/decoupleRBench'); Rscript R/process/get_data.R
```

## Evaluation signals

- The network retains mode of regulation (sign) and interaction weights, enabling sign-aware methods such as ULM, VIPER and mlm to run.
- The retrieved benchmark data matches the Holland et al. 2020 transcriptomics perturbation collection from Zenodo record 5645208.
- Per-method TF activity scores are produced for all regulators and experiments covered by the GRN (regulators outside the network cannot be scored).
- Downsampled AUROC/AUPRC evaluation completes, with negative classes randomly sampled to balance limited true positive regulators.
- Aggregated ranking reproduces the expected top-three methods: consensus, multivariate linear model and ULM (Supplementary Table S3).

## Limitations

- True positive classes are limited to regulators covered by the perturbation experiments, forcing a downsampling strategy for AUROC/AUPRC calculation.
- Only TF–target relationships present in the prior-knowledge resource can be scored; uncovered perturbed TFs are invisible to the benchmark.
- Some reproduction steps are slow: the RNA noise analysis takes ~6 hours and the PHP noise analysis ~2 hours per the manuscript repository.

## Evidence

- [abstract] OmniPath meta-resource leverage: "it leverages OmniPath, a meta-resource comprising over 100 databases of prior knowledge"
- [abstract] signed/weighted methods: "including methods that leverage mode of regulation and weights of interactions, which are not present in other frameworks"
- [readme] flexible method-resource pairing: "decoupleR allows the user to flexibly test any method with any resource."
- [readme] benchmark input requirements: [evidence span withheld — anchor status: external_doc]
- [readme] downsampling rationale: [evidence span withheld — anchor status: external_doc]
