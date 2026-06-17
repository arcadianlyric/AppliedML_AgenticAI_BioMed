# Applied ML & Agentic AI in Biomedical Genomics

**Portfolio for Biotech Machine Learning Engineer roles**

End-to-end ML engineering across production genomics pipelines, foundation model fine-tuning, agentic AI orchestration, and real-time data drift monitoring — all applied to sequencing data at scale.

---

## Enterprise Agentic AI Pain Points

Seven pain points commonly block enterprise agentic AI deployments. Each project below targets one or more directly.

| # | Pain Point | Root Cause |
|---|---|---|
| P1 | **Multi-step error compounding** | 95% per-step accuracy → 60% over 10 steps |
| P2 | **Tool-use unreliability** | Hallucinated parameters, silent failures, wrong call order |
| P3 | **Eval gap** | No standard metric for agent performance, hallucination rate, tool accuracy |
| P4 | **Observability blindness** | Can't trace which step in a loop caused the failure |
| P5 | **Context degradation** | Retrieval quality and constraint memory decay in long sessions |
| P6 | **Human-in-loop design** | No principled stopping criterion — agent either over-asks or goes rogue |
| P7 | **Production drift** | ML tools silently degrade when input distribution shifts |

---

## Projects

### 1. [PhasedVariants AgenticCurator](https://github.com/arcadianlyric/PhasedVariants_AgenticCurator)
> Multi-agent LLM system with RAG and knowledge graph integration that automates interpretation of phased WGS variants — linking haplotype-resolved genotypes to gene function, disease mechanisms, and clinical significance.

- **Stack**: LangChain · FAISS · DeepSeek · Grok (xAI) · PrimeKG · VEP · PubMed / GeneCards / arXiv / Tavily · sentence-transformers · ClinVar
- **Highlights**:
  - **3-stage pipeline**: VEP-annotated phased VCF → interactive gene-disease-pathway network (PrimeKG) → multi-source literature retrieval (progressive search across 4 APIs) → agentic curation with quality-controlled output
  - **Dual-agent review loop**: Output Agent (DeepSeek, RAG + KG context) generates analysis; Review Agent (Grok) independently scores 5 dimensions (0–10) and flags hallucinations; loop iterates until quality threshold (7.0/10) is met. Different LLMs used by design to eliminate shared blind spots
  - **Hallucination reduction**: Cross-model review + FAISS-grounded retrieval; benchmarked against single-agent baselines (`llm_queryAlone`, `llm_augmented`, `llm_rag`, v1 7-agent legacy)
  - Directly consumes phased VCF output from Project 1 (DNBSEQ_Complete_WGS)
- **Pain points addressed**: **P1 Multi-step error** — 3-iteration stop/revise loop with explicit convergence criterion prevents unchecked error propagation; **P3 Eval gap** — 5-dimension scoring rubric (accuracy, completeness, clinical relevance, hallucination rate, tool-use accuracy) is a deployable evaluation framework for any agentic output; **P5 Context degradation** — FAISS chunk-alignment fix ensures retrieval context matches LLM window; **P6 Human-in-loop** — quality gate (7.0/10) and stop/revise logic encode principled stopping criterion; **P2 Tool-use reliability** — cross-model adversarial review flags tool-call errors that a single LLM would miss.

---

### 2. [LFR Data Monitor — Sequencing QC & Drift Detection](https://github.com/arcadianlyric/LFR_DataMonitor)
> ML-powered QC monitor for MGI stLFR/cLFR co-barcoding runs; detects data drift that degrades downstream variant-calling model performance.

- **Stack**: Snakemake · BWA · DeepVariant · LightGBM · Autoencoders · PCA · Isolation Forest
- **Highlights**: Per-run feature matrices from alignment + variant metrics; unsupervised drift detection; automated fine-tuning trigger when distribution shift exceeds threshold
- **Pain points addressed**: **P4 Observability** — real-time monitoring of ML tool output quality with alerting; **P7 Production drift** — unsupervised detection of input distribution shift (insert-size, GC bias, UMI quality) before it silently degrades model accuracy; **P2 Tool-use reliability** — automated trigger prevents agents from calling degraded models.

---

### 3. [Google DeepVariant Fine-Tuning](https://github.com/arcadianlyric/GoogleDeepVariant_FineTuning)
> Adapts pretrained DeepVariant (Inception-v3 CNN) to shifted sequencing distributions — reducing detection-to-retraining cycle from days to hours.

- **Stack**: TensorFlow/Keras · TFRecord · hap.py · GIAB truth sets · Google Colab
- **Highlights**: Transfer learning on pileup image tensors; benchmarked against GIAB HG001/HG002; integrates upstream drift signals from LFR_DataMonitor for automated adaptation without training from scratch
- **Pain points addressed**: **P7 Production drift** — closes the loop on drift detection by automated model adaptation; demonstrates the MLOps pattern (detect → retrain → validate → deploy) that enterprise agentic systems need for any ML tool in their stack.

---

### 4. [Agentic bioArchitect — Multi-Agent Pipeline Design](https://github.com/arcadianlyric/Agentic_bioArchitect)
> Multi-agent LLM system that designs and implements bioinformatics pipelines for 16S rRNA metatranscriptomics with UMI support, eliminating PCR amplification bias.

- **Stack**: CrewAI · OpenAI / DeepSeek / Gemini / Grok · Tavily · PubMed API · MEGAHIT · DADA2 · Snakemake
- **Highlights**: Phase 1 — research + reviewer agents produce validated workflow designs; Phase 2 — coder agents generate production Python/Snakemake; benchmarked against ZymoBIOMICS community standards
- **Pain points addressed**: **P1 Multi-step error compounding** — Researcher → Analyst → Reviewer chain with reflection loop prevents error propagation; **P6 Human-in-loop design** — quality gate (score ≥ 7/10) controls when to proceed vs. iterate; **P2 Tool-use reliability** — structured tool wrappers (Tavily, PubMed) with validated output schemas before passing to downstream agents.

---

### 5. [ZeroShot Immune Feature Drift](https://github.com/arcadianlyric/ZeroShot_ImmuneFeatureDrift)
> Zero-shot foundation model monitoring of immune aging (immunosenescence) across longitudinal bulk RNA-seq without fine-tuning on small datasets.

- **Stack**: scGPT-blood (12-layer Transformer, 10.3 M cells pre-training) · CIBERSORTx · Isolation Forest · PCA · stLFR sequencing
- **Highlights**: Tracks immune shifts across 3 PBMC timepoints (2024–2026); identifies isoform switches in aging markers (PTPRC/CD45); cosine + Euclidean embedding drift metrics
- **Pain points addressed**: **P5 Context degradation** — embedding drift metrics quantify when a foundation model's internal representation of a domain has shifted, the same problem agents face when long-session retrieval quality decays; **P7 Production drift** — zero-shot application avoids overfitting on small N, a pattern directly applicable to enterprise agents that must generalize across new user contexts without per-customer fine-tuning.

---

### 6. [DNBSEQ Complete WGS Pipeline](https://github.com/Complete-Genomics/DNBSEQ_Complete_WGS)
> Production-grade WGS pipeline integrating two library chemistries (PCR-free + DNB-barcoded cWGS) into phased, high-accuracy variant calls.

- **Stack**: Nextflow DSL2 · Lariat · BWA · DeepVariant · GATK · HapCUT2 · SOAPnuke · MegaBOLT · Singularity
- **Scale**: Parallel real-world human sequencing data 
- **Highlights**: Pangenome-aware DeepVariant (GBZ graph reference), phased SNP/indel/SV calls, modular process design with configurable variant callers and aligners
- **Pain points addressed**: Prerequisite for enterprise agentic reliability — demonstrates production ML under CLIA/HIPAA regulatory constraints (auditable, reproducible, versioned). Agentic systems are only trustworthy if the upstream ML tools they call meet this bar.

---

## Pain Point Coverage Matrix

| Project | P1 Multi-step | P2 Tool reliability | P3 Eval gap | P4 Observability | P5 Context | P6 Human-in-loop | P7 Prod drift |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 1. DNBSEQ WGS | | | | | | | ✓ (baseline) |
| 2. LFR DataMonitor | | ✓ | | ✓ | | | ✓ |
| 3. DeepVariant FT | | | | | | | ✓ |
| 4. Agentic bioArchitect | ✓ | ✓ | | | | ✓ | |
| 5. ZeroShot ImmuneFeatureDrift | | | | | ✓ | | ✓ |
| 6. PhasedVariants AgenticCurator | ✓ | ✓ | ✓ | | ✓ | ✓ | |

P3 (Eval gap) and P6 (Human-in-loop) are addressed by the most technically novel project (PhasedVariants) — these are the two hardest enterprise blockers and the primary differentiators of this portfolio.

---

## Competency Map

| Domain | Skills |
|---|---|
| **Genomics pipelines** | WGS · variant calling · phasing · SV · QC · Nextflow · Snakemake |
| **ML / DL** | CNN fine-tuning · transfer learning · LightGBM · autoencoders · foundation models |
| **Drift & monitoring** | Unsupervised drift detection · Isolation Forest · PCA · embedding similarity |
| **Agentic AI** | Multi-agent orchestration (CrewAI / LangChain) · RAG · FAISS · knowledge graphs · hallucination reduction |
| **MLOps** | TFX · TFRecord · Docker · Singularity · automated retraining triggers |
| **Languages & tools** | Python · Groovy · R · Bash · TensorFlow · PyTorch · HuggingFace |

---

## Architecture: Integrated Workflow

```mermaid
flowchart TD
    SEQ(["`🧬 **Sequencing Run**
    MGI stLFR · PCR-free · cWGS`"])

    subgraph MONITOR["📡 QC & Drift Detection"]
        LFR["`**2. LFR_DataMonitor**
        Snakemake · LightGBM · Autoencoders
        PCA · Isolation Forest
        Per-run feature matrices + drift score`"]
    end

    subgraph FINETUNE["🔧 Model Adaptation"]
        DFT["`**3. DeepVariant_FineTuning**
        TensorFlow · TFRecord · hap.py
        Transfer learning on pileup image tensors
        Benchmarked vs GIAB HG001/HG002`"]
    end

    subgraph PIPELINE["⚙️ Production WGS Pipeline"]
        WGS["`**1. DNBSEQ_Complete_WGS**
        Nextflow DSL2 · Lariat · BWA
        DeepVariant · GATK · HapCUT2
        Pangenome-aware · MegaBOLT`"]
    end

    subgraph DOWNSTREAM["🔬 Downstream Analysis"]
        BIO["`**4. Agentic_bioArchitect**
        CrewAI · DeepSeek · Snakemake
        Multi-agent pipeline design
        16S rRNA metatranscriptomics`"]
        ZS["`**5. ZeroShot_ImmuneFeatureDrift**
        scGPT-blood · CIBERSORTx
        Zero-shot longitudinal monitoring
        T/B/NK cell population drift`"]
    end

    subgraph INTERPRET["🤖 Agentic Clinical Interpretation"]
        PVC["`**6. PhasedVariants_AgenticCurator**
        LangChain · FAISS · PrimeKG · VEP
        DeepSeek + Grok dual-agent review
        RAG · KG · hallucination reduction`"]
    end

    VCF(["📄 Phased VCF
    SNP · Indel · SV"])

    SEQ --> LFR
    LFR -- "drift detected" --> DFT
    DFT -- "updated model weights" --> WGS
    LFR -- "QC passed" --> WGS
    WGS --> VCF
    VCF --> BIO
    VCF --> ZS
    VCF --> PVC
```

The six projects form a coherent ML system: a production pipeline (1) fed by monitored, drift-corrected models (2, 3), extended by agentic automation (4), foundation model analytics (5), and closed by automated clinical interpretation of the phased variants produced in step 1 (6).