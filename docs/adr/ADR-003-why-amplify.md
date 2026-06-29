# ADR-003: Why Amplify Instead of EC2?

| Field | Value |
|-------|-------|
| **Status** | Approved |
| **Date** | 2026-06-27 |
| **Decision Makers** | Wadzanai Maparura |
| **Category** | Frontend Hosting & Deployment |

---

## Context

MerchOS frontend is a Next.js 14 application requiring:
- Server-side rendering (SSR) for initial page loads
- Static generation for marketing pages
- CI/CD pipeline with branch previews for PRs
- Custom domain with automatic SSL
- Global CDN for static asset delivery
- Environment variable management per deployment stage

## Decision

Use **AWS Amplify Hosting** for the Next.js frontend instead of self-managed EC2/ECS/CloudFront infrastructure.

## Rationale

| Requirement | Amplify Solution |
|-------------|-----------------|
| Next.js SSR | Native Next.js support (build + deploy automatically) |
| CI/CD | Git-connected — push triggers build + deploy |
| Branch previews | Automatic preview URLs for every PR |
| Custom domain | One-click setup with auto-provisioned ACM certificate |
| CDN | CloudFront included (global edge delivery) |
| Env variables | Per-branch environment variable configuration |
| Cost | Pay-per-use; no idle servers |

## Alternatives Considered

| Alternative | Reason Rejected |
|-------------|----------------|
| **EC2 + Nginx + PM2** | Maximum ops overhead; manual scaling; patching; load balancer config; ~$50+/month idle |
| **ECS Fargate + ALB** | Over-engineering for a frontend app; higher cost; container management |
| **Vercel** | Non-AWS — adds vendor; data transfer costs between Vercel and AWS backend; no af-south-1 edge |
| **S3 + CloudFront (static)** | No SSR support; limits Next.js capabilities |
| **CloudFront + Lambda@Edge** | Complex custom setup; Amplify provides this out-of-box |

## Consequences

### Positive
- Zero server management for frontend
- Automatic CI/CD on every git push
- Branch previews enable easy PR review
- CloudFront CDN delivers assets globally with low latency
- SSL/TLS managed automatically
- Cost: ~$20–$100/month (vs. $50–$200 for EC2-based)
- Scales automatically with traffic

### Negative
- Less control over build environment (Amplify uses pre-defined build images)
- Build timeout limit (30 minutes — sufficient for MerchOS)
- SSR compute pricing can increase with high traffic
- Debugging build failures requires Amplify-specific knowledge
- Fewer customisation options vs. raw CloudFront + Lambda@Edge

### Why Not EC2 Specifically?

| Factor | EC2 | Amplify |
|--------|-----|---------|
| Setup time | Hours/days | Minutes |
| Maintenance | Patches, updates, monitoring | Zero |
| Scaling | Manual or ASG config | Automatic |
| CI/CD | Must build separately (CodePipeline, GitHub Actions) | Built-in |
| Cost (idle) | ~$50/month minimum | ~$0 |
| SSL | Manual ACM + ALB setup | Automatic |

---

## References

- AWS Amplify Hosting documentation
- Next.js on AWS — deployment options comparison
- MerchOS MERCH-016 (Frontend Architecture)
