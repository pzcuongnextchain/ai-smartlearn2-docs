# AI Tool Configuration Flows and Diagrams

## User Flow Diagrams

### API Key Management Flow
```mermaid
flowchart TD
    A[Sub-Admin Access AI Tool Config] --> B[API Key Management Dashboard]
    B --> C{Select Action}
    
    C -->|Add New API Key| D[API Key Registration Form]
    C -->|Update Existing Key| E[Select Key to Update]
    C -->|Test API Key| F[API Key Testing Interface]
    C -->|Monitor Usage| G[Usage Analytics Dashboard]
    
    D --> H[Enter API Key Details]
    H --> I[Select AI Service Provider]
    I --> J[Configure Key Permissions]
    J --> K[Test Key Validity]
    K --> L{Key Valid?}
    
    L -->|No| M[Show Validation Error]
    L -->|Yes| N[Encrypt and Store Key]
    
    M --> H
    N --> O[Enable Associated Tools]
    O --> P[Update Tool Availability]
    P --> Q[Notify System of Changes]
    
    E --> R[Load Current Key Details]
    R --> S[Modify Key Settings]
    S --> T[Re-validate Key]
    T --> U{Still Valid?}
    U -->|No| V[Disable Associated Tools]
    U -->|Yes| W[Update Key Configuration]
    
    V --> X[Notify Users of Disruption]
    W --> Y[Propagate Changes]
    
    F --> Z[Select Key to Test]
    Z --> AA[Send Test Request]
    AA --> BB[Display Test Results]
    
    G --> CC[Load Usage Statistics]
    CC --> DD[Display Cost Analytics]
    DD --> EE[Show Usage Trends]
```

### Session-Specific Tool Configuration Flow
```mermaid
flowchart TD
    A[Access Session Configuration] --> B[Select Session Type]
    B --> C{Session Category}
    
    C -->|Practice Session 1-40| D[Practice Session Config]
    C -->|Assignment 1-10| E[Assignment Config]
    
    D --> F[Load Available AI Tools]
    E --> F
    F --> G[Filter by Master Control]
    G --> H[Filter by API Key Status]
    H --> I[Display Selectable Tools]
    I --> J[Multi-Select Tool Interface]
    
    J --> K[Configure Tool Settings]
    K --> L[Set Usage Limits]
    L --> M[Define Tool Restrictions]
    M --> N[Preview Student Experience]
    N --> O{Configuration Valid?}
    
    O -->|No| P[Show Configuration Errors]
    O -->|Yes| Q[Save Session Configuration]
    
    P --> K
    Q --> R[Generate Session URL]
    R --> S[Update Session Database]
    S --> T[Notify Affected Students]
    T --> U[Configuration Complete]
```



## Sequence Diagrams

### API Key Registration and Validation Sequence
```mermaid
sequenceDiagram
    participant SA as Sub-Administrator
    participant UI as Config Interface
    participant KeyMgr as Key Manager
    participant Validator as API Validator
    participant Encryptor as Encryption Service
    participant ToolMgr as Tool Manager
    
    SA->>UI: Register new API key
    UI->>KeyMgr: Process key registration
    KeyMgr->>Validator: Validate key with provider
    Validator->>Validator: Test API connectivity
    Validator->>KeyMgr: Return validation result
    
    alt Key Valid
        KeyMgr->>Encryptor: Encrypt API key
        Encryptor->>KeyMgr: Return encrypted key
        KeyMgr->>ToolMgr: Enable associated tools
        ToolMgr->>UI: Update tool availability
        UI->>SA: Show success confirmation
    else Key Invalid
        KeyMgr->>UI: Return validation error
        UI->>SA: Show error message
    end
```

### Tool Configuration Propagation Sequence
```mermaid
sequenceDiagram
    participant SA as Sub-Administrator
    participant Config as Tool Config Service
    participant Master as Master Control
    participant Session as Session Manager
    participant Student as Student Interface
    participant Logger as Audit Logger
    
    SA->>Config: Configure tools for session
    Config->>Master: Check master control permissions
    Master->>Config: Return allowed tools
    Config->>Session: Update session tool list
    Session->>Student: Propagate tool changes
    Student->>Session: Confirm tools updated
    Session->>Config: Report update success
    Config->>Logger: Log configuration change
    Logger->>SA: Confirm change logged
```

### Cost Monitoring and Alert Sequence
```mermaid
sequenceDiagram
    participant Monitor as Usage Monitor
    participant Calculator as Cost Calculator
    participant AlertMgr as Alert Manager
    participant SA as Sub-Administrator
    participant ToolMgr as Tool Manager
    
    Monitor->>Calculator: Report API usage
    Calculator->>Calculator: Calculate current costs
    Calculator->>AlertMgr: Check budget thresholds
    
    alt Budget Exceeded
        AlertMgr->>SA: Send critical alert
        AlertMgr->>ToolMgr: Suggest restrictions
        ToolMgr->>SA: Show restriction options
    else Approaching Limit
        AlertMgr->>SA: Send warning alert
        AlertMgr->>SA: Show optimization suggestions
    else Normal Usage
        AlertMgr->>SA: Send usage summary
    end
```

## State Diagrams

### API Key Lifecycle States
```mermaid
stateDiagram-v2
    [*] --> Registered: Key Added
    Registered --> Validating: Test Key
    Validating --> Active: Validation Success
    Validating --> Invalid: Validation Failed
    
    Invalid --> Registered: Update Key
    Active --> InUse: Tools Enabled
    InUse --> Active: Usage Complete
    
    Active --> Expired: Key Expires
    InUse --> RateLimited: Usage Limit Hit
    RateLimited --> InUse: Limit Reset
    
    Expired --> Updated: Renew Key
    Updated --> Validating: Re-validate
    
    Active --> Revoked: Manual Revocation
    InUse --> Revoked: Security Issue
    Revoked --> [*]: Key Deleted
    Expired --> [*]: Key Purged
```

### Tool Configuration States
```mermaid
stateDiagram-v2
    [*] --> Available: Tool in Master Control
    Available --> Configurable: API Key Valid
    Configurable --> Configured: Session Setup
    Configured --> Active: Session Published
    
    Active --> InUse: Students Using
    InUse --> Active: Usage Complete
    
    Configured --> Modifying: Edit Configuration
    Modifying --> Configured: Save Changes
    
    Active --> Disabled: API Key Failed
    InUse --> Disabled: Emergency Disable
    Disabled --> Available: Issue Resolved
    
    Available --> Unavailable: Master Control Off
    Unavailable --> Available: Master Control On
    
    Disabled --> [*]: Tool Removed
    Unavailable --> [*]: Tool Deprecated
```

### Cost Management States
```mermaid
stateDiagram-v2
    [*] --> Monitoring: Start Tracking
    Monitoring --> Normal: Under Budget
    Monitoring --> Warning: 80% Budget Used
    Monitoring --> Critical: Budget Exceeded
    
    Normal --> Warning: Usage Increases
    Warning --> Normal: Usage Decreases
    Warning --> Critical: Continued Usage
    
    Critical --> Restricted: Auto-Restrict Tools
    Critical --> Manual: Manual Intervention
    
    Restricted --> Warning: Usage Reduced
    Manual --> Normal: Budget Increased
    Manual --> Restricted: Apply Restrictions
    
    Normal --> [*]: End Billing Period
    Warning --> [*]: End Billing Period
    Critical --> [*]: End Billing Period
    Restricted --> [*]: End Billing Period
```

## Activity Diagrams

### Daily AI Tool Management Workflow
```mermaid
flowchart TD
    Start([Start Daily Tool Management]) --> CheckKeys[Check API Key Status]
    CheckKeys --> ValidateKeys[Validate All Keys]
    ValidateKeys --> KeyIssues{Key Issues Found?}
    
    KeyIssues -->|Yes| IdentifyProblems[Identify Problem Keys]
    KeyIssues -->|No| MonitorUsage[Monitor Tool Usage]
    
    IdentifyProblems --> CategorizeIssues[Categorize Issues]
    CategorizeIssues --> ExpiredKeys{Expired Keys?}
    ExpiredKeys -->|Yes| RenewKeys[Renew Expired Keys]
    ExpiredKeys -->|No| InvalidKeys{Invalid Keys?}
    
    InvalidKeys -->|Yes| UpdateKeys[Update Invalid Keys]
    InvalidKeys -->|No| RateLimited{Rate Limited?}
    
    RateLimited -->|Yes| AdjustLimits[Adjust Usage Limits]
    RateLimited -->|No| MonitorUsage
    
    RenewKeys --> TestRenewed[Test Renewed Keys]
    UpdateKeys --> TestUpdated[Test Updated Keys]
    AdjustLimits --> MonitorUsage
    
    TestRenewed --> MonitorUsage
    TestUpdated --> MonitorUsage
    
    MonitorUsage --> AnalyzeCosts[Analyze Cost Trends]
    AnalyzeCosts --> BudgetCheck{Budget Status OK?}
    
    BudgetCheck -->|Yes| OptimizeUsage[Optimize Tool Usage]
    BudgetCheck -->|No| ImplementRestrictions[Implement Cost Controls]
    
    OptimizeUsage --> GenerateReports[Generate Usage Reports]
    ImplementRestrictions --> NotifyUsers[Notify Affected Users]
    NotifyUsers --> GenerateReports
    
    GenerateReports --> PlanImprovements[Plan System Improvements]
    PlanImprovements --> End([Complete Daily Management])
```

### Tool Configuration Optimization Workflow
```mermaid
flowchart TD
    OptimizationStart([Start Tool Optimization]) --> AnalyzeUsage[Analyze Current Tool Usage]
    AnalyzeUsage --> IdentifyPatterns[Identify Usage Patterns]
    IdentifyPatterns --> UnderutilizedTools{Underutilized Tools?}
    
    UnderutilizedTools -->|Yes| ReviewUnderutilized[Review Underutilized Tools]
    UnderutilizedTools -->|No| OverusedTools{Overused Tools?}
    
    ReviewUnderutilized --> ConsiderRemoval[Consider Tool Removal]
    ConsiderRemoval --> TestWithoutTool[Test Sessions Without Tool]
    TestWithoutTool --> RemovalImpact{Negative Impact?}
    
    RemovalImpact -->|Yes| KeepTool[Keep Tool Active]
    RemovalImpact -->|No| RemoveTool[Remove from Sessions]
    
    OverusedTools -->|Yes| ReviewOverused[Review Overused Tools]
    OverusedTools -->|No| CostAnalysis[Analyze Cost Efficiency]
    
    ReviewOverused --> ScaleResources[Scale Tool Resources]
    ScaleResources --> MonitorPerformance[Monitor Performance Impact]
    
    KeepTool --> CostAnalysis
    RemoveTool --> CostAnalysis
    MonitorPerformance --> CostAnalysis
    
    CostAnalysis --> OptimizeCosts[Optimize Cost Structure]
    OptimizeCosts --> ValidateChanges[Validate All Changes]
    ValidateChanges --> DeployOptimizations[Deploy Optimizations]
    DeployOptimizations --> MonitorResults[Monitor Optimization Results]
    MonitorResults --> Complete([Optimization Complete])
```

## Use Case Diagrams

### Sub-Administrator AI Tool Configuration Use Cases
```mermaid
graph LR
    SubAdmin((Sub-Administrator))
    
    SubAdmin --> UC1[Manage API Keys]
    SubAdmin --> UC2[Configure Session Tools]
    SubAdmin --> UC3[Monitor Tool Usage]
    SubAdmin --> UC4[Control Tool Costs]
    SubAdmin --> UC5[Test Tool Functionality]
    SubAdmin --> UC6[Optimize Tool Selection]
    SubAdmin --> UC7[Generate Usage Reports]
    
    UC1 --> KeyMgmtSystem[Key Management System]
    UC2 --> ToolConfigSystem[Tool Configuration System]
    UC3 --> UsageMonitor[Usage Monitoring System]
    UC4 --> CostMgmtSystem[Cost Management System]
    UC5 --> TestingSystem[Testing System]
    UC6 --> OptimizationSystem[Optimization System]
    UC7 --> ReportingSystem[Reporting System]
```

### AI Tool Integration Use Cases
```mermaid
graph TB
    ToolSystem((AI Tool System))
    
    ToolSystem --> UC1[Validate API Keys]
    ToolSystem --> UC2[Route Tool Requests]
    ToolSystem --> UC3[Monitor Tool Performance]
    ToolSystem --> UC4[Handle Tool Failures]
    ToolSystem --> UC5[Track Usage Costs]
    ToolSystem --> UC6[Enforce Usage Limits]
    
    UC1 --> ValidationService[Validation Service]
    UC2 --> RoutingService[Routing Service]
    UC3 --> MonitoringService[Monitoring Service]
    UC4 --> ErrorHandler[Error Handler]
    UC5 --> CostTracker[Cost Tracker]
    UC6 --> LimitEnforcer[Limit Enforcer]
```

## Component Interaction Diagrams

### AI Tool Configuration Architecture
```mermaid
graph TB
    subgraph "Configuration Management Layer"
        KeyManager[API Key Manager]
        ToolConfigurator[Tool Configurator]
        SessionMapper[Session Mapper]
        CostController[Cost Controller]
    end
    
    subgraph "Validation and Security Layer"
        KeyValidator[Key Validator]
        Encryptor[Encryption Service]
        AccessController[Access Controller]
        AuditLogger[Audit Logger]
    end
    
    subgraph "Tool Integration Layer"
        EdenAIConnector[Eden AI Connector]
        CustomAPIConnector[Custom API Connector]
        ToolRouter[Tool Router]
        ResponseProcessor[Response Processor]
    end
    
    subgraph "Data Storage Layer"
        KeyStorage[(Key Storage)]
        ConfigDB[(Configuration Database)]
        UsageDB[(Usage Database)]
        CostDB[(Cost Database)]
    end
    
    KeyManager --> KeyValidator
    ToolConfigurator --> AccessController
    SessionMapper --> ToolRouter
    CostController --> AuditLogger
    
    KeyValidator --> Encryptor
    AccessController --> EdenAIConnector
    ToolRouter --> CustomAPIConnector
    AuditLogger --> ResponseProcessor
    
    Encryptor --> KeyStorage
    EdenAIConnector --> ConfigDB
    CustomAPIConnector --> UsageDB
    ResponseProcessor --> CostDB
```

### Real-time Tool Management Flow
```mermaid
graph LR
    subgraph "Configuration Changes"
        ConfigUpdate[Configuration Update]
        KeyUpdate[Key Update]
        ToolUpdate[Tool Update]
    end
    
    subgraph "Validation Pipeline"
        Validator[Validator]
        SecurityCheck[Security Check]
        CompatibilityCheck[Compatibility Check]
    end
    
    subgraph "Propagation System"
        SessionUpdater[Session Updater]
        StudentNotifier[Student Notifier]
        CacheInvalidator[Cache Invalidator]
    end
    
    ConfigUpdate --> Validator
    KeyUpdate --> SecurityCheck
    ToolUpdate --> CompatibilityCheck
    
    Validator --> SessionUpdater
    SecurityCheck --> StudentNotifier
    CompatibilityCheck --> CacheInvalidator
    
    SessionUpdater --> ConfigUpdate
    StudentNotifier --> KeyUpdate
    CacheInvalidator --> ToolUpdate
```

### Cost Management Integration
```mermaid
graph TB
    subgraph "Usage Tracking"
        UsageCollector[Usage Collector]
        MetricsAggregator[Metrics Aggregator]
        CostCalculator[Cost Calculator]
    end
    
    subgraph "Budget Management"
        BudgetTracker[Budget Tracker]
        ThresholdMonitor[Threshold Monitor]
        AlertGenerator[Alert Generator]
    end
    
    subgraph "Control Actions"
        UsageLimiter[Usage Limiter]
        ToolRestrictor[Tool Restrictor]
        EmergencyController[Emergency Controller]
    end
    
    UsageCollector --> BudgetTracker
    MetricsAggregator --> ThresholdMonitor
    CostCalculator --> AlertGenerator
    
    BudgetTracker --> UsageLimiter
    ThresholdMonitor --> ToolRestrictor
    AlertGenerator --> EmergencyController
    
    UsageLimiter --> UsageCollector
    ToolRestrictor --> MetricsAggregator
    EmergencyController --> CostCalculator
```