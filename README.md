# iris-path-prioritization
Cluster-Guided Path Prioritization for Efficient LLM-Assisted Static Vulnerability Detection
# Cluster-Guided Path Prioritization for Efficient LLM-Assisted Static Vulnerability Detection

**Authors:** Soniya Pranav Apte, Ali Zia Khan, Zubair Malik  
**Affiliation:** North Dakota State University  
**Status:** Working paper (2026)

## Overview

This project extends [IRIS](https://openreview.net/forum?id=wSaCSMnP4R) (ICLR 2025), an LLM-assisted static analysis tool that uses large language models to infer missing source/sink specifications for CodeQL taint analysis. While IRIS improves CodeQL's vulnerability detection — particularly for CWE-94 (Code Injection) where CodeQL finds zero alerts — it generates thousands of candidate taint paths, most of which are false positives.

We propose a **cluster-guided path prioritization layer** that is inserted between CodeQL path generation and LLM inspection. The layer scores, clusters, and ranks candidate paths so that only the highest-risk groups proceed to expensive LLM analysis.

## Key Results

- **IRIS Reproduction:** Detected 9/16 vulnerabilities across CWE-94, CWE-79, and CWE-22 where CodeQL found zero or few alerts
- **Path Prioritization:** Up to 28.8x precision improvement (Struts: 51.8% vs 1.8% baseline) with 98% LLM-call reduction
- **Fine-Tuning:** DeepSeekCoder 7B accuracy improved from 53% to 82% for API labeling via LoRA

## Method

The prioritization method consists of five steps:

1. **Feature Extraction** — Parse source/sink URIs, path length, file count, CWE type from SARIF output
2. **CWE-Aware Scoring** — Score paths using vulnerability-specific heuristic patterns
3. **Path Clustering** — Group similar paths by signature: `source_file | sink_file | length_bucket | CWE`
4. **Convergence Ranking** — Rank clusters by `ClusterScore(C) = max Score(p) + Convergence(C)`
5. **Top-K Selection** — Forward only top-K clusters for LLM inspection

## Results Summary

| Project | Paths | Clusters | P@10 | R@10 | MRR | LLM Saved |
|---------|-------|----------|------|------|-----|-----------|
| Struts | 5,546 | 1,647 | 51.8% | 43.6% | 1.000 | 98% |
| Cron-utils | 18 | 11 | 43.8% | 77.8% | 0.250 | 0% |
| ESAPI | 610 | 223 | 47.4% | 8.2% | 0.200 | 91% |
| ff4j | 1,083 | 341 | 23.5% | 2.5% | 0.500 | 97% |
| Antisamy | 129 | 26 | 16.4% | 47.6% | 0.333 | 24% |
| Commons-text | 201 | 63 | 32.0% | 6.7% | 0.250 | 80% |

## Repository Structure

```
iris-path-prioritization/
├── README.md
├── docs/
│   ├── pipeline.png              # Pipeline diagram
│   └── paper.pdf                 # Working paper (latest draft)
├── src/
│   ├── path_prioritizer.py       # Path prioritization implementation
│   ├── eval_all_metrics.py       # Evaluation with all metrics
│   ├── ablation_study.py         # Ablation study
│   ├── finetune_model.py         # LoRA fine-tuning script
│   ├── eval_finetune.py          # Fine-tuned model evaluation
│   ├── eval_base.py              # Base model evaluation
│   └── collect_results.py        # Results collection
├── data/
│   └── finetune_data/            # Training/validation/test splits
└── results/
    └── sarif/                    # IRIS output (SARIF files)
```

## Reproducing IRIS

We identified and documented **5 compatibility fixes** required to run the published IRIS code:

| # | Issue | Fix |
|---|-------|-----|
| 1 | Pack version wildcard | Pinned to v0.8.3, used --additional-packs |
| 2 | ActiveThreatModelSource absent | Replaced with RemoteFlowSource |
| 3 | Query version mismatch | Updated config.py |
| 4 | Temperature=0.0 rejected | Changed to 0.01 with do_sample=True |
| 5 | --search-path ignored | Switched to --additional-packs |

## Environment

- **Hardware:** AMD EPYC 7532 32-core, 256 GB RAM, NVIDIA A100 GPU
- **Software:** CodeQL 2.15.3, Java 8/11, Maven 3.6.3, Python 3.9
- **LLM:** DeepSeekCoder 7B Instruct (local inference, zero API cost)
- **Dataset:** [CWE-Bench-Java](https://github.com/iris-sast/cwe-bench-java) (16 projects)

## Acknowledgment

This work used resources of the Center for Computationally Assisted Science and Technology (CCAST) at North Dakota State University, which were made possible in part by NSF MRI Award No. 2019077.

## Paper

[Working Paper (PDF)](docs/paper.docx)
