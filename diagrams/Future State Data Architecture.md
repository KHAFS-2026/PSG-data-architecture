# PSG Future State Data Architecture - Fabric-Native Design

## Executive Summary
This future state architecture implements Microsoft Fabric best practices to replace the legacy architecture. It provides a unified, scalable, and maintainable data platform with proper data governance, reduced redundancy, and built-in resilience.

---

```mermaid
flowchart TD
    subgraph Sources["📦 SOURCE SYSTEMS<br/>(Pre-Validated)"]
        AS400["AS400 ERP<br/>(Operational Data)"]
        SAP["SAP<br/>(Batch & Materials)"]
        Mercatus["Mercatus<br/>(Feed/Production)"]
        ArmyCoE["Army CoE<br/>(Environmental)"]
        GoFormz["GoFormz<br/>(Mobile Forms)"]
        EHSForms["EHS Forms<br/>(Compliance Data)"]
        OtherSources["Other LOB Systems<br/>(HR, Payroll, WFM)"]
    end

    subgraph Ingestion["🔄 UNIFIED INGESTION LAYER<br/>(Single Fabric Pipelines Strategy)"]
        PipelineMain["Fabric Data Factory Pipelines<br/>(Primary Path)"]
        PipelineBackup["Fabric Data Factory Pipelines<br/>(Backup/Failover Path)"]
        ValidationCheck["Data Validation Gate<br/>(Pre-Fabric)<br/>✅ Validated Data Only"]
    end

    subgraph Lakehouse["💾 UNIFIED LAKEHOUSE<br/>(OneLake - Single Source of Truth)"]
        Bronze["🔵 BRONZE ZONE<br/>(Raw Data)<br/>- Minimal transformation<br/>- Full audit trail<br/>- As-is from source"]
        Silver["🟢 SILVER ZONE<br/>(Cleaned & Deduplicated)<br/>- Business rules applied<br/>- Referential integrity<br/>- Ready for analytics"]
        Gold["🟡 GOLD ZONE<br/>(Aggregated & Optimized)<br/>- Denormalized for performance<br/>- Pre-calculated metrics<br/>- Report-ready"]
    end

    subgraph MDM["🎯 MASTER DATA MANAGEMENT LAYER<br/>(Unified Dimensions)"]
        DimCustomer["📋 Customer Master<br/>(Single version of truth)<br/>- Customer ID<br/>- Account hierarchy<br/>- Attributes"]
        DimProduct["📋 Product Master<br/>(Single version of truth)<br/>- Product ID<br/>- Category<br/>- Cost structure"]
        DimLocation["📋 Location Master<br/>(Single version of truth)<br/>- Facility/Site<br/>- Region<br/>- Operational data"]
        DimDate["📋 Date Dimension<br/>(Calendar)<br/>- Fiscal calendar<br/>- Standard calendar<br/>- Holiday flags"]
        DimEmployee["📋 Employee Master<br/>(Single version of truth)<br/>- Employee ID<br/>- Department<br/>- Role"]
    end

    subgraph Models["📊 SEMANTIC MODELS<br/>(Consolidated & Dimension-Shared)"]
        SalesModel["Sales & Revenue<br/>Fact: Transactions<br/>Dims: Customer, Product,<br/>Location, Date"]
        OpsModel["Operations<br/>Fact: Inventory, Production<br/>Dims: Product, Location,<br/>Date, Facility"]
        QCModel["Quality & Compliance<br/>Fact: Quality metrics, VCQ, PQI<br/>Dims: Location, Date,<br/>Employee"]
        HRModel["HR & Workforce<br/>Fact: Payroll, Headcount<br/>Dims: Employee, Date,<br/>Location"]
    end

    subgraph Consumption["📈 CONSUMPTION LAYER<br/>(Report & Analytics Apps)"]
        PSGReports["PSG Reports App"]
        PSGSales["PSG Sales App"]
        PSGAquaculture["PSG Aquaculture App"]
        PSGVCQ["PSG VCQ App"]
        PSGShellfish["PSG Shellfish App"]
        SteelheadApp["Steelhead Analysis"]
        TeamPSG["Team PSG App"]
        StandaloneReports["Standalone Reports"]
    end

    subgraph Governance["🛡️ GOVERNANCE & METADATA LAYER"]
        DataCatalog["Data Catalog<br/>(Purview Integration)<br/>- Data lineage<br/>- Data ownership<br/>- Sensitivity tags"]
        MonitoringAlerts["Monitoring & Alerts<br/>- Pipeline health<br/>- Data freshness<br/>- Quality metrics"]
    end

    %% SOURCE TO INGESTION
    AS400 --> ValidationCheck
    SAP --> ValidationCheck
    Mercatus --> ValidationCheck
    ArmyCoE --> ValidationCheck
    GoFormz --> ValidationCheck
    EHSForms --> ValidationCheck
    OtherSources --> ValidationCheck

    ValidationCheck --> PipelineMain
    ValidationCheck -.->|Failover| PipelineBackup

    %% INGESTION TO LAKEHOUSE ZONES
    PipelineMain -->|Daily Batch| Bronze
    PipelineBackup -->|Failover| Bronze
    
    %% BRONZE TO SILVER TRANSFORMATIONS
    Bronze -->|Deduplication<br/>Business Rules<br/>Cleaning| Silver
    
    %% SILVER TO GOLD AGGREGATIONS
    Silver -->|Aggregations<br/>Pre-calculations<br/>Denormalization| Gold
    
    %% SILVER TO MDM
    Silver -->|Customer Data| DimCustomer
    Silver -->|Product Data| DimProduct
    Silver -->|Location Data| DimLocation
    Silver -->|Employee Data| DimEmployee
    
    %% MDM TO MODELS (Shared Dimensions)
    Gold -->|Facts| SalesModel
    DimCustomer --> SalesModel
    DimProduct --> SalesModel
    DimLocation --> SalesModel
    DimDate --> SalesModel
    
    Gold -->|Facts| OpsModel
    DimProduct --> OpsModel
    DimLocation --> OpsModel
    DimDate --> OpsModel
    
    Gold -->|Facts| QCModel
    DimLocation --> QCModel
    DimDate --> QCModel
    DimEmployee --> QCModel
    
    Gold -->|Facts| HRModel
    DimEmployee --> HRModel
    DimDate --> HRModel
    DimLocation --> HRModel
    
    %% MODELS TO CONSUMPTION
    SalesModel -->|Direct Query| PSGSales
    SalesModel -->|Direct Query| PSGReports
    
    OpsModel -->|Direct Query| PSGAquaculture
    OpsModel -->|Direct Query| PSGReports
    OpsModel -->|Direct Query| SteelheadApp
    
    QCModel -->|Direct Query| PSGVCQ
    QCModel -->|Direct Query| PSGReports
    
    HRModel -->|Direct Query| PSGShellfish
    HRModel -->|Direct Query| TeamPSG
    HRModel -->|Direct Query| PSGReports
    
    SalesModel -->|Direct Query| StandaloneReports
    
    %% GOVERNANCE CONNECTIONS
    Bronze -.->|Metadata| DataCatalog
    Silver -.->|Lineage| DataCatalog
    Gold -.->|Ownership| DataCatalog
    Models -.->|Business Rules| DataCatalog
    
    PipelineMain -.->|Health| MonitoringAlerts
    PipelineBackup -.->|Health| MonitoringAlerts

    style Sources fill:#e1f5ff
    style Ingestion fill:#fff3e0
    style Lakehouse fill:#f3e5f5
    style MDM fill:#c8e6c9
    style Models fill:#e8f5e9
    style Consumption fill:#fce4ec
    style Governance fill:#ede7f6
