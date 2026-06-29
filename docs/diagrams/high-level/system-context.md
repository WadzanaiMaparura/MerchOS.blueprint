# System Context Diagram

> High-level view of MerchOS and its external interactions.

```mermaid
graph TB
    subgraph External["External Marketplaces"]
        TK[Takealot<br/>CSV + Seller API]
        AZ[Amazon<br/>SP-API]
        MK[Makro<br/>CSV Upload]
        SP[Shopify<br/>Admin API]
        WC[WooCommerce<br/>REST API]
    end

    subgraph Users["Platform Users"]
        SELLER[Seller / Brand Owner]
        ADMIN[Platform Admin]
        AGENCY[Agency Manager]
        SUP_USER[Supplier]
    end

    subgraph MerchOS["MerchOS Platform"]
        PLATFORM[MerchOS SaaS<br/>Multi-Tenant Platform<br/>AWS Serverless]
    end

    SELLER -->|Manage products<br/>Review AI suggestions<br/>Trigger exports| PLATFORM
    ADMIN -->|Configure platform<br/>Manage tenants<br/>Monitor health| PLATFORM
    AGENCY -->|Manage multiple<br/>seller accounts| PLATFORM
    SUP_USER -->|Upload catalogues<br/>Invoices, price lists| PLATFORM

    PLATFORM -->|Export listings<br/>CSV / API push| External
    PLATFORM <-->|Sync inventory<br/>Receive webhooks| External
```

---

## Key Interactions

| Actor | Direction | Interaction |
|-------|-----------|-------------|
| Seller | Inbound | Product creation, AI review, export triggers |
| Supplier | Inbound | Catalogue uploads (CSV, PDF, Excel) |
| Marketplaces | Outbound | Listing exports, inventory sync |
| Marketplaces | Inbound | Webhooks, order notifications, schema updates |
