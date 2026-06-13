# AnalystGPT ( Multimodal Financial Intelligence And Forecasting Platform )
Multimodal financial intelligence platform for forecasting revenue, EPS, earnings surprises, and risk using SEC filings, earnings calls, news, and market data.

```mermaid
flowchart TD
    %% ─── DATA SOURCES ───
    subgraph SRC["Data Sources"]
        S1["SEC Filings\n10-K · 10-Q"]
        S2["Earnings Calls\nTranscripts"]
        S3["Market Data\nOHLCV"]
        S4["News & Articles\nFinancial · Earnings"]
        S5["Financial Statements\nIncome · Balance · CF"]
    end

    %% ─── INGESTION LAYER ───
    subgraph ING["Ingestion Layer (Celery + Redis Beat)"]
        I1["SEC Filing Pipeline\nEDGAR Download → Parse\nChunk · Clean · Store"]
        I2["Earnings Call Pipeline\nSpeaker ID · Remarks\nQ&A Extraction"]
        I3["Market Data Pipeline\nOHLCV · Feature Gen\nHistorical Storage"]
        I4["News Pipeline\nArticle Ingest\nDedup · Tag"]
    end

    %% ─── NLP & FEATURE ENGINEERING ───
    subgraph PROC["Processing & Feature Engineering"]
        direction TB
        NLP["FinBERT NLP Pipeline\nSentiment · Confidence\nUncertainty · Risk Score"]
        EMB["Embeddings\nSentence Transformers\n384 / 768-d vectors"]
        MF["Market Features\nReturns · Volatility · RSI\nMACD · Beta · Drawdown · ATR"]
        FF["Financial Features\nRevenue Growth · EPS Growth\nMargins · Debt Ratio · ROE · ROA"]
        NF["NLP Features\nSentiment Score\nManagement Confidence\nUncertainty · Risk"]
        DQ["Data Quality & Validation\nMissing Values · Outliers\nConsistency · Lineage"]
        FSL["Feature Store Loader\nValidate & Store FeatureSet"]
    end

    %% ─── FEATURE STORE ───
    subgraph STORE["Feature Store & Data Layer"]
        PG[("PostgreSQL\nCompanies · Filings\nTranscripts · Market\nNews · Features\nForecasts")]
        VEC[("pgvector\nFiling Embeddings\nTranscript Embeddings\nNews Embeddings")]
        CACHE[("Redis Cache\nAPI Cache · Feature Cache\nRate Limiting")]
    end

    %% ─── ML LAYER ───
    subgraph ML["ML & Forecasting Layer"]
        direction TB
        subgraph FUSION["Multimodal Fusion Model"]
            TE["Tabular Encoder\nFinancial + Market Features"]
            TXE["Text Encoder\nFiling + Transcript + News Embeddings"]
            CA["Cross Attention & Fusion Layer"]
        end
        subgraph HEADS["Prediction Heads"]
            H1["Revenue Forecast"]
            H2["EPS Forecast"]
            H3["Earnings Surprise"]
            H4["Risk Score"]
            H5["Mgmt Confidence"]
        end
        subgraph TS["Time Series Models"]
            TS1["Linear Regression\nBaseline"]
            TS2["XGBoost / LightGBM\nGradient Boosting"]
            TS3["LSTM\nSequence Model"]
            TS4["PatchTST\nTransformer"]
            TS5["TFT\nTemporal Fusion"]
        end
        ENS["Ensemble / Model Selection\nBenchmark · Leaderboard"]
        subgraph XAI["Explainability"]
            SHAP["SHAP Values\nTop +/- Factors"]
            ATT["Attention Heatmaps\nFiling · Transcript · News"]
        end
    end

    %% ─── SERVING LAYER ───
    subgraph SERVE["Serving & Application Layer"]
        API["FastAPI Backend\nREST APIs · Auth\nCaching · WebSockets"]
        subgraph PAGES["React Frontend (TypeScript + Recharts)"]
            P1["Company Overview\nRevenue · EPS · Risk Trends"]
            P2["Forecasting\nRevenue · EPS · Surprise"]
            P3["Explainability\nSHAP · Attention Maps"]
            P4["Peer Comparison\nAAPL · MSFT · NVDA · AMZN · META"]
            P5["Research Benchmarks\nModel Leaderboard · Metrics"]
        end
    end

    %% ─── MLOPS ───
    subgraph OPS["🛠️ MLOps · Monitoring · Infrastructure"]
        MLF["MLflow\nExperiment Tracking\nModel Registry · Artifacts"]
        MON["Monitoring\nDrift Detection\nPerformance · Alerts"]
        INFRA["Docker Compose\nGitHub Actions CI/CD"]
    end

    %% ─── DATA FLOW EDGES ───
    S1 --> I1
    S2 --> I2
    S3 --> I3
    S4 --> I4
    S5 --> I3

    I1 --> NLP
    I2 --> NLP
    I4 --> NLP
    NLP --> EMB
    I3 --> MF
    I1 --> FF
    I2 --> FF
    NLP --> NF

    MF --> DQ
    FF --> DQ
    NF --> DQ
    EMB --> DQ
    DQ --> FSL

    FSL --> PG
    EMB --> VEC
    PG --> CACHE

    PG --> TE
    VEC --> TXE
    TE --> CA
    TXE --> CA
    CA --> H1 & H2 & H3 & H4 & H5

    PG --> TS1 & TS2 & TS3 & TS4 & TS5
    TS1 & TS2 & TS3 & TS4 & TS5 --> ENS

    CA --> SHAP & ATT
    ENS --> SHAP

    H1 & H2 & H3 & H4 & H5 --> API
    ENS --> API
    SHAP & ATT --> API
    CACHE --> API

    API --> P1 & P2 & P3 & P4 & P5

    PG --> MLF
    ENS --> MLF
    MLF --> MON
    INFRA --> OPS

    %% ─── STYLES ───
    classDef source fill:#dbeafe,stroke:#3b82f6,color:#1e3a8a
    classDef ingest fill:#ede9fe,stroke:#7c3aed,color:#3b0764
    classDef proc fill:#fef3c7,stroke:#d97706,color:#78350f
    classDef store fill:#dcfce7,stroke:#16a34a,color:#14532d
    classDef ml fill:#fce7f3,stroke:#db2777,color:#831843
    classDef serve fill:#ffedd5,stroke:#ea580c,color:#7c2d12
    classDef ops fill:#f1f5f9,stroke:#64748b,color:#1e293b

    class S1,S2,S3,S4,S5 source
    class I1,I2,I3,I4 ingest
    class NLP,EMB,MF,FF,NF,DQ,FSL proc
    class PG,VEC,CACHE store
    class TE,TXE,CA,H1,H2,H3,H4,H5,TS1,TS2,TS3,TS4,TS5,ENS,SHAP,ATT ml
    class API,P1,P2,P3,P4,P5 serve
    class MLF,MON,INFRA ops
```
