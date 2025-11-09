# De-Identification of PHI in Electronic Health Records (EHR)



## Overview

This project provides a production-oriented **Python-based de-identification engine** for Electronic Health Records (EHR). It automatically detects and processes Protected Health Information (PHI) according to **HIPAA Safe Harbor** guidance, producing data that is safe for research and analytics while preserving maximal information utility.

The codebase includes components for:

* NLP-based PHI detection (token + pattern matching)
* Data transformation and masking strategies
* Batch and streaming ingestion
* Cloud storage integration (AWS S3)
* Persistent storage of results in MongoDB
* A lightweight React/Node.js front-end for visualization and job management

---

## Key Features

* **Automated PHI detection** using hybrid NLP + rule-based approach
* **HIPAA Safe Harbor compliance** — removes or masks the 18 identifiers
* **Flexible input support**: CSV, JSON, HL7, FHIR excerpts, and plain text notes
* **Configurable masking strategies**: redaction, pseudonymization, generalization
* **Secure cloud integration** with AWS S3 for file upload/download
* **Audit logs & provenance** for traceability of every transformation
* **Batch & real-time processing** support
* **Developer-friendly API** for integration into pipelines

---

## Architecture & Components

1. **De-ID Engine (Python)** — core NLP, pattern matching, and transformation modules.
2. **API Layer (Node.js / Express)** — exposes REST endpoints for job submission, status, and retrieval.
3. **Web UI (React)** — job management, sample previews, and audit logs.
4. **Storage**

   * **MongoDB**: stores anonymized records, job metadata, and audit trails.
   * **AWS S3**: raw and processed file storage, with server-side encryption recommended.
5. **Worker / Batch Processor** — orchestrates large-scale de-identification jobs (Celery / RQ compatible).

Diagram (conceptual): De-ID Engine ⇄ API ⇄ UI ⇄ Storage (MongoDB, S3)

---

## Supported PHI Types & De-identification Rules

The implementation targets the 18 HIPAA Safe Harbor identifiers, including but not limited to:

* Names (patients, relatives, providers)
* All geographic subdivisions smaller than a state (addresses, postal codes)
* All elements of dates (except year) directly related to an individual
* Telephone and fax numbers
* Email addresses
* Social Security numbers, medical record numbers, and other unique IDs
* Biometric identifiers and full-face photos

**Default behavior** is conservative: remove or mask all risky fields. Each field's transformation is configurable via the `config.yml`.

---

## Install & Quick Start

**Prerequisites**

* Python 3.8+ (recommend 3.10+)
* Node.js 16+
* MongoDB 4.4+ or managed Atlas
* AWS account with S3 bucket

**Quick local run (development)**

1. Clone the repo

```bash
git clone <repo-url>
cd deid-ehr
```

2. Python env and dependencies

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

3. Start API & workers

```bash
# from /server
npm install
npm run dev
# from /workers
python workers/run_worker.py
```

4. Run the UI

```bash
# from /client
npm install
npm start
```

---

## Configuration

Configuration is driven by `config.yml` with clear sections for:

* PHI detection thresholds (confidence cutoffs)
* Masking strategy per field (e.g., `redact`, `pseudonymize`, `hash`)
* Storage backends and credentials (MongoDB URI, S3 bucket)
* Audit log retention and verbosity

**Secrets** must be injected via environment variables or a secrets manager (do not store secrets in the repo).

---

## Usage

### CLI

Simple single-file de-identification:

```bash
python deid.py --input sample/clinical_note.txt --output out/cleaned.json --config config.yml
```

### API

* `POST /jobs` — submit a de-identification job (file or S3 pointer)
* `GET /jobs/{id}` — check job status and logs
* `GET /results/{id}` — download anonymized output

### Batch Processing

Submit a manifest file or S3 prefix and the worker pool will process in parallel with retry semantics and checkpointing.

---

## Input Formats & Data Handling

Supported formats:

* Structured: CSV, JSON, MongoDB documents, FHIR (JSON)
* Semi-structured: free-text clinical notes, HL7 v2 segments

Pre-processing steps include charset normalization, sentence segmentsation, and basic PII pattern heuristics to increase recall.

---

## Security, Compliance & Auditability

* **HIPAA-aware design**: retains only the minimum necessary data for analytics
* **Encryption**: TLS for data in transit; S3 server-side encryption and encrypted MongoDB at rest recommended
* **Access control**: RBAC for UI/API endpoints
* **Audit logs**: detailed transformation logs (what was changed, by which module, and why)
* **Data retention**: configurable retention policies for raw and processed data

---

## Testing & Validation

* Unit tests for tokenizers, pattern matchers, and transform modules (`pytest`)
* Integration tests for end-to-end job processing
* Synthetic data generator for validation and privacy benchmarking

---

## Deployment Options

* **Docker**: `docker-compose.yml` provided for local/QA deployments
* **Kubernetes**: Helm chart template for production-scale orchestration
* **Cloud**: example deployment scripts for AWS (ECS/EKS) and CI/CD pipelines

---

## Contributing

Guidelines:

* Follow the repo style and linting rules
* Write tests for new features and edge cases
* Submit PRs with descriptive titles and linked issue numbers

---

## License

This project is offered under the **MIT License**. See `LICENSE` for details.

---

## Contact & Acknowledgements

For questions or commercial support, contact: `dev-team@example.com`.

Thanks to the open-source projects and libraries used in this project (spaCy, regex, MongoDB, React, Express, AWS SDK).

*Last updated: November 9, 2025*
