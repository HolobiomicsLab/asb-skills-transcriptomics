---
name: case-control-phenotype-simulation
description: Use when a benchmark with known ground-truth genetic meta-markers is
  needed to test pattern-mining tools like HOGImine, e.g. for the simulation study
  of Section 4.1.1.
when_to_use_negative:
- The task is association discovery on real data (e.g. data/athaliana or data/mmusculus)
  — this skill only produces synthetic benchmark inputs
- The evaluation target is single-SNP or single-interval methods (e.g. FastCMH) rather
  than connected higher-order gene meta-markers — a network-subgraph causal module
  does not match that ground truth
- Samples are already genotyped and only phenotype labels need simulation without
  a planted network-structured signal
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: HOGImine
  role: Consumes the generated synthetic files (-i, -l, -s, -c, -m, -e); the generator's
    output format is defined by HOGImine's input specification
  repo: https://github.com/BorgwardtLab/HOGImine
- name: HOGImine (hogimine_additive)
  role: Smoke-run binary used to validate the synthetic marker/label/interaction files
    execute without error at -f 0.05
  repo: https://github.com/BorgwardtLab/HOGImine
- name: networkx
  role: Generates the Erdos-Renyi G(75,100) prior interaction graph via gnm_random_graph
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad273
    title: Higher-order genetic interaction discovery with network-based biological
      priors
  source_content_hash: sha256:7ad6eb4bb875ee7f282e95df09a3996e2cd0c3dfb0b1f4b56df2665a2bb4b3ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BorgwardtLab/HOGImine
  tools:
  - HOGImine
  - HOGImine (hogimine_additive)
  - networkx
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# case-control-phenotype-simulation

## Summary

Generates synthetic case-control GWAS datasets with a planted, ground-truth higher-order association: a random connected subgraph of a gene-interaction network whose spanning SNP intervals carry tunable phenotype association (strength rho), serialized into the HOGImine input format (-i, -l, -s, -c, -m, -e plus a ground-truth file).

## When to use

Use when a benchmark with known ground-truth genetic meta-markers is needed to test pattern-mining tools like HOGImine, e.g. for the simulation study of Section 4.1.1.

## When NOT to use

- The task is association discovery on real data (e.g. data/athaliana or data/mmusculus) — this skill only produces synthetic benchmark inputs
- The evaluation target is single-SNP or single-interval methods (e.g. FastCMH) rather than connected higher-order gene meta-markers — a network-subgraph causal module does not match that ground truth
- Samples are already genotyped and only phenotype labels need simulation without a planted network-structured signal

## Inputs

- Random seed / RNG specification (for reproducibility of the ER graph and causal subgraph)
- Number of samples (3000), SNP counts per gene (U[3,10]), covariate cardinality C, and association strength rho
- Choice of encoding: binary (dominant/recessive per SNP) and/or additive

## Outputs

- Marker matrix file (-i): n.SNPs rows x 3000 sample columns in binary or additive encoding
- Binary label file (-l), n.samples x 1
- SNP names file (-s)
- Covariate file (-c) with integer classes in {1,...,C}
- SNP map file (-m) linking gene names to their SNPs
- Gene interaction sets file (-e) from the ER(75,100) graph
- Ground-truth file listing genes, encodings, and SNP positions of the planted meta-marker

## How to apply

Sample an Erdos-Renyi G(n,m) random graph with 75 nodes and exactly 100 edges (e.g. via networkx gnm_random_graph) as the gene-interaction prior for the -e file. Assign each gene a number of SNPs drawn uniformly from U[3,10] and simulate genotypes for 3000 samples in binary and/or additive encoding (with per-SNP dominant/recessive annotation), plus a binary phenotype label column and an integer covariate column in {1,...,C}. Sample a random connected subgraph as the causal module and inject phenotype association with tunable strength rho into the per-gene SNP intervals it spans. Serialize all files in HOGImine format: marker matrix (-i, n.SNPs x 3000 sample columns), binary labels (-l), SNP names (-s), covariates (-c), SNP map (-m: gene name followed by its mapped SNPs), gene interaction sets (-e), and a ground-truth file recording the genes, encodings, and SNP positions of the planted meta-marker.

## Related tools

- **HOGImine** (Consumes the generated synthetic files (-i, -l, -s, -c, -m, -e); the generator's output format is defined by HOGImine's input specification) — https://github.com/BorgwardtLab/HOGImine
- **HOGImine (hogimine_additive)** (Smoke-run binary used to validate the synthetic marker/label/interaction files execute without error at -f 0.05) — https://github.com/BorgwardtLab/HOGImine
- **networkx** (Generates the Erdos-Renyi G(75,100) prior interaction graph via gnm_random_graph)

## Examples

```
import networkx as nx; G = nx.gnm_random_graph(75, 100, seed=0)  # -e interactions prior; then validate: ./src/hogimine_additive -i synth_X.txt -l synth_y.txt -e synth_interactions.txt -m synth_map.txt -f 0.05
```

## Evaluation signals

- All six generated files parse as valid HOGImine inputs
- Marker matrix has exactly 3000 sample columns with values in binary or additive encoding
- Labels file is binary with shape n.samples x 1
- The -e file has exactly 100 edges over 75 gene nodes (matching G(n,m))
- Smoke run of hogimine_additive on the synthetic data with -f 0.05 executes without error
- Ground-truth file's planted subgraph is connected in the -e network and its SNP intervals match the -m map

## Limitations

- Only the FastCMH R helper makefastcmhdata supports K=2 covariate classes, while HOGImine synthetic covariates span {1,...,C} — covariate sampling logic must be custom
- The planted signal is limited to a single random connected subgraph, so ground truth is one meta-marker per dataset
- README has no changelog; format details for covariates and encodings must be cross-checked against the tool's input spec rather than documentation

## Evidence

- [other] planted connected causal subgraph: [evidence span withheld — anchor status: not_found]
- [readme] encoding options: "HOGImine accepts both binary encodings and additive encodings for the markers."
- [intro] meta-marker class: "it expands the class of discoverable genetic meta-markers by considering higher-order interactions of genes"
- [abstract] prior network usage: "Our method can exploit prior biological knowledge on gene interactions, such as protein-protein interaction networks, genetic pathways, and protein complexes, to restrict its search space."
