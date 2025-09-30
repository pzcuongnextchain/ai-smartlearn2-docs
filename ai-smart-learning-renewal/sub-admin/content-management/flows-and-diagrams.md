# Content Management Flows and Diagrams

## User Flow Diagrams

### Video Content Management Flow (Sessions 1-40)
```mermaid
flowchart TD
    A[Sub-Admin Login] --> B[Access Content Management]
    B --> C[Select Session Slot 1-40]
    C --> D[Video Management Interface]
    D --> E{Choose Action}
    
    E -->|Add Video| F[Upload Video File]
    E -->|Edit Content| G[Modify Existing Video]
    E -->|Configure AI| H[Add Script Content]
    E -->|Manage Sessions| I[Session Organization]
    
    F --> J[Video Upload Process]
    J --> K[Generate Thumbnail]
    K --> L[Add Video Metadata]
    L --> M[Script Content Entry]
    
    G --> N[Edit Video Details]
    N --> O[Update Metadata]
    O --> M
    
    H --> P[HTML Script Editor]
    P --> Q[Add Hashtags]
    Q --> R[AI Content Processing]
    
    I --> S[Session URL Generation]
    S --> T[Organizational URL Config]
    
    M --> U[Save Content]
    R --> U
    T --> U
    U --> V[Update AI Recommendation Database]
    V --> W[Notify System of Changes]
```

### Practice Session Configuration Flow (Sessions 1-40)
```mermaid
flowchart TD
    A[Access Practice Management] --> B[Select Practice Session 1-40]
    B --> C[Session Configuration Interface]
    C --> D{Configuration Type}
    
    D -->|AI Tools| E[Select Available Tools]
    D -->|Pre-Message| F[Configure Guidance Message]
    D -->|Session URL| G[Generate Practice URL]
    D -->|Tool Settings| H[Configure Tool Parameters]
    
    E --> I[Filter by Master Control]
    I --> J[Check Organization Permissions]
    J --> K[Display Available Tools]
    K --> L[Multi-Select Tool Interface]
    
    F --> M[HTML Message Editor]
    M --> N[Preview Message Display]
    N --> O[Save Message Configuration]
    
    G --> P[Generate Unique URL]
    P --> Q[Include Org Identifier]
    Q --> R[Add Security Tokens]
    
    H --> S[Tool-Specific Settings]
    S --> T[API Key Validation]
    T --> U[Save Tool Configuration]
    
    L --> V[Validate Tool Selection]
    O --> V
    R --> V
    U --> V
    V --> W[Update Session Database]
    W --> X[Test Session Configuration]
    X --> Y[Publish Session]
```

### Assignment Management Flow (Assignments 1-10)
```mermaid
flowchart TD
    A[Access Assignment Management] --> B[Select Assignment Slot 1-10]
    B --> C[Assignment Creation Wizard]
    C --> D[Assignment Details Form]
    D --> E[Configure Assignment Parameters]
    E --> F{Assignment Components}
    
    F -->|Instructions| G[HTML Instruction Editor]
    F -->|AI Tools| H[Select Assignment Tools]
    F -->|Grading| I[Configure Grading Rubric]
    F -->|Timeline| J[Set Due Dates and Limits]
    
    G --> K[Rich Text Formatting]
    K --> L[Preview Instructions]
    L --> M[Save Instructions]
    
    H --> N[Filter Available Tools]
    N --> O[Select Tools for Assignment]
    O --> P[Configure Tool Restrictions]
    
    I --> Q[Rubric Builder Interface]
    Q --> R[Define Grading Criteria]
    R --> S[Set Point Values]
    
    J --> T[Calendar Interface]
    T --> U[Set Due Date]
    U --> V[Configure Time Limits]
    
    M --> W[Validate Assignment]
    P --> W
    S --> W
    V --> W
    W --> X[Generate Assignment URL]
    X --> Y[Publish Assignment]
    Y --> Z[Notify Students]
```

## Sequence Diagrams

### Video Upload and AI Processing Sequence
```mermaid
sequenceDiagram
    participant SA as Sub-Administrator
    participant UI as Content Interface
    participant Upload as Upload Service
    participant AI as AI Processor
    participant DB as Content Database
    participant RAG as RAG System
    
    SA->>UI: Upload video with script
    UI->>Upload: Process video file
    Upload->>Upload: Generate thumbnail
    Upload->>AI: Process script content
    AI->>AI: Extract keywords and topics
    AI->>AI: Generate semantic embeddings
    AI->>RAG: Store processed content
    RAG->>DB: Save content and metadata
    DB->>UI: Confirm content ready
    UI->>SA: Show upload success
    
    Note over AI,RAG: Content now available for AI recommendations
```

### Session URL Generation Sequence
```mermaid
sequenceDiagram
    participant SA as Sub-Administrator
    participant Config as Session Config
    participant URL as URL Generator
    participant Security as Security Service
    participant DB as Session Database
    
    SA->>Config: Configure session settings
    Config->>URL: Request session URL
    URL->>Security: Generate security tokens
    Security->>URL: Return encrypted tokens
    URL->>URL: Build URL with org context
    URL->>DB: Store URL mapping
    DB->>Config: Confirm URL created
    Config->>SA: Display session URL
    
    Note over URL,Security: URL includes org ID and security validation
```

### AI Tool Configuration Propagation
```mermaid
sequenceDiagram
    participant SA as Sub-Administrator
    participant ToolConfig as Tool Configuration
    participant Master as Master Control
    participant Session as Session Manager
    participant Student as Student Interface
    
    SA->>ToolConfig: Configure tools for session
    ToolConfig->>Master: Validate tool permissions
    Master->>ToolConfig: Return available tools
    ToolConfig->>Session: Update session tool list
    Session->>Student: Update tool availability
    Student->>Session: Confirm tools loaded
    Session->>ToolConfig: Report configuration success
    ToolConfig->>SA: Show configuration complete
```

## State Diagrams

### Content Management States
```mermaid
stateDiagram-v2
    [*] --> Draft: Create New Content
    Draft --> InProgress: Start Editing
    InProgress --> Draft: Save Draft
    InProgress --> Processing: Submit for AI Processing
    Processing --> Processed: AI Analysis Complete
    Processing --> Failed: Processing Error
    Failed --> InProgress: Retry Processing
    Processed --> Published: Publish Content
    Published --> InProgress: Edit Published Content
    Published --> Archived: Archive Content
    Archived --> Published: Restore Content
    Archived --> [*]: Delete Content
```

### Session Configuration States
```mermaid
stateDiagram-v2
    [*] --> Created: New Session Created
    Created --> Configuring: Start Configuration
    Configuring --> ToolSelection: Select AI Tools
    Configuring --> MessageSetup: Configure Messages
    Configuring --> URLGeneration: Generate URLs
    
    ToolSelection --> Validating: Validate Tool Access
    MessageSetup --> Validating: Validate Messages
    URLGeneration --> Validating: Validate URLs
    
    Validating --> Ready: All Validations Pass
    Validating --> Configuring: Validation Errors
    
    Ready --> Active: Publish Session
    Active --> Configuring: Edit Session
    Active --> Inactive: Deactivate Session
    Inactive --> Active: Reactivate Session
    Inactive --> [*]: Delete Session
```

## Activity Diagrams

### Daily Content Management Workflow
```mermaid
flowchart TD
    Start([Start Daily Content Management]) --> CheckUpdates{Content Updates Needed?}
    CheckUpdates -->|Yes| ReviewContent[Review Content Performance]
    CheckUpdates -->|No| MonitorUsage[Monitor Content Usage]
    
    ReviewContent --> AnalyzeMetrics[Analyze Video Metrics]
    AnalyzeMetrics --> IdentifyIssues{Issues Found?}
    IdentifyIssues -->|Yes| UpdateContent[Update Content]
    IdentifyIssues -->|No| OptimizeContent[Optimize for AI]
    
    UpdateContent --> ProcessAI[Reprocess with AI]
    OptimizeContent --> ProcessAI
    ProcessAI --> TestRecommendations[Test AI Recommendations]
    
    MonitorUsage --> CheckSessions[Check Session Performance]
    CheckSessions --> ReviewTools[Review Tool Usage]
    ReviewTools --> UpdateSessions{Sessions Need Updates?}
    UpdateSessions -->|Yes| ConfigureSessions[Update Session Config]
    UpdateSessions -->|No| GenerateReports[Generate Usage Reports]
    
    TestRecommendations --> ValidateChanges[Validate Changes]
    ConfigureSessions --> ValidateChanges
    ValidateChanges --> GenerateReports
    GenerateReports --> PlanImprovements[Plan Future Improvements]
    PlanImprovements --> End([End Daily Workflow])
```

### Content Publishing Workflow
```mermaid
flowchart TD
    ContentReady([Content Ready for Publishing]) --> FinalReview[Final Content Review]
    FinalReview --> CheckCompliance{Meets Standards?}
    CheckCompliance -->|No| ReviseContent[Revise Content]
    CheckCompliance -->|Yes| ProcessAI[Process with AI Systems]
    
    ReviseContent --> FinalReview
    ProcessAI --> GenerateMetadata[Generate AI Metadata]
    GenerateMetadata --> CreateThumbnails[Create Thumbnails]
    CreateThumbnails --> ConfigureAccess[Configure Access Permissions]
    
    ConfigureAccess --> TestAccess[Test Student Access]
    TestAccess --> ValidationCheck{Access Working?}
    ValidationCheck -->|No| FixAccess[Fix Access Issues]
    ValidationCheck -->|Yes| PublishContent[Publish Content]
    
    FixAccess --> TestAccess
    PublishContent --> NotifyUsers[Notify Relevant Users]
    NotifyUsers --> MonitorInitialUsage[Monitor Initial Usage]
    MonitorInitialUsage --> DocumentChanges[Document Changes]
    DocumentChanges --> Complete([Publishing Complete])
```

## Use Case Diagrams

### Sub-Administrator Content Management Use Cases
```mermaid
graph LR
    SubAdmin((Sub-Administrator))
    
    SubAdmin --> UC1[Manage Video Content]
    SubAdmin --> UC2[Configure Practice Sessions]
    SubAdmin --> UC3[Create Assignments]
    SubAdmin --> UC4[Set AI Tool Access]
    SubAdmin --> UC5[Generate Session URLs]
    SubAdmin --> UC6[Monitor Content Performance]
    SubAdmin --> UC7[Configure AI Messages]
    SubAdmin --> UC8[Manage Content Metadata]
    
    UC1 --> VideoSystem[Video Management System]
    UC2 --> SessionSystem[Session Management System]
    UC3 --> AssignmentSystem[Assignment System]
    UC4 --> ToolSystem[AI Tool System]
    UC5 --> URLSystem[URL Generation System]
    UC6 --> AnalyticsSystem[Analytics System]
    UC7 --> MessageSystem[Message Configuration System]
    UC8 --> MetadataSystem[Metadata Management System]
```

### Content Integration Use Cases
```mermaid
graph TB
    ContentMgmt((Content Management))
    
    ContentMgmt --> UC1[Process Video Content]
    ContentMgmt --> UC2[Generate AI Embeddings]
    ContentMgmt --> UC3[Create Session URLs]
    ContentMgmt --> UC4[Configure Tool Access]
    ContentMgmt --> UC5[Manage Organizational Scope]
    
    UC1 --> VideoProcessor[Video Processing Service]
    UC2 --> AIProcessor[AI Processing Service]
    UC3 --> URLGenerator[URL Generation Service]
    UC4 --> ToolController[Tool Control Service]
    UC5 --> OrgManager[Organization Manager]
    
    VideoProcessor --> Storage[(Content Storage)]
    AIProcessor --> RAGSystem[(RAG System)]
    URLGenerator --> Security[Security Service]
    ToolController --> MasterControl[Master Control]
    OrgManager --> Database[(Organization Database)]
```

## Component Interaction Diagrams

### Content Management Architecture
```mermaid
graph TB
    subgraph "Content Management Layer"
        VideoMgmt[Video Management]
        SessionMgmt[Session Management]
        AssignmentMgmt[Assignment Management]
        MessageConfig[Message Configuration]
    end
    
    subgraph "Processing Layer"
        VideoProcessor[Video Processor]
        AIProcessor[AI Content Processor]
        URLGenerator[URL Generator]
        ToolValidator[Tool Validator]
    end
    
    subgraph "Integration Layer"
        RAGSystem[RAG System]
        ToolControl[Tool Control System]
        SecurityService[Security Service]
        NotificationService[Notification Service]
    end
    
    subgraph "Data Layer"
        ContentDB[(Content Database)]
        SessionDB[(Session Database)]
        MetadataDB[(Metadata Database)]
        AIDB[(AI Processing Database)]
    end
    
    VideoMgmt --> VideoProcessor
    SessionMgmt --> URLGenerator
    AssignmentMgmt --> ToolValidator
    MessageConfig --> SecurityService
    
    VideoProcessor --> AIProcessor
    AIProcessor --> RAGSystem
    URLGenerator --> SecurityService
    ToolValidator --> ToolControl
    
    RAGSystem --> AIDB
    ToolControl --> SessionDB
    SecurityService --> ContentDB
    NotificationService --> MetadataDB
```

### Real-time Content Synchronization
```mermaid
graph LR
    subgraph "Content Updates"
        VideoUpdate[Video Updates]
        SessionUpdate[Session Updates]
        ToolUpdate[Tool Updates]
    end
    
    subgraph "Processing Pipeline"
        EventQueue[Event Queue]
        Processor[Content Processor]
        Validator[Validation Service]
    end
    
    subgraph "Distribution"
        StudentInterface[Student Interfaces]
        InstructorInterface[Instructor Interfaces]
        AnalyticsSystem[Analytics System]
    end
    
    VideoUpdate -->|Content Events| EventQueue
    SessionUpdate -->|Config Events| EventQueue
    ToolUpdate -->|Tool Events| EventQueue
    
    EventQueue --> Processor
    Processor --> Validator
    Validator -->|Valid Updates| StudentInterface
    Validator -->|Valid Updates| InstructorInterface
    Validator -->|Usage Data| AnalyticsSystem
    
    StudentInterface -->|Feedback| EventQueue
    InstructorInterface -->|Feedback| EventQueue
    AnalyticsSystem -->|Metrics| EventQueue
```