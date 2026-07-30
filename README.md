# AI Platform Lab

Hands-on lab building the core components of an internal AI provisioning platform:
model gateway, observability, and infrastructure-as-code on AWS.

## Components

### 1. LiteLLM Gateway (`litellm/`)
An OpenAI-compatible gateway in front of a local Ollama model (llama3.2:3b).
One endpoint, provider-agnostic. Supports rate limiting and cost tracking.

- Runs on port 4000
- `config.yaml` defines model routing and Langfuse callbacks

### 2. Langfuse Observability
Self-hosted via Docker Compose. Every request through LiteLLM is traced:
prompt, response, token usage, and latency.

- Wired to LiteLLM via `success_callback` / `failure_callback`
- Credentials passed as environment variables, never in code

### 3. Terraform + AWS (`terraform/`)
Infrastructure as Code. An S3 bucket created, planned, and destroyed
entirely through Terraform instead of the console.

- IAM user with least-privilege (S3-only) access
- AWS credentials via environment variables, never committed

## Problems solved along the way

- **Langfuse SDK v3 vs LiteLLM**: LiteLLM's `langfuse` callback requires the
  v2 Python SDK. v3 threw `module 'langfuse' has no attribute 'version'`.
  Fixed by pinning `langfuse>=2.0.0,<3.0.0`.
- **Terraform credentials**: `No valid credential sources found` was resolved
  by exporting AWS keys as environment variables in the same shell.

## Security notes

- `.gitignore` excludes Terraform state and any secret/env files
- Terraform state can contain plaintext credentials, so it is never committed
- IAM follows least-privilege: the Terraform user has S3 access only
