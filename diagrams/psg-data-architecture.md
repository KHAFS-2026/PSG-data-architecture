# PSG Data Architecture Flowchart - Enhanced Lineage

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
    AS400 --> Pipelines
    SAP --> Pipelines
    Mercatus --> Pipelines
    ArmyCoE --> Pipelines
    GoFormz --> Dataflows
    EHSForms --> Dataflows
    OtherSources --> Pipelines

    %% INGESTION TO STORAGE - SPECIFIC CONNECTIONS
    Pipelines --> Lakehouse
    Pipelines --> Warehouse
    Dataflows --> ImportedModel
    DirectConnect --> Models

    %% STORAGE TO MODELS - SPECIFIC CONNECTIONS
    Lakehouse --> SalesModel
    Lakehouse --> InventoryModel
    Lakehouse --> AquacultureModel
    Warehouse --> QCModel
    Warehouse --> WFMModel
    ImportedModel --> HRModel

    %% MODELS TO CONSUMPTION - SPECIFIC CONNECTIONS
    SalesModel --> PSGSales
    SalesModel --> StandaloneReports
    
    InventoryModel --> PSGReports
    
    AquacultureModel --> PSGAquaculture
    AquacultureModel --> PSGReports
    AquacultureModel --> SteelheadApp
    
    QCModel --> PSGVCQ
    QCModel --> PSGReports
    
    WFMModel --> PSGShellfish
    WFMModel --> PSGReports
    
    HRModel --> TeamPSG

    style Sources fill:#e1f5ff
    style Ingestion fill:#fff3e0
    style Storage fill:#f3e5f5
    style Models fill:#e8f5e9
    style Consumption fill:#fce4ec
```

## Legend
- **Solid Lines**: Direct data flow/lineage from source to consumption
- **Color Coding**:
  - 🔵 Blue: Source Systems
  - 🟠 Orange: Ingestion & Integration
  - 🟣 Purple: Storage & Transform
  - 🟢 Green: Semantic Models
  - 🔴 Pink: Consumption & Reporting

## Data Lineage Notes
- **Direct Query/Live Connections**: Bypass storage layers, connect directly to models
- **Lakehouse**: Serves Sales, Inventory, and Aquaculture models
- **Warehouse (SQL)**: Serves Quality/Compliance and Workforce Management models
- **Imported Tables**: Serves HR & Payroll models

## Next Steps for Architecture Review
- [ ] Identify data quality checkpoints needed between layers
- [ ] Map freshness/SLA requirements per app
- [ ] Document cross-model data dependencies
- [ ] Identify consolidation opportunities among semantic models
- [ ] Plan "future state" architecture improvements
