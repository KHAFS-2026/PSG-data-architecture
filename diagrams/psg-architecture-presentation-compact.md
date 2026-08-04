# PSG Future State Data Architecture - Presentation Ready (Compact)
## Consolidated 21 Models → 6 | Unified Lakehouse Platform

---

```mermaid
flowchart TB
    subgraph Sources["📊 DATA SOURCES"]
        direction LR
        SQL["7 SQL Databases<br/>Daily Batch"]
        EXCEL["18 Excel Files<br/>Multiple Schedules"]
        AIRTABLE["Airtable<br/>Environmental Data"]
    end

    subgraph Ingestion["🔄 INGESTION PIPELINES<br/>(Fabric Data Factory)"]
        direction TB
        INGEST_PAD1[" "]
        INGEST_PAD2[" "]
        INGEST_PAD3[" "]
        INGEST_ROW[" "]
        PipelineSQL["SQL Pipeline"]
        PipelineEXCEL["Excel Pipelines<br/>(Hourly / Daily / Weekly)"]
        PipelineAIR["Airtable Pipeline"]
        INGEST_PAD1 --> INGEST_PAD2 --> INGEST_PAD3 --> INGEST_ROW
        INGEST_ROW --- PipelineSQL
        INGEST_ROW --- PipelineEXCEL
        INGEST_ROW --- PipelineAIR
    end

    subgraph Lakehouse["💾 UNIFIED LAKEHOUSE"]
        direction LR
        Bronze["🔵 Bronze<br/>Raw Data"]
        Silver["🟢 Silver<br/>Business Rules"]
        Gold["🟡 Gold<br/>Optimized"]
    end

    subgraph MDM["🎯 MASTER DATA LAYER<br/>(14 Shared Dimensions)"]
        direction TB
        MDM_PAD1[" "]
        MDM_PAD2[" "]
        MDM_PAD3[" "]
        DIMS["DimCustomer • DimProduct<br/>DimLocation • DimDate • DimEmployee<br/>+ 9 Excel-based Dimensions"]
        MDM_PAD1 --> MDM_PAD2 --> MDM_PAD3 --> DIMS
    end

    subgraph Models["📊 SEMANTIC MODELS<br/>(6 total, down from 21)"]
        direction TB
        M1["1️⃣ Sales & Revenue<br/>(5 → 1)"]
        M2["2️⃣ Operations<br/>(4 → 1)"]
        M3["3️⃣ Finance<br/>(2 → 1)"]
        M4["4️⃣ Quality<br/>(4 → 1)"]
        M5["5️⃣ HR / Workforce<br/>(3 → 1)"]
        M6["6️⃣ Training<br/>(1 → 1)"]
    end

    subgraph Apps["📈 POWER BI APPS<br/>(17 total with RLS)"]
        APPS["PSG Sales • PSG Reports • Kroger QA<br/>PSG Aquaculture • VCQ • HR KPIs<br/>+ 11 more apps"]
    end

    subgraph GOV["🛡️ GOVERNANCE<br/>(Purview • Monitoring • RLS)"]
        gov["Full Lineage<br/>& Metadata"]
    end

    %% MAIN FLOW
    SQL --> PipelineSQL
    EXCEL --> PipelineEXCEL
    AIRTABLE --> PipelineAIR

    PipelineSQL --> Bronze
    PipelineEXCEL --> Bronze
    PipelineAIR --> Bronze

    Bronze --> Silver --> Gold
    Silver --> DIMS

    Gold --> Models
    DIMS --> Models
    Models --> APPS

    %% GOVERNANCE LINKS
    Bronze -.-> gov
    Silver -.-> gov
    Gold -.-> gov
    APPS -.-> gov

    %% STYLES
    style Sources fill:#e1f5ff
    style Ingestion fill:#ffd4d4
    style Lakehouse fill:#f3e5f5
    style MDM fill:#c8e6c9
    style Models fill:#e8f5e9
    style Apps fill:#fce4ec
    style GOV fill:#ede7f6
    style INGEST_PAD1 fill:transparent,stroke:transparent,color:transparent
    style INGEST_PAD2 fill:transparent,stroke:transparent,color:transparent
    style INGEST_PAD3 fill:transparent,stroke:transparent,color:transparent
    style MDM_PAD1 fill:transparent,stroke:transparent,color:transparent
    style MDM_PAD2 fill:transparent,stroke:transparent,color:transparent
    style MDM_PAD3 fill:transparent,stroke:transparent,color:transparent
    style INGEST_ROW fill:transparent,stroke:transparent,color:transparent
```

---

## CONSOLIDATION AT A GLANCE

| Element | Current State | Future State | Benefit |
|---------|---------------|--------------|---------|
| **Semantic Models** | 21 (scattered, redundant) | 6 (consolidated, governed) | 71% reduction |
| **Data Storage** | 3 systems (duplicated) | 1 unified Lakehouse | Single source of truth |
| **Excel Integration** | Direct to Power BI | Lakehouse + Governance | Full audit trail |
| **Shared Dimensions** | Duplicated per model | 14 shared MDM layer | No data inconsistency |
| **Row-Level Security** | Separate models | RLS on unified models | Eliminates redundancy |

---

## SIX CONSOLIDATED SEMANTIC MODELS

### 1️⃣ **Sales & Revenue** 
Consolidates 5 models → AS400 + SAP + Excel Sales Data
- **Apps**: PSG Sales, Kroger QA, Inventory Reports
- **RLS**: Kroger | Amazon filtering

### 2️⃣ **Operations**
Consolidates 4 models → Production tracking + Inventory
- **Apps**: Aquaculture, Steelhead, Production Tracker
- **RLS**: Amazon-only inventory rows

### 3️⃣ **Finance & Budgeting**
Consolidates 2 models → GL + AP/AR + Budget data
- **Apps**: Team PSG, Buyer Metrics
- **RLS**: None (Finance sees all)

### 4️⃣ **Quality & Compliance**
Consolidates 4 models → VCQ + EHS + Airtable
- **Apps**: VCQ, Shellfish compliance reports
- **RLS**: None (Compliance sees all)

### 5️⃣ **HR & Workforce**
Consolidates 3 models → Payroll + WFM + Staffing
- **Apps**: HR KPIs, Shellfish Piece Rate
- **RLS**: None (HR sees all)

### 6️⃣ **Training & Development**
Consolidates 1 model → Course + STAR 360 + Certifications
- **Apps**: T&D KPIs
- **RLS**: None

---

## DATA FLOW: SOURCES → LAKEHOUSE → MODELS → APPS

```
SQL Server (7 DBs)    Excel (18 Files)    Airtable
        │                    │                │
        └────────┬───────────┴────────┬───────┘
                 │                    │
            Fabric Pipelines    Scheduled Ingestion
                 │                    │
        ┌────────┴────────────────────┴────────┐
        ▼                                       ▼
    BRONZE ZONE (Raw)  ◄─────┐
           │                  │ Zero Transformation
           ▼                  │
    SILVER ZONE (Cleaned) ◄───┘
           │
           ├──► 14 Shared Dimensions (MDM)
           │
           ▼
    GOLD ZONE (Optimized)
           │
           ├──► 6 Semantic Models
           │
           ▼
    Power BI Apps (17 total with RLS)
```

---

## DATA SOURCES SUMMARY

### **SQL Server (7 Databases - Daily)**
- AS400 (Sales, Customers, GL)
- SAP (Materials, Inventory, Products)  
- Mercatus (Feed, Production)
- Army CoE (Environmental)
- GoFormz (Mobile Forms)
- EHS Forms (Compliance)
- HR/Payroll/WFM (Workforce)

### **Excel (18 Files - Multiple Schedules)**
- **Hourly (2)**: Kroger IWI, Amazon Inventory
- **Daily (7)**: Shellfish Flow, Profitability, VCQ, Vena Budget
- **Twice Daily (8)**: Kroger files, Budgets, Forecasts
- **Weekly (1)**: Farm Summary

### **Airtable (1 Source)**
- Environmental Compliance (Permits, Violations, Enforcement)

---

## BENEFITS & SUCCESS METRICS

✅ **Consolidation**
- 71% model reduction (21→6)
- Unified Lakehouse (single source of truth)
- 14 shared dimensions (no duplication)

✅ **Performance**
- Query response: <5 seconds
- Hourly refresh: <60 min end-to-end
- Daily refresh: <2 hours end-to-end

✅ **Governance**
- 100% lineage tracking (Purview)
- Full metadata & data quality controls
- Complete audit trail from source to consumption

✅ **User Experience**
- All 17 Power BI apps unified
- Consistent data definitions
- RLS-based multi-tenant access

---

## 4-PHASE IMPLEMENTATION (16 Weeks)

| Phase | Focus | Timeline | Outcome |
|-------|-------|----------|---------|
| **1** | Lakehouse foundation + SQL ingestion | Weeks 1-4 | Bronze zone operational |
| **2** | Excel & Airtable integration | Weeks 5-8 | All 18 Excel files + dimensions |
| **3** | Model consolidation & migration | Weeks 9-12 | 6 models ready, RLS tested |
| **4** | Optimization & cutover | Weeks 13-16 | All apps live, training complete |

---

## NEXT STEPS

1. ✅ **Approve** this architecture
2. ✅ **Confirm** 6-model consolidation strategy
3. ✅ **Assign** data owners (1 per model)
4. ✅ **Validate** refresh SLAs with stakeholders
5. ✅ **Kick off** Phase 1 implementation
