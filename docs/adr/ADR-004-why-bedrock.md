# ADR-004: Why Bedrock?

| Field | Value |
|-------|-------|
| **Status** | Approved |
| **Date** | 2026-06-27 |
| **Decision Makers** | Wadzanai Maparura |
| **Category** | AI / Machine Learning |

---

## Context

MerchOS requires LLM capabilities for:
- Product description generation (marketplace-quality copywriting)
- Attribute extraction from unstructured text and images
- Category recommendation using marketplace taxonomies (RAG)
- SEO keyword identification
- Multi-language translation
- Data quality assessment

The platform needs AI that is cost-effective, scalable, and doesn't require ML engineering expertise to operate.

## Decision

Use **Amazon Bedrock** with Claude 3.5 Sonnet (primary) and Claude 3 Haiku (cost-optimised fallback) for all LLM inference. Use Bedrock Knowledge Bases for RAG (category recommendation).

## Rationale

| Requirement | Bedrock Solution |
|-------------|-----------------|
| No infrastructure | Fully managed — no GPUs, no model hosting, no scaling |
| Pay-per-use | Token-based pricing — $0 when idle |
| Multiple models | Claude 3.5 Sonnet, Claude 3 Haiku, Titan — switch without code changes |
| Quality | Claude 3.5 Sonnet — top-tier for generation and extraction |
| Safety | Bedrock Guardrails — content filtering, topic blocking, grounding |
| RAG | Built-in Knowledge Bases with Titan Embeddings |
| AWS native | IAM authentication; no API keys; CloudWatch metrics; X-Ray traces |
| Compliance | Data not used for model training; HIPAA eligible |

## Alternatives Considered

| Alternative | Reason Rejected |
|-------------|----------------|
| **OpenAI API (GPT-4)** | Non-AWS vendor; data leaves AWS; API key management; no IAM integration; data residency concerns |
| **Self-hosted (SageMaker)** | GPU instances required (~$1,000+/month idle); ML engineering expertise needed; operational complexity |
| **Hugging Face on SageMaker** | Same GPU cost issues; model quality below Claude for our tasks |
| **Cohere** | Smaller model selection; less capable for complex extraction; additional vendor |
| **Google Vertex AI** | Non-AWS; cross-cloud data transfer; additional vendor relationship |

## Consequences

### Positive
- Zero ML infrastructure management
- Pay-per-token aligns cost with usage (startup-friendly)
- Claude 3.5 Sonnet produces high-quality product content
- Bedrock Guardrails prevent harmful/non-compliant output
- Knowledge Bases enable RAG without building vector search infrastructure
- Same IAM permissions model as all other AWS services
- CloudWatch integration for monitoring tokens, latency, errors

### Negative
- Cross-region calls may be needed (Bedrock may not be in af-south-1)
- Rate limits require queuing and backpressure management
- Token costs are the #1 variable cost driver (~$0.03/product with Sonnet)
- Model version updates could change output quality (pin versions)
- Limited fine-tuning options (prompt engineering is primary lever)

### Cost Projection

| Scale | Monthly AI Cost | Strategy |
|-------|----------------|----------|
| 1,000 products/month | ~$30 | Sonnet for all |
| 10,000 products/month | ~$150 | Mixed Sonnet + Haiku |
| 100,000 products/month | ~$800 | Haiku primary, Sonnet for complex |

---

## References

- Amazon Bedrock User Guide
- Anthropic Claude Model Card
- MerchOS MERCH-007 (AI Architecture)
