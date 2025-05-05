# Personal-AI-Twin
Personal AI/LLM: scrapes my posts, streams them through MongoDB CDC→RabbitMQ→Bytewax, embeds into Qdrant, fine‑tunes an 8B model (LoRA/QLoRA) on AWS SageMaker, deploys a RAG API + Gradio UI that writes like me. Terraform + Docker + CI/CD, Comet/Opik for tracking, sub‑second responses, &lt;$0.05/1k queries. Plug‑and‑play.
# 🧑‍💻 LLM Twin – My Personal AI Replica

---

## Table of Contents

1. [Project Goals](#project-goals)
2. [Architecture](#architecture)
3. [Quick Start](#quick-start)
4. [Detailed Pipelines](#detailed-pipelines)
5. [Tech Stack](#tech-stack)
6. [Project Structure](#project-structure)
7. [Cost Footprint](#cost-footprint)
8. [Contributing](#contributing)
9. [License](#license)

---

## Project Goals

* **Create an "LLM twin"** that mimics my writing voice across blogs and code comments.
* Showcase **production‑grade LLMOps**: reproducible data pipelines, experiment tracking, model registry, prompt monitoring, and automated deployment.
* Keep costs < \$10 by relying on free tiers + spot instances.

## Architecture

<p align="center">
  <img src="docs/architecture.png" alt="System architecture diagram" width="700"/>
</p>

### High‑level Flow

1. **Data Collection Pipeline** – Serverless crawlers pull content from Medium, Substack, LinkedIn, and GitHub into MongoDB; changes are emitted via CDC to RabbitMQ.
2. **Feature Pipeline** – Bytewax ingests the queue, cleans & chunks text, generates embeddings, and upserts vectors to Qdrant (optionally Redis) in real‑time.
3. **Training Pipeline** – Auto‑builds an instruction dataset, fine‑tunes an 8 B model with LoRA/QLoRA on SageMaker, logs runs to Comet ML, evaluates with Opik, and versions artifacts on Hugging Face Hub.
4. **Inference Pipeline** – Loads & quantizes the accepted model, wraps a RAG layer with hybrid BM25 + vector search, exposes a REST API on SageMaker endpoints, monitors prompts/answers, and surfaces a Gradio UI.

## Quick Start

```bash
# Clone & setup
git clone https://github.com/<your‑handle>/llm‑twin.git
cd llm‑twin
cp .env.example .env   # fill in secrets
make bootstrap          # installs deps & pre‑commit hooks

# 1️⃣ Kick off data crawl (runs as AWS Lambda locally)
make crawl

# 2️⃣ Start streaming feature pipeline
make stream

# 3️⃣ Launch fine‑tuning job on SageMaker (uses spot instances)
make train

# 4️⃣ Deploy inference API + UI
make deploy
```

> **Note:** Detailed, step‑by‑step instructions live in [`docs/INSTALL_AND_USAGE.md`](docs/INSTALL_AND_USAGE.md).

## Detailed Pipelines

| Stage                 | Highlights                                                          |
| --------------------- | ------------------------------------------------------------------- |
| **Data → Queue**      | ETL in Python, packaged as Lambda; MongoDB → RabbitMQ CDC connector |
| **Queue → Vector DB** | Bytewax stream; Superlinked refactor cuts 70 % of boilerplate       |
| **Fine‑tune**         | 8 B model, LoRA rank = 64, 4‑bit QLoRA; tracked with Comet          |
| **Evaluation**        | Automatic perplexity & style metrics via Opik; manual spot checks   |
| **Serving**           | Text‑generation‑inference (TGI) container on SageMaker, autoscaling |
| **Monitoring**        | Prompt/response logging, latency & cost dashboards                  |

## Tech Stack

| Layer             | Tools                                                |
| ----------------- | ---------------------------------------------------- |
| **Lang/Runtime**  | Python 3.11, Docker, Makefile                        |
| **Data**          | MongoDB, RabbitMQ, Bytewax                           |
| **Vector Store**  | Qdrant, Redis (VS)                                   |
| **Modeling**      | Hugging Face Transformers, LoRA/QLoRA, AWS SageMaker |
| **Observability** | Comet ML, Opik, CloudWatch                           |
| **Infra‑as‑Code** | Terraform, GitHub Actions CI/CD                      |
| **UI**            | Gradio                                               |

## Project Structure

```text
llm‑twin/
├── src/
│   ├── data_crawling/
│   ├── data_cdc/
│   ├── feature_pipeline/
│   ├── training_pipeline/
│   └── inference_pipeline/
├── terraform/           # IaC for AWS resources
├── docs/                # Diagrams & guides
├── .env.example
├── Makefile
└── pyproject.toml
```

## Cost Footprint

| Provider          | Purpose                     | Est. Cost |
| ----------------- | --------------------------- | --------- |
| OpenAI (optional) | GPT‑4 eval prompts          | \~ \$1    |
| AWS SageMaker     | Fine‑tune + endpoint (spot) | < \$10    |
| All others        | Free tier                   |           |

## Contributing

Found a bug or have an idea? PRs & issues welcome! See [`CONTRIBUTING.md`](CONTRIBUTING.md).

## License

MIT – free to fork, tweak, and deploy. Just keep the license & attribution.

---

> *Inspired by the excellent LLM Twin framework, re‑implemented & customised for my own workflow.*
