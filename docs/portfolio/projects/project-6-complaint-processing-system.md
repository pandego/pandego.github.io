---
title: Public complaint processing system
description: Automated multi-channel complaint intake and insights for a public entity.
---

# Public complaint processing system

!!! abstract "Case Study Summary"
    **Client**: Confidential  
    **Industry**: Public sector  
    **Role**: AI Solution Architect

    **Impact Metrics**:

    - 70% reduction in manual complaint handling time
    - 91% complaint categorization accuracy
    - Top 5 complaint themes identified within the first week
    - Unified five intake channels into one processing pipeline
    - Automated anonymization before analysis to meet privacy requirements

I built an end-to-end complaint processing system that ingests text, voice, and handwritten submissions, normalizes and anonymizes them, and delivers clustered insights to a dashboard so public-sector teams can prioritize responses quickly.

## The Challenge

The entity received complaints from email, WhatsApp, voice, and handwritten letters, but every channel required different handling and manual triage. The team needed a single pipeline that could standardize content, protect personally identifiable information, and surface top themes without slowing response times.

## The Solution

### → Implementation

I designed a staged pipeline (ingestion → normalization → anonymization → topic modeling → insights) on Google Cloud to keep processing consistent across formats. Cloud Functions orchestrated ingestion, Document AI handled OCR for handwritten letters, and Speech-to-Text converted voice messages into text. A small Vertex AI LLM standardized complaint language, clustered topics, and summarized themes for a lightweight web dashboard used by staff.

### → Solution Architecture

*Architecture diagram placeholder per request.*

### → Tech Stack

- **Infrastructure**: Google Cloud Platform
- **Data Platform**: Cloud Storage
- **CI/CD**: Cloud Build
- **Containerization**: Serverless Cloud Functions
- **Backend**: Cloud Functions, Cloud Run (dashboard API)
- **Modeling**: Vertex AI, Document AI, Speech-to-Text API
- **Vector Index**: Not used (topic clustering in Vertex AI)

## Key Learnings

- **Multichannel normalization**: Standardized text, voice, and OCR outputs early to keep downstream logic simple.
- **Privacy compliance**: Stripped PII before any LLM processing to meet regulatory requirements.
- **Cost control**: Chose a small LLM in Vertex AI to balance accuracy with operating costs.
- **Operational relevance**: Tailored cluster summaries to the language used by frontline teams.

## Measurable Impact

- 70% reduction in manual complaint handling time
- 91% complaint categorization accuracy
- Top five complaint themes identified within the first week
- Faster prioritization of response plans using cluster summaries

<div class="grid cards" style="margin-top: 3rem" markdown>

-   :material-coffee:{ .lg .middle } Let's have a virtual coffee together!

    ---

    Want to see if we're a match? Let's have a chat and find out. Schedule a free 30-minute strategy session to discuss your AI challenges and explore how we can work together.

    [Book Intro Call :material-arrow-top-right:](https://calendar.app.google/2FvRfdbidFFGPKcu6){ .md-button .md-button--primary target="_blank" }
</div>
