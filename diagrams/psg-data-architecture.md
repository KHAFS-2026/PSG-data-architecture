# PSG Data Architecture Flowchart

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

    %% Source to Ingestion
    AS400 --> Pipelines
    SAP --> Pipelines
    Mercatus --> Pipelines
    ArmyCoE --> Pipelines
    GoFormz --> Pipelines
    EHSForms --> Pipelines
    OtherSources --> Pipelines
    
    Sources -.->|Alternative Path| Dataflows
    Sources -.->|Where Applicable| DirectConnect

    %% Ingestion to Storage
    Pipelines --> Lakehouse
    Pipelines --> Warehouse
    Dataflows --> Lakehouse
    Dataflows --> ImportedModel
    DirectConnect -.->|Live Connection| Models

    %% Storage to Models
    Lakehouse --> Models
    Warehouse --> Models
    ImportedModel --> Models

    %% Models to Consumption
    Models --> PSGReports
    Models --> PSGShellfish
    Models --> PSGVCQ
    Models --> PSGAquaculture
    Models --> PSGSales
    Models --> SteelheadApp
    Models --> TeamPSG
    Models --> StandaloneReports

Legend
Solid Lines: Primary data flow paths
Dotted Lines: Alternative or conditional paths
Color Coding:
🔵 Blue: Source Systems
🟠 Orange: Ingestion & Integration
🟣 Purple: Storage & Transform
🟢 Green: Semantic Models
🔴 Pink: Consumption & Reporting
Next Steps for Architecture Review
 Identify data quality checkpoints needed between layers
 Map freshness/SLA requirements per app
 Document cross-model data dependencies
 Identify consolidation opportunities among semantic models
 Plan "future state" architecture improvements
    style Sources fill:#e1f5ff
    style Ingestion fill:#fff3e0
    style Storage fill:#f3e5f5
    style Models fill:#e8f5e9
    style Consumption fill:#fce4ec
