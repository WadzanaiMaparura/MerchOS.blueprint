# AWS Infrastructure Diagram

> Complete AWS service topology for MerchOS.

```mermaid
graph TB
    subgraph Client["Client Layer"]
        BROWSER[Browser<br/>Next.js App]
    end

    subgraph Edge["AWS Edge"]
        CF[CloudFront<br/>via Amplify]
        WAF[AWS WAF<br/>Rate Limiting + Protection]
    end

    subgraph Region["af-south-1 (Cape Town)"]
        subgraph Auth["Authentication"]
            COGNITO[Amazon Cognito<br/>User Pool + App Client]
        end

        subgraph API["API Layer"]
            APIGW[API Gateway<br/>HTTP API v2<br/>29s timeout]
        end

        subgraph Compute["Compute Layer (Lambda — arm64)"]
            L_AUTH[Auth Handlers<br/>512MB / 10s]
            L_PROD[Product API<br/>512MB / 29s]
            L_EXP[Export Handlers<br/>1024MB / 15min]
            L_AI[AI Orchestrator<br/>1024MB / 15min]
            L_IMG[Image Processor<br/>2048MB / 5min]
            L_INV[Inventory API<br/>512MB / 29s]
            L_SUP[Supplier Processor<br/>1024MB / 15min]
        end

        subgraph Orchestration["Orchestration"]
            SF[Step Functions<br/>Standard Workflows]
            EB[EventBridge<br/>Custom Event Bus]
            SQS_IMPORT[SQS: Bulk Import]
            SQS_AI[SQS: AI Enrichment]
            SQS_IMG[SQS: Image Processing]
            SQS_EXPORT[SQS: Export Chunks]
        end

        subgraph Data["Data Layer"]
            DDB[(DynamoDB<br/>On-Demand<br/>PITR Enabled)]
            S3_MEDIA[(S3: Media<br/>Intelligent-Tiering)]
            S3_EXPORT[(S3: Exports<br/>90d Lifecycle)]
            S3_IMPORT[(S3: Imports<br/>30d Lifecycle)]
            SECRETS[Secrets Manager<br/>API Keys + Rotation]
        end

        subgraph Monitoring["Observability"]
            CW[CloudWatch<br/>Logs + Metrics]
            XRAY[X-Ray<br/>Distributed Traces]
            SNS[SNS<br/>Alert Topics]
        end
    end

    subgraph AI_Region["us-east-1 (AI Services)"]
        BEDROCK[Amazon Bedrock<br/>Claude 3.5 Sonnet<br/>Claude 3 Haiku]
        TEXTRACT[Amazon Textract<br/>Document OCR]
        REKOG[Amazon Rekognition<br/>Image Analysis]
    end

    BROWSER --> CF --> WAF --> APIGW
    APIGW --> COGNITO
    APIGW --> Compute
    Compute --> Orchestration
    Compute --> Data
    Compute --> AI_Region
    Orchestration --> Compute
    Compute --> Monitoring
```

---

## AWS Account Strategy

```mermaid
graph TB
    ORG[AWS Organisation Root]
    ORG --> OU_PROD[OU: Production]
    ORG --> OU_NONPROD[OU: Non-Production]
    ORG --> OU_SHARED[OU: Shared Services]
    
    OU_PROD --> ACC_PROD[merchos-prod<br/>Production workloads]
    OU_NONPROD --> ACC_STAGING[merchos-staging<br/>Pre-production]
    OU_NONPROD --> ACC_DEV[merchos-dev<br/>Development]
    OU_SHARED --> ACC_SHARED[merchos-shared<br/>CI/CD, Monitoring, Security]
```

---

## Network Architecture (VPC-Less)

```mermaid
graph LR
    subgraph Public["Public Internet"]
        USER[End Users]
        MKTPLACE[Marketplace APIs]
    end

    subgraph AWS_Edge["AWS Edge"]
        CF[CloudFront]
        WAF[WAF Rules]
    end

    subgraph Serverless["af-south-1 — VPC-Less Services"]
        APIGW[API Gateway]
        LAMBDA[Lambda Functions]
        DDB[DynamoDB]
        S3[S3 Buckets]
        SQS[SQS Queues]
        EB[EventBridge]
        SF[Step Functions]
        COGNITO[Cognito]
    end

    USER --> CF --> WAF --> APIGW --> LAMBDA
    LAMBDA --> DDB
    LAMBDA --> S3
    LAMBDA --> SQS
    LAMBDA --> MKTPLACE
```

> **Note:** MerchOS uses a fully VPC-less architecture. All services communicate via AWS service endpoints. No VPC, subnets, or NAT Gateways are required — reducing cost by ~$32/month and eliminating network complexity.
