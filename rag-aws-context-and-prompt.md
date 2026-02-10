# RAG on AWS — Discussion Summary & Reusable Tasking Prompt

## Summary of our discussion

### Context
- **Current system:** Containerized Go app on-prem that does RAG using the ChatGPT API and PostgreSQL with pgvector.
- **Goal:** Implement the same RAG capability on AWS with minimal operational burden and long-term production in mind.

### Decisions made
- **Option 1 (ECS):** ECS (Fargate) + ECR + **S3 Vectors** (vector store) + S3 (document storage) + Bedrock (embeddings + LLM). OpenSearch is **not** used.
- **Option 2 (EKS):** EKS + ECR + **Aurora PostgreSQL with pgvector** (vector store) + S3 (documents) + Bedrock. OpenSearch is **not** used.
- **Scale:** Very few users; documents = legal PDFs + some video; volume TBD.
- **Ops:** Minimal maintenance; team wants to focus on development, not infrastructure.
- **App:** Exists on-prem; team is willing to adapt the app (e.g. from pgvector SQL to S3 Vectors API if choosing Option 1).

### Outcome
- **Design doc:** `docs/aws-rag-implementation-design.md` — validates both options, includes ASCII + Mermaid diagrams, comparison (complexity + cost), bare minimum vs best practice, and recommendation.
- **Recommendation:** Option 1 (ECS + S3 Vectors) for lower ops and cost; trade-off is adapting the Go app to use the AWS SDK for Go v2 `s3vectors` client instead of pgvector SQL. Option 2 (EKS + Aurora pgvector) is valid if you prefer zero app change for the vector layer and accept higher ops and cost.

### Key references
- [Amazon S3 Vectors](https://aws.amazon.com/s3/features/vectors/) — product overview  
- [S3 Vectors user guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-vectors.html)  
- [AWS SDK for Go v2 — s3vectors](https://pkg.go.dev/github.com/aws/aws-sdk-go-v2/service/s3vectors)

---

## Reusable prompt for future iterations

Copy and paste the block below when you want to iterate on this RAG-on-AWS work. Add your specific task at the end.

```markdown
## Context (RAG on AWS — do not re-derive)

We are implementing a RAG application on AWS. The existing design is in this repo:

- **Design doc:** `docs/aws-rag-implementation-design.md` — read it for full architecture, diagrams, comparison, and bare minimum vs best practice.
- **Context summary:** `docs/rag-aws-context-and-prompt.md` (this file).

**Decisions already made:**
- Two options only: (1) **ECS + S3 Vectors + S3 + Bedrock**, (2) **EKS + Aurora pgvector + S3 + Bedrock**. OpenSearch is **not** used.
- Target: minimal ops, long-term production, very few users; documents = legal PDFs + some video.
- Current on-prem: Go app + ChatGPT API + PostgreSQL with pgvector; team can adapt the app (e.g. to S3 Vectors API).
- Recommended option: **ECS + S3 Vectors** for lower ops and cost; app must use AWS SDK for Go v2 `s3vectors` instead of pgvector SQL.

**Canonical components:**
- Compute: ECS Fargate (Option 1) or EKS (Option 2).
- Registry: ECR.
- Documents: S3.
- Vector store: S3 Vectors (Option 1) or Aurora PostgreSQL with pgvector (Option 2).
- Embeddings + LLM: Bedrock (Titan Embed + chat model).

Do **not** reintroduce OpenSearch or “S3 only for vectors” (we use **S3 Vectors**, which is a dedicated vector store in S3 with native APIs). Do **not** assume different scale/ops goals unless I say so.

## Task (fill in)

[Describe what you want to do next, e.g.:]
- Add a minimal Terraform outline for Option 1 (ECS + S3 Vectors).
- Draft a migration checklist for the Go app: pgvector → S3 Vectors (APIs, IAM, config).
- Add a section to the design doc for disaster recovery / backups.
- Compare S3 Vectors vs Aurora pgvector for our exact document volume (assume X docs, Y GB).
```

---

## Example follow-up tasks you might add

- *“Add a minimal Terraform outline for Option 1 (ECS + S3 Vectors + S3 + Bedrock), following the bare minimum in the design doc.”*
- *“Draft a migration checklist for the Go app: pgvector SQL → S3 Vectors API (code changes, IAM, env/config).”*
- *“Add a short section to the design doc on backups and disaster recovery for Option 1.”*
- *“Extend the cost comparison with assumed document/vector volume (e.g. 100k docs, 50M vectors).”*
- *“Produce an infrastructure checklist (IAM, networking, security) for Option 1 before first deploy.”*
