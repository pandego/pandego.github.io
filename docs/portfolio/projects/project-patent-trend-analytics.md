---
title: Patent Trend Analysis with LLMs
description: End-to-end AI pipeline for classifying, translating, clustering, and summarizing patent filings to surface EV-battery technology trends for IP and innovation teams.
---

# Patent Trend Analysis with LLMs

!!! abstract "Case Study Summary"
    **Client**: Confidential / Leading European EV Battery Manufacturer  
    **Industry**: Automotive (EV batteries)  
    **Role**: AI Tech Lead
    
    **Impact Metrics**:

    - Analysts reallocated from manual tagging to higher-value trend interpretation
    - -90% review/labeling time per weekly batch (from **5 days → 0.5 day**)  
    - **≈ €490k/year** cost savings (based on 10 IP engineers' time saved)

The IP team needed faster, consistent insight into thousands of new patents to inform R&D on emerging EV-battery technologies. I delivered a pipeline that turns raw patent exports into titled clusters with summaries and a trends dashboard.

## 🧩 The Challenge

IP engineers were manually scanning and tagging multilingual patents—slow, inconsistent, and hard to replicate at scale. Multilingual content and unstructured abstracts made it difficult to compare filings and report consolidated trends to innovation stakeholders.

## 💡 The Solution

### → Implementation ⚙️
I built REST endpoints for **language detection → translation → domain-specific embedding → unsupervised clustering → cluster summarization → trend tracking**, exposed via a FastAPI service on Azure and backed by a vector index for semantic lookups. Clusters receive concise titles/summaries and can optionally align to **IPC** categories for consistent reporting.

### → Solution Architecture 🏗️

<img src="../../assets/patent-trend-analysis-architecture.png" alt="Architecture Diagram" width="600">

*Baseline EV-battery patent trend analytics solution architecture*

### → Tech Stack 🧰
- **Cloud**: Microsoft Azure Cloud Infrastructure
- **CI/CD**: Azure DevOps Pipelines
- **Containerization**: Docker
- **Data Platform**: Azure Databricks (APIs for ingestion & jobs)
- **Vector Index**: Databricks Vector Search
- **Backend**: Python services with FastAPI (REST)
- **Language Detection**: [XLM-Roberta](https://huggingface.co/qanastek/51-languages-classifier)
- **Translation**: [mBART-large-50 many to one](https://huggingface.co/facebook/mbart-large-50-many-to-one-mmt)
- **Embeddings**: [BERT for Patents](https://huggingface.co/anferico/bert-for-patents) (fine-tuned for EV-battery patents)
- **LLM (summarization)**: [Mistral 7B](https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.2)


## 📚 Key Learnings

- **Multilingual filings**: Used XLM-RoBERTa + mBART-50 to standardize language before embedding.
- **Noisy abstracts & jargon**: Fine-tuned domain embeddings (BERT for Patents) to boost semantic cohesion before clustering.
- **Inconsistent labels**: Auto-titled clusters with Mistral-7B; optionally mapped to IPC-labels.
- **Scalability & repeatability**: Orchestrated Databricks jobs with containerized services and Azure DevOps CI/CD for stable weekly runs.
- **Analyst adoption**: Added concise summaries + trends dashboard, shifting effort from manual tagging to interpretation.

## 📊 Measurable Impact

- **Earlier visibility** of emerging battery-tech themes; analysts redeployed to higher-value analysis.
- Analysts reallocated from manual tagging to **higher-value trend interpretation**
- -90% review/labeling time per weekly batch (from **5 days → 0.5 day**)  
- **≈ €490k/year** cost savings (based on 10 IP engineers' time saved alone)

<div class="grid cards" style="margin-top: 3rem" markdown>

-   :material-coffee:{ .lg .middle } Let's have a virtual coffee together!

    ---
    
    Want to see if we're a match? Let's have a chat and find out. Schedule a free 30-minute strategy session to discuss your AI challenges and explore how we can work together.

    [Book Intro Call :material-arrow-top-right:](https://calendar.app.google/2FvRfdbidFFGPKcu6){ .md-button .md-button--primary target="_blank" }

</div>
