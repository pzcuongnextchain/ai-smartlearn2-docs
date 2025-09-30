# AI Video Recommendations Flows and Diagrams

## User Flow Diagrams

### Complete AI Video Recommendation Process (3-Step Flow)
```mermaid
flowchart TD
    A[Student Login] --> B[Access Learning Session]
    B --> C[Step 1: AI Chatbot Conversation]
    C --> D[Display Initial Message]
    D --> E[Student Responds to Questions]
    E --> F[AI Analyzes Response]
    F --> G{More Info Needed?}
    G -->|Yes| H[Ask Follow-up Questions] --> E
    G -->|No| I[Generate Recommendations]
    
    I --> J[Step 2: Video Selection]
    J --> K[Display Recommended Videos]
    K --> L[Student Reviews Options]
    L --> M[Auto-Select 5+ Minutes]
    M --> N[Manual Add/Remove Videos]
    N --> O{Total Time >= 5 min?}
    O -->|No| P[Show Time Warning] --> N
    O -->|Yes| Q[Confirm Selection]
    
    Q --> R[Step 3: Learning Experience]
    R --> S[Load Video Playlist]
    S --> T[Track Progress]
    T --> U[Display Comments/Likes]
    U --> V[Complete Session]
```

### AI Chatbot Conversation Flow (Step 1 Detail)
```mermaid
flowchart TD
    A[Session Start] --> B[Load Admin-Configured Initial Message]
    B --> C[Display Personalized Greeting]
    C --> D[Student Input]
    D --> E[AI Processing]
    E --> F[Analyze Job/Experience Context]
    F --> G{Sufficient Information?}
    
    G -->|No| H[Generate Follow-up Question]
    H --> I[Display Question to Student]
    I --> D
    
    G -->|Yes| J[Match Against Video Database]
    J --> K[Apply RAG Methodology]
    K --> L[Score Video Relevance]
    L --> M[Generate Recommendation List]
    M --> N[Transition to Step 2]
    
    style C fill:#e1f5fe
    style F fill:#f3e5f5
    style K fill:#e8f5e8
```

### Video Selection and Configuration Flow (Step 2 Detail)
```mermaid
flowchart TD
    A[Receive AI Recommendations] --> B[Display Video List]
    B --> C[Show Video Details]
    C --> D[Calculate Total Duration]
    D --> E{Duration >= 5 minutes?}
    
    E -->|No| F[Show Auto-Select Option]
    F --> G[Click Auto-Select]
    G --> H[Randomly Add Videos to Meet 5min]
    H --> I[Update Total Duration]
    
    E -->|Yes| J[Enable Manual Selection]
    J --> K[Student Adds/Removes Videos]
    K --> L[Update Duration Counter]
    L --> M{Still >= 5 minutes?}
    
    M -->|No| N[Show Warning Message] --> K
    M -->|Yes| O[Enable Setup Complete]
    
    I --> O
    O --> P[Confirm Final Selection]
    P --> Q[Generate Learning Playlist]
    Q --> R[Proceed to Step 3]
    
    style F fill:#fff3e0
    style N fill:#ffebee
    style O fill:#e8f5e8
```

## Sequence Diagrams

### AI Recommendation Generation Sequence
```mermaid
sequenceDiagram
    participant S as Student
    participant UI as Chat Interface
    participant AI as AI Service
    participant RAG as RAG Engine
    participant DB as Content Database
    participant Rec as Recommendation Engine
    
    S->>UI: Start video recommendation
    UI->>DB: Load initial message for session
    DB->>UI: Return configured message
    UI->>S: Display personalized greeting
    
    S->>UI: Send response about job/experience
    UI->>AI: Process student input
    AI->>RAG: Analyze context and requirements
    RAG->>DB: Query video content and scripts
    DB->>RAG: Return matching content
    RAG->>Rec: Generate relevance scores
    Rec->>AI: Return ranked recommendations
    AI->>UI: Send recommendation list
    UI->>S: Display recommended videos
```

### Video Content Analysis for Recommendations
```mermaid
sequenceDiagram
    participant Admin as Sub-Administrator
    participant CMS as Content Management
    participant AI as AI Analysis
    participant RAG as RAG System
    participant DB as Database
    
    Admin->>CMS: Upload video with script
    CMS->>AI: Process script content
    AI->>AI: Extract keywords and topics
    AI->>AI: Generate semantic embeddings
    AI->>RAG: Store processed content
    RAG->>DB: Save embeddings and metadata
    DB->>CMS: Confirm content ready
    CMS->>Admin: Show processing complete
    
    Note over AI,RAG: Content now available for recommendations
```

### Student Learning Progress Tracking
```mermaid
sequenceDiagram
    participant S as Student
    participant Player as Video Player
    participant Progress as Progress Tracker
    participant Analytics as Analytics Service
    participant DB as Database
    
    S->>Player: Start video playlist
    Player->>Progress: Initialize tracking
    
    loop For each video
        Player->>Progress: Update watch time
        Progress->>Analytics: Log progress event
        Player->>S: Display video content
        S->>Player: Interact (pause/resume/skip)
        Player->>Progress: Record interaction
    end
    
    Progress->>DB: Save completion status
    Analytics->>DB: Store learning analytics
    DB->>Player: Confirm data saved
    Player->>S: Show completion summary
```

## State Diagrams

### AI Recommendation Session States
```mermaid
stateDiagram-v2
    [*] --> Initialized: Student starts session
    Initialized --> Chatting: Display initial message
    Chatting --> Processing: Student provides input
    Processing --> Chatting: Need more information
    Processing --> Recommending: Sufficient data collected
    Recommending --> Selecting: Show video recommendations
    Selecting --> Configuring: Student makes selections
    Configuring --> Selecting: Adjust video list
    Configuring --> Learning: Confirm 5+ minute selection
    Learning --> Completed: Finish all videos
    Learning --> Paused: Student pauses session
    Paused --> Learning: Resume session
    Completed --> [*]: Session ends
    
    Chatting --> [*]: Student exits early
    Selecting --> [*]: Student exits early
    Configuring --> [*]: Student exits early
```

### Video Content Processing States
```mermaid
stateDiagram-v2
    [*] --> Uploaded: Admin uploads video
    Uploaded --> Processing: AI analysis starts
    Processing --> Analyzed: Script content processed
    Analyzed --> Indexed: Added to RAG system
    Indexed --> Available: Ready for recommendations
    Available --> Recommended: Included in student recommendations
    Recommended --> Selected: Student chooses video
    Selected --> Watched: Student views content
    Watched --> Completed: Video finished
    
    Processing --> Failed: Analysis error
    Failed --> Uploaded: Retry processing
    Available --> Updated: Content modified
    Updated --> Processing: Re-analyze content
```

## Activity Diagrams

### Complete Learning Journey
```mermaid
flowchart TD
    Start([Student Login]) --> CheckSession{Has Active Session?}
    CheckSession -->|Yes| ResumeSession[Resume Previous Session]
    CheckSession -->|No| NewSession[Start New Session]
    
    NewSession --> LoadMessage[Load Initial AI Message]
    LoadMessage --> DisplayGreeting[Show Personalized Greeting]
    ResumeSession --> DisplayGreeting
    
    DisplayGreeting --> WaitInput[Wait for Student Input]
    WaitInput --> ProcessInput[AI Processes Response]
    ProcessInput --> AnalyzeContext[Analyze Job/Experience Context]
    AnalyzeContext --> CheckSufficient{Enough Information?}
    
    CheckSufficient -->|No| GenerateQuestion[Create Follow-up Question]
    GenerateQuestion --> WaitInput
    CheckSufficient -->|Yes| QueryContent[Search Video Database]
    
    QueryContent --> ApplyRAG[Apply RAG Methodology]
    ApplyRAG --> ScoreVideos[Calculate Relevance Scores]
    ScoreVideos --> GenerateList[Create Recommendation List]
    GenerateList --> DisplayVideos[Show Recommended Videos]
    
    DisplayVideos --> StudentReview[Student Reviews Options]
    StudentReview --> CheckDuration{Total >= 5 minutes?}
    CheckDuration -->|No| ShowAutoSelect[Offer Auto-Select]
    ShowAutoSelect --> AutoAdd[Randomly Add Videos]
    AutoAdd --> UpdateDuration[Update Total Time]
    
    CheckDuration -->|Yes| ManualSelect[Allow Manual Selection]
    ManualSelect --> AddRemove[Student Adds/Removes Videos]
    AddRemove --> UpdateDuration
    UpdateDuration --> ValidateDuration{Still >= 5 minutes?}
    
    ValidateDuration -->|No| ShowWarning[Display Time Warning]
    ShowWarning --> AddRemove
    ValidateDuration -->|Yes| ConfirmSelection[Enable Setup Complete]
    
    ConfirmSelection --> CreatePlaylist[Generate Learning Playlist]
    CreatePlaylist --> StartLearning[Begin Video Learning]
    StartLearning --> TrackProgress[Monitor Learning Progress]
    TrackProgress --> Complete([Session Complete])
```

## Use Case Diagrams

### Student AI Recommendation Use Cases
```mermaid
graph LR
    Student((Student))
    AISystem[AI Recommendation System]
    ContentDB[(Content Database)]
    Analytics[Analytics System]
    
    Student --> UC1[Start Recommendation Session]
    Student --> UC2[Interact with AI Chatbot]
    Student --> UC3[Review Video Recommendations]
    Student --> UC4[Select Learning Videos]
    Student --> UC5[Watch Recommended Content]
    Student --> UC6[Track Learning Progress]
    
    UC1 --> AISystem
    UC2 --> AISystem
    UC3 --> AISystem
    UC4 --> AISystem
    UC5 --> ContentDB
    UC6 --> Analytics
    
    AISystem --> ContentDB
    AISystem --> Analytics
```

### AI System Internal Use Cases
```mermaid
graph TB
    AISystem((AI Recommendation System))
    
    AISystem --> UC1[Process Student Input]
    AISystem --> UC2[Analyze Context and Requirements]
    AISystem --> UC3[Query Content Database]
    AISystem --> UC4[Apply RAG Methodology]
    AISystem --> UC5[Calculate Relevance Scores]
    AISystem --> UC6[Generate Recommendations]
    AISystem --> UC7[Validate Minimum Duration]
    AISystem --> UC8[Create Learning Playlist]
    
    UC1 --> NLP[Natural Language Processing]
    UC2 --> ContextAnalysis[Context Analysis Engine]
    UC3 --> ContentSearch[Content Search Service]
    UC4 --> RAGEngine[RAG Processing Engine]
    UC5 --> ScoringAlgorithm[Scoring Algorithm]
    UC6 --> RecommendationEngine[Recommendation Engine]
    UC7 --> ValidationService[Validation Service]
    UC8 --> PlaylistGenerator[Playlist Generator]
```

## Component Interaction Diagrams

### AI Recommendation Architecture
```mermaid
graph TB
    subgraph "Student Interface Layer"
        ChatUI[Chat Interface]
        VideoUI[Video Selection UI]
        PlayerUI[Video Player UI]
    end
    
    subgraph "AI Processing Layer"
        NLP[Natural Language Processor]
        ContextAnalyzer[Context Analyzer]
        RAGEngine[RAG Engine]
        RecommendationEngine[Recommendation Engine]
    end
    
    subgraph "Data Layer"
        ContentDB[(Video Content DB)]
        ScriptDB[(Script Content DB)]
        UserDB[(User Profile DB)]
        AnalyticsDB[(Analytics DB)]
    end
    
    ChatUI --> NLP
    NLP --> ContextAnalyzer
    ContextAnalyzer --> RAGEngine
    RAGEngine --> ContentDB
    RAGEngine --> ScriptDB
    RAGEngine --> RecommendationEngine
    RecommendationEngine --> VideoUI
    VideoUI --> PlayerUI
    PlayerUI --> AnalyticsDB
    ContextAnalyzer --> UserDB
```