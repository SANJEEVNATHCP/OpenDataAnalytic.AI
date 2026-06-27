# System Architecture Diagrams

## High-Level Architecture

```mermaid
graph TB
    User["👤 User"]
    Desktop["🖥️ Desktop App<br/>Tauri + React"]
    API["🔌 API Server<br/>FastAPI"]
    DB["💾 Database<br/>SQLite/PostgreSQL"]
    AI["🧠 AI Services<br/>LLMs + Ollama"]
    
    User -->|IPC/HTTP| Desktop
    Desktop -->|HTTP| API
    API -->|Query| DB
    API -->|Inference| AI
    API -->|Store| DB
```

## Authentication Flow

```mermaid
sequenceDiagram
    participant User
    participant Client as Client App
    participant API as API Server
    participant AuthDB as Auth DB
    
    User->>Client: Enter Credentials
    Client->>API: POST /auth/token
    API->>AuthDB: Verify Credentials
    AuthDB-->>API: Password Hash Match
    API-->>Client: JWT Token
    Client->>Client: Store Token
    Client->>API: GET /projects (Auth: Bearer JWT)
    API->>API: Verify JWT
    API-->>Client: Projects List
```

## Data Processing Pipeline

```mermaid
graph LR
    Input["Raw Data"] -->|Upload| Storage["Storage"]
    Storage -->|Load| Profile["Profile AI"]
    Profile -->|Analysis| Quality["Quality Check"]
    Quality -->|Suggestions| Clean["Cleaning AI"]
    Clean -->|Transform| Processed["Clean Data"]
    Processed -->|Store| DB["Database"]
    DB -->|Query| Analysis["Analysis"]
    Analysis -->|Recommend| Viz["Visualization AI"]
    Viz -->|Display| Dashboard["Dashboard"]
```

## Component Interaction Diagram

```mermaid
graph TB
    UI["🎨 Frontend<br/>React Components"]
    State["📊 State Management<br/>Redux/Context"]
    API["🔌 API Client"]
    Backend["🔧 FastAPI Backend"]
    Services["📦 Services<br/>Business Logic"]
    Agents["🤖 AI Agents<br/>LLM Integration"]
    DB["💾 Data Layer<br/>SQLAlchemy"]
    
    UI -->|Dispatch| State
    State -->|Call| API
    API -->|HTTP| Backend
    Backend -->|Route| Services
    Services -->|Delegate| Agents
    Services -->|Query| DB
    Agents -->|LLM Call| LLM["Claude/Ollama"]
```

## Deployment Architecture

```mermaid
graph TB
    subgraph Desktop["Desktop Deployment"]
        DApp["Tauri App<br/>Windows/Mac/Linux"]
        DDB["Local SQLite"]
        DPython["Python Runtime"]
        DApp -->|IPC| DPython
        DPython -->|File I/O| DDB
    end
    
    subgraph Cloud["Cloud Deployment"]
        LB["Load Balancer"]
        API1["API Server 1"]
        API2["API Server 2"]
        API3["API Server 3"]
        Cache["Redis Cache"]
        DB["PostgreSQL"]
        AI["AI Service Cluster"]
        
        LB -->|Route| API1
        LB -->|Route| API2
        LB -->|Route| API3
        API1 -->|Cache| Cache
        API2 -->|Cache| Cache
        API3 -->|Cache| Cache
        API1 -->|Query| DB
        API2 -->|Query| DB
        API3 -->|Query| DB
        API1 -->|Infer| AI
        API2 -->|Infer| AI
        API3 -->|Infer| AI
    end
```

## User Role Access Control

```mermaid
graph TB
    User["User"]
    Auth["Authentication<br/>OAuth/Email"]
    GetRole["Get User Role"]
    
    subgraph Access["Role-Based Access"]
        Owner["Owner<br/>✅ Full Control"]
        Editor["Editor<br/>✅ Create/Edit"]
        Analyst["Analyst<br/>✅ Create Analyses"]
        Viewer["Viewer<br/>✅ Read Only"]
        Commenter["Commenter<br/>✅ Comments Only"]
    end
    
    User -->|Login| Auth
    Auth -->|Verified| GetRole
    GetRole -->|Assign| Owner
    GetRole -->|Assign| Editor
    GetRole -->|Assign| Analyst
    GetRole -->|Assign| Viewer
    GetRole -->|Assign| Commenter
    
    Owner -->|Access| AllFeatures["All Features"]
    Editor -->|Access| EditFeatures["Edit Features"]
    Analyst -->|Access| AnalysisFeatures["Analysis Features"]
    Viewer -->|Access| ViewFeatures["View Features"]
    Commenter -->|Access| CommentFeatures["Comment Features"]
```

## AI Agent Architecture

```mermaid
graph TB
    Input["User Input"]
    Router["Agent Router"]
    
    subgraph Agents["AI Agents"]
        PA["Data Profiling<br/>AI"]
        CA["Cleaning<br/>AI"]
        QA["Query<br/>AI"]
        VA["Visualization<br/>AI"]
        IA["Insight<br/>AI"]
        RA["Reporting<br/>AI"]
        KA["Knowledge<br/>AI"]
        VALA["Validation<br/>AI"]
    end
    
    LLM["LLM Model<br/>Claude/Ollama"]
    Validator["Validator"]
    Score["Confidence<br/>Scorer"]
    Output["Human Review<br/>& Approval"]
    
    Input -->|Analyze| Router
    Router -->|Route| PA
    Router -->|Route| CA
    Router -->|Route| QA
    Router -->|Route| VA
    Router -->|Route| IA
    Router -->|Route| RA
    Router -->|Route| KA
    Router -->|Route| VALA
    
    PA -->|Prompt| LLM
    CA -->|Prompt| LLM
    QA -->|Prompt| LLM
    VA -->|Prompt| LLM
    IA -->|Prompt| LLM
    RA -->|Prompt| LLM
    KA -->|Prompt| LLM
    VALA -->|Prompt| LLM
    
    LLM -->|Response| Validator
    Validator -->|Valid| Score
    Score -->|Score| Output
```

## Data Flow for Analysis

```mermaid
graph LR
    User["User"] -->|"1. Ask Question"| NLQuery["Natural Language<br/>Query Input"]
    NLQuery -->|"2. Process"| QueryAI["Query AI<br/>Agent"]
    QueryAI -->|"3. Generate"| SQL["Generated<br/>SQL"]
    SQL -->|"4. Execute"| DB["Database"]
    DB -->|"5. Results"| Parser["Result<br/>Parser"]
    Parser -->|"6. Recommend"| VizAI["Visualization<br/>AI"]
    VizAI -->|"7. Display"| Viz["Interactive<br/>Visualization"]
    Viz -->|"8. Analyze"| InsightAI["Insight<br/>AI"]
    InsightAI -->|"9. Generate"| Insights["Business<br/>Insights"]
    Insights -->|"10. Display"| Dashboard["Dashboard<br/>Widget"]
```

## Deployment Pipeline

```mermaid
graph LR
    Code["Code<br/>Commit"]
    CI["CI Pipeline<br/>GitHub Actions"]
    Test["Run Tests<br/>Unit/Integration"]
    Build["Build<br/>Artifacts"]
    Staging["Deploy to<br/>Staging"]
    SmokeTest["Smoke Tests"]
    Prod["Deploy to<br/>Production"]
    Monitor["Monitor<br/>& Alert"]
    
    Code -->|Trigger| CI
    CI -->|Run| Test
    Test -->|Pass| Build
    Build -->|Package| Staging
    Staging -->|Run| SmokeTest
    SmokeTest -->|Pass| Prod
    Prod -->|Track| Monitor
```

## Security Layers

```mermaid
graph TB
    subgraph Security["Security Architecture"]
        Network["🌐 Network Layer<br/>TLS, DDoS Protection"]
        Auth["🔐 Authentication<br/>OAuth, JWT, MFA"]
        AuthZ["👤 Authorization<br/>RBAC, ABAC"]
        App["🛡️ Application<br/>Input Validation, SQL Prevention"]
        Data["💾 Data Layer<br/>Encryption, Access Control"]
    end
    
    User["👤 User"]
    Database["📊 Database"]
    
    User -->|HTTPS| Network
    Network -->|Verify| Auth
    Auth -->|Grant| AuthZ
    AuthZ -->|Allow| App
    App -->|Execute Safely| Data
    Data -->|Store Securely| Database
```

## Workflow: Creating an Analysis

```mermaid
graph TD
    A["1. Create Project"]
    B["2. Upload Dataset"]
    C["3. AI Profiles Data"]
    D["4. Review Profile"]
    E["5. Ask Question"]
    F["6. Query AI Generates SQL"]
    G["7. Execute Query"]
    H["8. Get Results"]
    I["9. Viz AI Recommends Chart"]
    J["10. Select Chart"]
    K["11. Insight AI Analyzes"]
    L["12. Display Insights"]
    M["13. Add to Dashboard"]
    
    A -->|Create| B
    B -->|Upload| C
    C -->|Complete| D
    D -->|Approve| E
    E -->|Submit| F
    F -->|Generate| G
    G -->|Execute| H
    H -->|Provide| I
    I -->|Suggest| J
    J -->|Select| K
    K -->|Analyze| L
    L -->|Display| M
    M -->|Save| N["✅ Analysis Complete"]
```

## Database Schema Relationships

```mermaid
erDiagram
    USERS ||--o{ PROJECTS : owns
    USERS ||--o{ API_KEYS : has
    USERS ||--o{ AUDIT_LOGS : generates
    
    PROJECTS ||--o{ DATASETS : contains
    PROJECTS ||--o{ ANALYSES : contains
    PROJECTS ||--o{ VISUALIZATIONS : contains
    PROJECTS ||--o{ DASHBOARDS : contains
    PROJECTS ||--o{ REPORTS : contains
    PROJECTS ||--o{ COLLABORATORS : has
    
    DATASETS ||--o{ DATASET_COLUMNS : has
    DATASETS ||--o{ DATA_QUALITY_ISSUES : contains
    DATASETS ||--o{ DATASET_VERSIONS : tracks
    
    ANALYSES ||--o{ VISUALIZATIONS : generates
    
    DASHBOARDS ||--o{ DASHBOARD_WIDGETS : contains
    DASHBOARD_WIDGETS ||--o{ VISUALIZATIONS : displays
    
    USERS ||--o{ COMMENTS : makes
    PROJECTS ||--o{ COMMENTS : receives
```

## Feature Adoption Timeline

```mermaid
gantt
    title Feature Roadmap Timeline
    
    section v1.0
    Core Features :active, v1, 2024-12-15, 90d
    AI Agents :crit, v1, 2024-12-15, 90d
    Desktop App :v1, 2024-12-15, 90d
    
    section v1.1
    Forecasting AI :v11, 2025-03-15, 60d
    Performance :v11, 2025-03-15, 60d
    Mobile :v11, 2025-03-15, 60d
    
    section v1.2
    Governance :v12, 2025-06-15, 90d
    Enterprise :v12, 2025-06-15, 90d
    Marketplace :v12, 2025-06-15, 90d
    
    section v2.0
    Cloud :v2, 2025-09-15, 120d
    Multi-tenant :v2, 2025-09-15, 120d
    Advanced ML :v2, 2025-09-15, 120d
```

---

**Last Updated**: December 2024
