# Event-Driven Architecture — Data Flow

> Shows how domain events flow through EventBridge connecting all MerchOS services.

```mermaid
graph TB
    subgraph Producers["Event Producers"]
        PROD_API[Product API<br/>product.created<br/>product.updated]
        IMG_PROC[Image Processor<br/>image.processed<br/>image.failed]
        AI_ENGINE[AI Engine<br/>product.enriched<br/>enrichment.failed]
        EXP_ENGINE[Export Engine<br/>export.completed<br/>export.failed]
        INV_ENGINE[Inventory Engine<br/>inventory.low_stock<br/>inventory.updated]
        SUP_ENGINE[Supplier Engine<br/>catalogue.ingested<br/>supplier.onboarded]
    end

    subgraph Bus["EventBridge — merchos-events"]
        EB[Event Bus<br/>Content-Based Routing<br/>Schema Registry<br/>30-day Archive]
    end

    subgraph Consumers["Event Consumers"]
        PIE[Product Intelligence<br/>→ Trigger enrichment]
        IIE[Image Intelligence<br/>→ Process images]
        EXPORT[Export Engine<br/>→ Check staleness]
        NOTIF[Notification Service<br/>→ Alert users]
        AUDIT[Audit Logger<br/>→ Record all events]
        ADMIN[Admin Alerts<br/>→ Platform monitoring]
    end

    Producers --> EB
    EB --> Consumers
```

---

## Event Catalogue

```mermaid
graph LR
    subgraph ProductEvents["Product Domain Events"]
        PC[product.created]
        PU[product.updated]
        PE[product.enriched]
        PA[product.approved]
        PD[product.deleted]
    end

    subgraph ImageEvents["Image Domain Events"]
        IU[image.uploaded]
        IP[image.processed]
        IF[image.failed]
    end

    subgraph ExportEvents["Export Domain Events"]
        ES[export.started]
        EC[export.completed]
        EF[export.failed]
    end

    subgraph InventoryEvents["Inventory Domain Events"]
        IUP[inventory.updated]
        ILS[inventory.low_stock]
        IOO[inventory.out_of_stock]
    end

    subgraph SupplierEvents["Supplier Domain Events"]
        SO[supplier.onboarded]
        CI[catalogue.ingested]
        CF[catalogue.failed]
    end
```

---

## Event Schema Example

```json
{
  "version": "0",
  "id": "evt-abc123",
  "source": "merchos.product-hub",
  "detail-type": "product.created",
  "account": "123456789012",
  "time": "2026-07-15T10:30:00Z",
  "region": "af-south-1",
  "detail": {
    "tenantId": "t_abc123",
    "productId": "p_def456",
    "title": "Wireless Bluetooth Headphones",
    "category": "electronics/audio",
    "hasImages": true,
    "imageCount": 3,
    "source": "manual_upload",
    "timestamp": "2026-07-15T10:30:00Z"
  }
}
```

---

## Event Routing Rules

| Event | Consumer | Action |
|-------|----------|--------|
| `product.created` | Product Intelligence Engine | Start AI enrichment workflow |
| `product.created` | Image Intelligence Engine | Process uploaded images |
| `product.enriched` | Export Engine | Mark product as export-ready |
| `product.enriched` | Notification Service | Notify seller "Ready for review" |
| `export.completed` | Notification Service | Notify seller "Export ready" |
| `export.failed` | Admin Alerts | Alert ops team |
| `inventory.low_stock` | Notification Service | Warn seller |
| `catalogue.ingested` | Product Hub | Create products from supplier data |
