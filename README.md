## Meow: End-to-End Outline Writing for Automatic Academic Survey

[中文版 README](./README_CN.md)

Meow is an end-to-end, metadata-driven model for **automatic academic survey outline generation**.
Given a target topic and a set of candidate papers (title + abstract), Meow directly generates hierarchical survey outlines that balance structural coherence, content relevance, and academic writing style, providing high-quality foundations for subsequent automatic writing or human authoring.

---

### Highlights

- **End-to-End Outline Generation**
  From "topic + metadata" directly to "complete hierarchical outline" without relying on multi-agent or multi-stage complex pipelines, making it more suitable for long-context and batch automation scenarios.

- **Large-Scale, High-Quality Survey Corpus** [\[dataset\]](https://huggingface.co/datasets/haajimi/Meow)
  - 2,828,998 arXiv paper metadata
  - 252,357 bioRxiv and 72,059 medRxiv papers
  - Systematically filtered high-quality surveys with extracted human-written outlines and completed reference information (including abstracts retrieved via semantic search).

- **Two-Stage Training: CoT + RL (GRPO)**
  - Uses DeepSeek-R1 for **Chain-of-Thought distillation**, learning explicit reasoning processes from literature to taxonomy/outline.
  - Followed by **Group Relative Policy Optimization (GRPO)** reinforcement learning to optimize structural similarity and format compliance rewards.

- **Structure-Aware Reward Design**
  - Uses **Tree Edit Distance (TED)** as structural distance metric to measure hierarchical similarity between generated and human-written outlines.
  - Combines structural similarity reward with format compliance reward to explicitly encourage structures that "resemble real surveys".

---

### Model Illustration

<p align="center">
  <img src="./figure.png" alt="Meow 框架图" width="450" />
</p>

### Method Overview

- **Task Definition**
  Given a topic $\mathcal{T}$ and a paper collection $\mathcal{P} = \{p_i\}$, where each $p_i = (t_i, a_i)$ consists of title + abstract, the goal is to generate a hierarchical outline $\mathcal{O} = \{s_j\}_{j=1}^M$, where each section $s_j$ corresponds to a heading $h_j$, requiring:
  - Clear structure with reasonable hierarchy
  - Distinct content boundaries without redundant overlap
  - Good citation coverage and logical organization of input literature

- **Data Curation**
  - Keyword + structural rules to filter survey papers
  - Clean and standardize section structures, removing non-core parts like acknowledgments and appendices
  - Vectorize reference titles using `all-MiniLM-L6-v2`, retrieve matching abstracts from large-scale metadata to complete citations
  - Use DeepSeek-R1 to derive taxonomy and CoT reasoning chains from references for supervised training

- **Training Pipeline**
  - **Stage 1: CoT Supervised Fine-Tuning (SFT)**
    Supervised fine-tuning on CoT-augmented data to learn explicit reasoning + generation process from metadata to outline.
  - **Stage 2: GRPO Reinforcement Learning**
    - Sample multiple candidate outlines for each input
    - Combine structural similarity + format compliance into total reward $R_{\text{total}}$
    - Use GRPO for policy optimization with KL constraint to maintain stability with the reference policy.

---

### Results (LLM-as-a-Judge & Structural Distance)

We compare various mainstream LLMs and academic baseline models on our self-constructed test set and the SurveyX test set. Evaluation uses LLM-as-a-Judge (five dimensions) and Structural Distance metrics, with scores corresponding exactly to Table 1 and Table 2 in the paper.

#### Our Test Set (Ours-100)

| **Model**              | **Structure Locate ↑** | **Structure Detail ↑** | **Content Exclusion ↑** | **Content Depth ↑** | **Pragmatics Concise ↑** | **Total ↑** | **Structural Distance ↓** |
|------------------------|------------------------|-------------------------|--------------------------|---------------------|---------------------------|-------------|---------------------------|
| Human-written          | 7.80                   | 7.21                    | 7.93                     | 6.00                | 8.29                      | 37.23       | 0.00                      |
| DeepSeek-R1            | 8.09                   | 5.31                    | 8.15                     | 5.75                | 8.85                      | 36.15       | 0.48                      |
| DeepSeek-V3            | 8.26                   | 4.17                    | 8.04                     | 5.46                | 8.95                      | 34.88       | 0.46                      |
| GPT-5 Nano             | 6.99                   | 5.66                    | 6.19                     | 5.04                | 7.84                      | 31.72       | 0.46                      |
| Gemini 2.5 Flash-Lite  | 8.01                   | 5.68                    | 8.18                     | 5.84                | 8.72                      | 36.43       | 0.52                      |
| Qwen3-8B               | 4.97                   | 3.71                    | 4.47                     | 3.65                | 4.62                      | 21.42       | 0.59                      |
| **Meow-8B-SFT**        | 7.79                   | 6.75                    | 7.11                     | 5.34                | 8.11                      | 35.10       | 0.43                      |
| **Meow-8B-SFT-GRPO**   | 8.01                   | 6.46                    | 7.98                     | 5.73                | 8.61                      | **36.79**   | **0.39**                  |

#### SurveyX Test Set

| **Model**            | **Structure Locate ↑** | **Structure Detail ↑** | **Content Exclusion ↑** | **Content Depth ↑** | **Pragmatics Concise ↑** | **Total ↑** | **Structural Distance ↓** |
|----------------------|------------------------|-------------------------|--------------------------|---------------------|---------------------------|-------------|---------------------------|
| Human-written        | 8.11                   | 7.39                    | 8.14                     | 5.21                | 8.07                      | 36.92       | 0.00                      |
| SurveyX              | 7.37                   | 5.30                    | 4.60                     | 3.90                | 8.43                      | 29.60       | 0.44                      |
| **Meow-8B-SFT**      | 6.56                   | 5.83                    | 7.33                     | 5.28                | 7.83                      | 32.83       | 0.53                      |
| **Meow-8B-SFT-GRPO** | 7.71                   | 6.12                    | 7.71                     | 5.46                | 8.62                      | **35.62**   | **0.42**                  |

> The results show that **Meow-8B-SFT-GRPO** achieves total scores close to human-written outlines on both test sets, along with the lowest or near-lowest Structural Distance among all models, demonstrating that the reinforcement learning stage significantly improves outline structural fidelity and overall writing quality.

---

### Citation

If this work or repository is helpful to your research, please cite:

```bibtex
@article{ma2025meow,
  author={Ma, Zhaoyu and Shan, Yuan and Zhao, Jiahao and Xu, Nan and Wang, Lei},
  booktitle={ICASSP 2026 - 2026 IEEE International Conference on Acoustics, Speech and Signal Processing (ICASSP)}, 
  title={Meow: End-to-End Outline Writing for Automatic Academic Survey}, 
  year={2026},
  pages={20561-20565},
}
```
