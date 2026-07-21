# PSG Data Architecture Flowchart - Enhanced Lineage with Data Flow Types

```mermaid
flowchart TD
    subgraph Sources["📦 SOURCE SYSTEMS"]
        AS400["AS400 ERP<br/>(Operational Data)"]
        SAP["SAP<br/>(Batch & Materials)"]
        Mercatus["Mercatus<br/>(Feed/Production)"]
        ArmyCoE["Army CoE<br/>(Environmental)"]
        GoFormz["GoFormz<br/>(Mobile Forms)"]
        EHSForms["EHS Forms<br/>(Compliance Data)"]
        OtherSources["Other LOB Systems<br/>(HR, Payroll, WFM)"]
    end

    subgraph Ingestion["🔄 INGESTION & INTEGRATION"]
        Pipelines["Fabric Pipelines<br/>Data Factory ETL"]
        Dataflows["Power BI/Fabric<br/>Dataflows"]
        DirectConnect["Direct Connections<br/>(DirectQuery/Live)"]
    end

    subgraph Storage["💾 STORAGE & TRANSFORM"]
        Lakehouse["Fabric Lakehouse<br/>(OneLake Tables)"]
        Warehouse["Fabric Warehouse<br/>(SQL)"]
        ImportedModel["Imported Tables<br/>(Semantic Model)"]
    end

    subgraph Models["📊 SEMANTIC MODELS"]
        SalesModel["Sales & Margin"]
        HRModel["HR & Payroll"]
        InventoryModel["Inventory & Aged"]
        QCModel["Quality & Compliance<br/>(VCQ, PQI, WQI, DOR)"]
        AquacultureModel["Aquaculture & Steelhead"]
        WFMModel["Workforce Mgmt"]
    end

    subgraph Consumption["📈 CONSUMPTION & REPORTING"]
        PSGReports["PSG Reports App<br/>(Aquaculture, Distribution)"]
        PSGShellfish["PSG Shellfish App"]
        PSGVCQ["PSG VCQ App"]
        PSGAquaculture["PSG Aquaculture App"]
        PSGSales["PSG Sales App"]
        SteelheadApp["Steelhead Analysis"]
        TeamPSG["Team PSG App<br/>(Fiscal Calendar)"]
        StandaloneReports["Standalone Reports<br/>(e.g., Churn Dashboard)"]
    end

    %% SOURCE TO INGESTION - SPECIFIC CONNECTIONS
    AS400 -->|Batch| Pipelines
    SAP -->|Batch| Pipelines
    Mercatus -->|Batch| Pipelines
    ArmyCoE -->|Batch| Pipelines
    GoFormz -->|Scheduled| Dataflows
    EHSForms -->|Scheduled| Dataflows
    OtherSources -->|Batch| Pipelines

    %% INGESTION TO STORAGE - SPECIFIC CONNECTIONS
    Pipelines -->|Daily| Lakehouse
    Pipelines -->|Daily| Warehouse
    Dataflows -->|Scheduled| ImportedModel
    DirectConnect -->|Live/Real-time| Models

    %% STORAGE TO MODELS - SPECIFIC CONNECTIONS
    Lakehouse -->|Refresh| SalesModel
    Lakehouse -->|Refresh| InventoryModel
    Lakehouse -->|Refresh| AquacultureModel
    Warehouse -->|Refresh| QCModel
    Warehouse -->|Refresh| WFMModel
    ImportedModel -->|Refresh| HRModel

    %% MODELS TO CONSUMPTION - SPECIFIC CONNECTIONS
    SalesModel -->|Direct Query| PSGSales
    SalesModel -->|Direct Query| StandaloneReports
    
    InventoryModel -->|Direct Query| PSGReports
    
    AquacultureModel -->|Direct Query| PSGAquaculture
    AquacultureModel -->|Direct Query| PSGReports
    AquacultureModel -->|Direct Query| SteelheadApp
    
    QCModel -->|Direct Query| PSGVCQ
    QCModel -->|Direct Query| PSGReports
    
    WFMModel -->|Direct Query| PSGShellfish
    WFMModel -->|Direct Query| PSGReports
    
    HRModel -->|Direct Query| TeamPSG

    style Sources fill:#e1f5ff
    style Ingestion fill:#fff3e0
    style Storage fill:#f3e5f5
    style Models fill:#e8f5e9
    style Consumption fill:#fce4ec
```

## Legend

### Data Flow Types
- 🔵 **Batch**: Scheduled data transfer at set intervals (typically nightly)
- 🟡 **Scheduled**: Regular dataflow runs on a defined schedule
- 🟢 **Daily**: Data refreshes once per day
- 🟣 **Refresh**: Semantic model refresh cycle
- 🔴 **Direct Query**: Real-time query against source data (no caching)
- ⚪ **Live/Real-time**: Continuous or near-real-time data flow

### Color Coding by Layer
- 🔵 Blue: Source Systems
- 🟠 Orange: Ingestion & Integration
- 🟣 Purple: Storage & Transform
- 🟢 Green: Semantic Models
- 🔴 Pink: Consumption & Reporting

## Data Lineage & Flow Characteristics
- **Source → Ingestion**: Mostly batch/scheduled processes moving data from operational systems
- **Ingestion → Storage**: Batch daily loads to Lakehouse and Warehouse; scheduled dataflows to Imported Tables
- **Storage → Models**: Semantic models refresh from underlying storage layers
- **Models → Consumption**: Direct Query connections provide real-time/fresh data to reporting apps
- **Direct Query Path**: DirectConnect bypasses storage layers for real-time data access directly to models

## Key Observations
- **Real-time Reporting**: Apps use Direct Query connections for fresh data
- **Batch Processing**: Most source data flows through daily batch pipelines
- **Storage Consolidation**: Data distributed across 3 storage types (Lakehouse, Warehouse, Imported Tables)
- **Critical Dependencies**: AquacultureModel and PSGReports serve as central hubs feeding multiple downstream apps

## Next Steps for Architecture Review
- [ ] Identify data quality checkpoints needed between layers
- [ ] Map freshness/SLA requirements per app (confirm refresh frequencies align with business needs)
- [ ] Document cross-model data dependencies
- [ ] Identify consolidation opportunities among semantic models
- [ ] Plan "future state" architecture improvements
