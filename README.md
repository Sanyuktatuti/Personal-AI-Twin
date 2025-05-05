# Personal-AI-Twin
Personal AI/LLM: scrapes my posts, streams them through MongoDB CDC→RabbitMQ→Bytewax, embeds into Qdrant, fine‑tunes an 8B model (LoRA/QLoRA) on AWS SageMaker, deploys a RAG API + Gradio UI that writes like me. Terraform + Docker + CI/CD, Comet/Opik for tracking, sub‑second responses, &lt;$0.05/1k queries. Plug‑and‑play.
