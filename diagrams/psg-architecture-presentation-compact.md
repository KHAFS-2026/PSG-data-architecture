# PSG Future State Data Architecture - Presentation Ready (Compact)
## Consolidated 21 Models → 6 | Unified Lakehouse Platform

> Download the presentation-ready SVG version here: [psg-architecture-presentation-compact.svg](./psg-architecture-presentation-compact.svg)

---

```mermaid
flowchart TB
    subgraph Sources["📊 DATA SOURCES"]
        direction TB
        SRC_NOTE["7 SQL DBs • 18 Excel Files • Airtable"]
        SQL["SQL Sources<br/>Daily Batch"]
        EXCEL["Excel Sources<br/>Multiple Schedules"]
        AIRTABLE["Airtable<br/>Environmental Data"]
        SRC_NOTE --> SQL
        SRC_NOTE --> EXCEL
        SRC_NOTE --> AIRTABLE
    end

    subgraph Ingestion["🔄 INGESTION PIPELINES"]
        direction TB
        INGEST_NOTE["Fabric Data Factory"]
        INGEST_ROW[" "]
        PipelineSQL["SQL Pipeline"]
        PipelineEXCEL["Excel Pipelines"]
        PipelineAIR["Airtable Pipeline"]
        INGEST_NOTE --> INGEST_ROW
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

    subgraph MDM["🎯 MASTER DATA LAYER"]
        direction TB
        MDM_NOTE["14 Shared Dimensions"]
        DIMS["Customer • Product • Location<br/>Date • Employee<br/>+ 9 Excel-based Dimensions"]
        MDM_NOTE --> DIMS
    end

    subgraph Models["📊 SEMANTIC MODELS"]
        direction TB
        MODELS_NOTE["6 total, down from 21"]
        M1["1. Sales & Revenue"]
        M2["2. Operations"]
        M3["3. Finance"]
        M4["4. Quality"]
        M5["5. HR / Workforce"]
        M6["6. Training"]
        MODELS_NOTE --> M1
        MODELS_NOTE --> M2
        MODELS_NOTE --> M3
        MODELS_NOTE --> M4
        MODELS_NOTE --> M5
        MODELS_NOTE --> M6
    end

    subgraph Apps["📈 POWER BI APPS"]
        direction TB
        APPS_NOTE["17 total with RLS"]
        APPS["PSG Sales • PSG Reports • Kroger QA<br/>PSG Aquaculture • VCQ • HR KPIs<br/>+ 11 more apps"]
        APPS_NOTE --> APPS
    end

    subgraph GOV["🛡️ GOVERNANCE"]
        direction TB
        GOV_NOTE["Purview • Monitoring • RLS"]
        gov["Full Lineage<br/>and Metadata"]
        GOV_NOTE --> gov
    end

    %% MAIN FLOW
    SQL --> PipelineSQL
    EXCEL --> PipelineEXCEL
    AIRTABLE --> PipelineAIR

    PipelineSQL --> Bronze
    PipelineEXCEL --> Bronze
    PipelineAIR --> Bronze

    Bronze --> Silver
    Silver --> Gold
    Silver --> DIMS

    Gold --> M1
    Gold --> M2
    Gold --> M3
    Gold --> M4
    Gold --> M5
    Gold --> M6

    DIMS --> M1
    DIMS --> M2
    DIMS --> M3
    DIMS --> M4
    DIMS --> M5
    DIMS --> M6

    M1 --> APPS
    M2 --> APPS
    M3 --> APPS
    M4 --> APPS
    M5 --> APPS
    M6 --> APPS

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
