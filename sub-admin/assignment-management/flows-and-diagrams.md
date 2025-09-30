# Assignment Management Flows and Diagrams

## User Flow Diagrams

### Assignment Creation Flow (Slots 1-10)
```mermaid
flowchart TD
    A[Sub-Admin Access Assignment Management] --> B[Assignment Dashboard]
    B --> C[Select Assignment Slot 1-10]
    C --> D[Assignment Creation Wizard]
    D --> E[Enter Assignment Details]
    E --> F[Configure Assignment Parameters]
    F --> G{Assignment Components}
    
    G -->|Instructions| H[HTML Instruction Editor]
    G -->|AI Tools| I[Select Assignment Tools]
    G -->|Grading| J[Configure Grading Rubric]
    G -->|Timeline| K[Set Due Dates and Limits]
    
    H --> L[Rich Text Formatting]
    L --> M[Preview Instructions]
    M --> N[Save Instructions]
    
    I --> O[Filter Available Tools]
    O --> P[Select Tools for Assignment]
    P --> Q[Configure Tool Restrictions]
    
    J --> R[Rubric Builder Interface]
    R --> S[Define Grading Criteria]
    S --> T[Set Point Values]
    
    K --> U[Calendar Interface]
    U --> V[Set Due Date]
    V --> W[Configure Time Limits]
    
    N --> X[Validate Assignment]
    Q --> X
    T --> X
    W --> X
    X --> Y[Generate Assignment URL]
    Y --> Z[Publish Assignment]
    Z --> AA[Notify Students]
```

### Student Assignment Submission Flow
```mermaid
flowchart TD
    A[Student Access Assignment] --> B[Load Assignment Interface]
    B --> C[Display Assignment Instructions]
    C --> D[Show Available AI Tools]
    D --> E[Student Selects Tool]
    E --> F[Initialize AI Tool Interface]
    F --> G[Student Uses AI Tool]
    G --> H[Capture Tool Interactions]
    H --> I[Log Prompts and Outputs]
    I --> J{Continue Working?}
    
    J -->|Yes| K[Switch Tools or Continue]
    J -->|No| L[Prepare Submission]
    
    K --> G
    L --> M[Compile Interaction History]
    M --> N[Generate Submission Report]
    N --> O[Submit Assignment]
    O --> P[Confirm Submission Received]
    P --> Q[Notify Instructor]
    Q --> R[Update Assignment Status]
```

### Assignment Grading and Evaluation Flow
```mermaid
flowchart TD
    A[Instructor Access Grading] --> B[Load Assignment Submissions]
    B --> C[Select Student Submission]
    C --> D[Review AI Tool Interactions]
    D --> E[Analyze Prompt Quality]
    E --> F[Evaluate Tool Usage]
    F --> G[Assess Learning Objectives]
    G --> H[Apply Grading Rubric]
    H --> I[Provide Detailed Feedback]
    I --> J[Calculate Final Grade]
    J --> K[Save Grade and Comments]
    K --> L[Notify Student]
    L --> M{More Submissions?}
    
    M -->|Yes| N[Select Next Submission]
    M -->|No| O[Generate Grade Report]
    
    N --> C
    O --> P[Update Assignment Analytics]
    P --> Q[Complete Grading Process]
```

## Sequence Diagrams

### Assignment Creation and Configuration Sequence
```mermaid
sequenceDiagram
    participant SA as Sub-Administrator
    participant UI as Assignment Interface
    participant AM as Assignment Manager
    participant TM as Tool Manager
    participant DB as Database
    participant URL as URL Generator
    
    SA->>UI: Create new assignment (slot 1-10)
    UI->>AM: Initialize assignment creation
    AM->>DB: Validate assignment slot availability
    DB->>AM: Confirm slot available
    
    SA->>UI: Configure assignment details
    UI->>TM: Request available AI tools
    TM->>TM: Filter by master control and org permissions
    TM->>UI: Return tool list
    
    SA->>UI: Select tools and configure settings
    UI->>AM: Process assignment configuration
    AM->>URL: Generate unique assignment URL
    URL->>AM: Return secure URL
    
    AM->>DB: Save assignment configuration
    DB->>AM: Confirm assignment created
    AM->>UI: Assignment ready for publication
    UI->>SA: Display assignment URL and settings
```

### Student Assignment Interaction Sequence
```mermaid
sequenceDiagram
    participant S as Student
    parameter UI as Assignment Interface
    participant AM as Assignment Manager
    participant TM as Tool Manager
    participant AI as AI Services
    participant Logger as Interaction Logger
    
    S->>UI: Access assignment URL
    UI->>AM: Load assignment configuration
    AM->>UI: Return assignment details and tools
    UI->>S: Display assignment instructions
    
    S->>UI: Select AI tool for use
    UI->>TM: Initialize selected tool
    TM->>AI: Connect to AI service
    AI->>TM: Tool ready for use
    TM->>UI: Tool interface loaded
    
    S->>UI: Submit prompt to AI tool
    UI->>AI: Process prompt request
    AI->>UI: Return AI response
    UI->>Logger: Log interaction (prompt + response)
    Logger->>UI: Confirm logging
    UI->>S: Display AI response
    
    Note over Logger: All interactions captured for grading
```

### Assignment Grading Sequence
```mermaid
sequenceDiagram
    participant I as Instructor
    participant UI as Grading Interface
    participant AM as Assignment Manager
    participant Logger as Interaction Logger
    participant GS as Grading System
    participant Notifier as Notification Service
    
    I->>UI: Access assignment grading
    UI->>AM: Load assignment submissions
    AM->>Logger: Retrieve interaction logs
    Logger->>AM: Return detailed interaction history
    AM->>UI: Format submission data
    
    I->>UI: Review student AI tool usage
    UI->>GS: Apply grading rubric
    GS->>UI: Calculate preliminary scores
    
    I->>UI: Provide feedback and final grade
    UI->>GS: Save grade and comments
    GS->>Notifier: Send grade notification
    Notifier->>Student: Notify of graded assignment
    
    GS->>AM: Update assignment analytics
    AM->>UI: Confirm grading complete
```

## State Diagrams

### Assignment Lifecycle States
```mermaid
stateDiagram-v2
    [*] --> Created: Sub-Admin Creates Assignment
    Created --> Configuring: Add Instructions and Tools
    Configuring --> ReadyToPublish: Configuration Complete
    ReadyToPublish --> Published: Publish Assignment
    
    Published --> Active: Students Can Access
    Active --> InProgress: Students Working
    InProgress --> SubmissionPeriod: Due Date Approaching
    SubmissionPeriod --> Closed: Due Date Passed
    
    Closed --> Grading: Instructor Grading
    Grading --> Graded: All Submissions Graded
    Graded --> Archived: Assignment Complete
    
    Configuring --> Created: Save Draft
    Published --> Configuring: Edit Assignment
    Active --> Closed: Manual Close
    
    Archived --> [*]: Delete Assignment
```

### Student Assignment Progress States
```mermaid
stateDiagram-v2
    [*] --> NotStarted: Assignment Available
    NotStarted --> InProgress: Student Starts Work
    InProgress --> Paused: Student Pauses
    Paused --> InProgress: Student Resumes
    
    InProgress --> Submitted: Student Submits
    InProgress --> AutoSubmitted: Time Limit Reached
    
    Submitted --> UnderReview: Instructor Grading
    AutoSubmitted --> UnderReview: Instructor Grading
    UnderReview --> Graded: Grade Assigned
    
    Graded --> [*]: Assignment Complete
    NotStarted --> [*]: Assignment Expired
```

### AI Tool Usage States in Assignments
```mermaid
stateDiagram-v2
    [*] --> Available: Tool Configured for Assignment
    Available --> Selected: Student Selects Tool
    Selected --> Active: Tool Interface Loaded
    
    Active --> Processing: Student Submits Prompt
    Processing --> ResponseReady: AI Generates Response
    ResponseReady --> Active: Student Reviews Response
    
    Active --> Switched: Student Changes Tool
    Switched --> Selected: New Tool Selected
    
    Active --> Completed: Assignment Submitted
    Processing --> Completed: Submission During Processing
    
    Completed --> [*]: Assignment Ends
```

## Activity Diagrams

### Assignment Management Workflow
```mermaid
flowchart TD
    Start([Start Assignment Management]) --> ReviewSlots[Review Assignment Slots 1-10]
    ReviewSlots --> CheckActive{Active Assignments?}
    
    CheckActive -->|Yes| MonitorProgress[Monitor Student Progress]
    CheckActive -->|No| CreateNew[Create New Assignment]
    
    MonitorProgress --> CheckSubmissions[Check Submissions]
    CheckSubmissions --> GradingNeeded{Grading Required?}
    GradingNeeded -->|Yes| ProcessGrading[Process Grading]
    GradingNeeded -->|No| AnalyzeUsage[Analyze Tool Usage]
    
    CreateNew --> SelectSlot[Select Available Slot 1-10]
    SelectSlot --> ConfigureAssignment[Configure Assignment Details]
    ConfigureAssignment --> SelectTools[Select AI Tools]
    SelectTools --> SetGrading[Set Grading Criteria]
    SetGrading --> PublishAssignment[Publish Assignment]
    
    ProcessGrading --> ReviewInteractions[Review AI Tool Interactions]
    ReviewInteractions --> ApplyRubric[Apply Grading Rubric]
    ApplyRubric --> ProvideFeedback[Provide Student Feedback]
    
    AnalyzeUsage --> OptimizeTools[Optimize Tool Selection]
    PublishAssignment --> NotifyStudents[Notify Students]
    ProvideFeedback --> UpdateAnalytics[Update Assignment Analytics]
    OptimizeTools --> UpdateAnalytics
    NotifyStudents --> UpdateAnalytics
    
    UpdateAnalytics --> PlanImprovements[Plan Assignment Improvements]
    PlanImprovements --> End([Complete Assignment Management])
```

### Student Assignment Completion Workflow
```mermaid
flowchart TD
    AccessAssignment([Student Accesses Assignment]) --> ReadInstructions[Read Assignment Instructions]
    ReadInstructions --> UnderstandRequirements[Understand Requirements]
    UnderstandRequirements --> SelectTool[Select AI Tool]
    SelectTool --> FormulatePrompt[Formulate Prompt]
    FormulatePrompt --> SubmitPrompt[Submit to AI Tool]
    SubmitPrompt --> ReviewResponse[Review AI Response]
    ReviewResponse --> EvaluateQuality{Response Quality Good?}
    
    EvaluateQuality -->|No| RefinePrompt[Refine Prompt]
    EvaluateQuality -->|Yes| DocumentWork[Document Work Process]
    
    RefinePrompt --> SubmitPrompt
    DocumentWork --> MoreWork{Need More Tools?}
    MoreWork -->|Yes| SelectTool
    MoreWork -->|No| FinalReview[Final Review of Work]
    
    FinalReview --> ReadyToSubmit{Ready to Submit?}
    ReadyToSubmit -->|No| ContinueWork[Continue Working]
    ReadyToSubmit -->|Yes| SubmitAssignment[Submit Assignment]
    
    ContinueWork --> SelectTool
    SubmitAssignment --> ConfirmSubmission[Confirm Submission]
    ConfirmSubmission --> Complete([Assignment Complete])
```

## Use Case Diagrams

### Sub-Administrator Assignment Management Use Cases
```mermaid
graph LR
    SubAdmin((Sub-Administrator))
    
    SubAdmin --> UC1[Create Assignments 1-10]
    SubAdmin --> UC2[Configure AI Tools per Assignment]
    SubAdmin --> UC3[Set Grading Rubrics]
    SubAdmin --> UC4[Monitor Student Progress]
    SubAdmin --> UC5[Review Assignment Analytics]
    SubAdmin --> UC6[Manage Due Dates]
    SubAdmin --> UC7[Configure Assignment URLs]
    
    UC1 --> AssignmentSystem[Assignment Management System]
    UC2 --> ToolSystem[AI Tool System]
    UC3 --> GradingSystem[Grading System]
    UC4 --> ProgressSystem[Progress Tracking]
    UC5 --> AnalyticsSystem[Analytics System]
    UC6 --> ScheduleSystem[Scheduling System]
    UC7 --> URLSystem[URL Generation System]
```

### Student Assignment Use Cases
```mermaid
graph LR
    Student((Student))
    
    Student --> UC1[Access Assignment]
    Student --> UC2[Use AI Tools]
    Student --> UC3[Submit Work]
    Student --> UC4[View Feedback]
    Student --> UC5[Track Progress]
    
    UC1 --> AssignmentInterface[Assignment Interface]
    UC2 --> AITools[AI Tool Interface]
    UC3 --> SubmissionSystem[Submission System]
    UC4 --> FeedbackSystem[Feedback System]
    UC5 --> ProgressTracker[Progress Tracker]
    
    style Student fill:#4ecdc4
```

### Instructor Grading Use Cases
```mermaid
graph LR
    Instructor((Instructor))
    
    Instructor --> UC1[Review Submissions]
    Instructor --> UC2[Analyze AI Tool Usage]
    Instructor --> UC3[Apply Grading Rubric]
    Instructor --> UC4[Provide Feedback]
    Instructor --> UC5[Generate Reports]
    
    UC1 --> GradingInterface[Grading Interface]
    UC2 --> UsageAnalyzer[Usage Analyzer]
    UC3 --> RubricSystem[Rubric System]
    UC4 --> FeedbackSystem[Feedback System]
    UC5 --> ReportGenerator[Report Generator]
```

## Component Interaction Diagrams

### Assignment Management Architecture
```mermaid
graph TB
    subgraph "Assignment Management Layer"
        AssignmentCreator[Assignment Creator]
        ToolConfigurator[Tool Configurator]
        GradingManager[Grading Manager]
        AnalyticsEngine[Analytics Engine]
    end
    
    subgraph "Student Interaction Layer"
        AssignmentInterface[Assignment Interface]
        AIToolInterface[AI Tool Interface]
        SubmissionHandler[Submission Handler]
        ProgressTracker[Progress Tracker]
    end
    
    subgraph "Data Storage Layer"
        AssignmentDB[(Assignment Database)]
        InteractionDB[(Interaction Database)]
        GradingDB[(Grading Database)]
        AnalyticsDB[(Analytics Database)]
    end
    
    AssignmentCreator --> AssignmentInterface
    ToolConfigurator --> AIToolInterface
    GradingManager --> SubmissionHandler
    AnalyticsEngine --> ProgressTracker
    
    AssignmentInterface --> AssignmentDB
    AIToolInterface --> InteractionDB
    SubmissionHandler --> GradingDB
    ProgressTracker --> AnalyticsDB
```

### AI Tool Integration in Assignments
```mermaid
graph LR
    subgraph "Assignment Context"
        AssignmentConfig[Assignment Configuration]
        ToolSelection[Tool Selection]
        UsageTracking[Usage Tracking]
    end
    
    subgraph "AI Tool Layer"
        ToolManager[Tool Manager]
        EdenAI[Eden AI Services]
        CustomAPIs[Custom AI APIs]
    end
    
    subgraph "Interaction Capture"
        PromptLogger[Prompt Logger]
        ResponseLogger[Response Logger]
        UsageAnalyzer[Usage Analyzer]
    end
    
    AssignmentConfig --> ToolSelection
    ToolSelection --> ToolManager
    ToolManager --> EdenAI
    ToolManager --> CustomAPIs
    
    EdenAI --> PromptLogger
    CustomAPIs --> ResponseLogger
    PromptLogger --> UsageAnalyzer
    ResponseLogger --> UsageAnalyzer
    UsageAnalyzer --> UsageTracking
```