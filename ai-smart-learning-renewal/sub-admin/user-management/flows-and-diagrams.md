# User Management Flows and Diagrams

## User Flow Diagrams

### Student Management Flow
```mermaid
flowchart TD
    A[Sub-Admin Login] --> B[Access User Management]
    B --> C[Student Management Dashboard]
    C --> D{Select Action}
    
    D -->|Add Student| E[Individual Student Creation]
    D -->|Bulk Import| F[Bulk Student Registration]
    D -->|Edit Student| G[Student Profile Editing]
    D -->|View Progress| H[Learning Progress View]
    D -->|Assign Instructor| I[Instructor Assignment]
    
    E --> J[Student Creation Form]
    J --> K[Fill Required Fields]
    K --> L[Generate Login Token]
    L --> M[Encrypt Password SHA256]
    M --> N[Save Student Record]
    
    F --> O[Upload CSV/Excel File]
    O --> P[Validate File Format]
    P --> Q{Validation Passed?}
    Q -->|No| R[Show Error Report]
    Q -->|Yes| S[Process Bulk Import]
    S --> T[Generate Import Summary]
    
    G --> U[Load Student Profile]
    U --> V[Edit Allowed Fields]
    V --> W[Update Password Encryption]
    W --> X[Save Changes]
    
    H --> Y[Load Learning Data]
    Y --> Z[Display AI Recommendations]
    Z --> AA[Show Practice Tool Usage]
    AA --> BB[Display Assignment Progress]
    
    I --> CC[Select Students]
    CC --> DD[Choose Instructor]
    DD --> EE[Update Assignments]
    EE --> FF[Notify Instructor]
```

### Instructor Management Flow
```mermaid
flowchart TD
    A[Access Instructor Management] --> B[Instructor Dashboard]
    B --> C{Select Action}
    
    C -->|Add Instructor| D[Instructor Creation Form]
    C -->|Edit Instructor| E[Instructor Profile Editing]
    C -->|Assign Students| F[Student Assignment Interface]
    C -->|View Workload| G[Instructor Workload Analysis]
    
    D --> H[Fill Instructor Details]
    H --> I[Set Contact Information]
    I --> J[Generate Credentials]
    J --> K[Encrypt Password SHA256]
    K --> L[Set Organizational Scope]
    L --> M[Save Instructor Record]
    
    E --> N[Load Instructor Profile]
    N --> O[Edit Profile Information]
    O --> P[Update Password if Changed]
    P --> Q[Maintain Organizational Scope]
    Q --> R[Save Profile Changes]
    
    F --> S[Select Instructor]
    S --> T[View Current Students]
    T --> U[Add/Remove Students]
    U --> V[Balance Workload]
    V --> W[Update Student Access]
    W --> X[Notify Changes]
    
    G --> Y[Calculate Student Counts]
    Y --> Z[Analyze Distribution]
    Z --> AA[Generate Recommendations]
    AA --> BB[Display Workload Report]
```

### Bulk Student Import Flow
```mermaid
flowchart TD
    A[Start Bulk Import] --> B[File Upload Interface]
    B --> C[Select CSV/Excel File]
    C --> D[File Format Validation]
    D --> E{Format Valid?}
    
    E -->|No| F[Show Format Error]
    F --> G[Display Format Requirements]
    G --> B
    
    E -->|Yes| H[Parse File Content]
    H --> I[Validate Data Fields]
    I --> J[Check for Duplicates]
    J --> K[Validate Required Fields]
    K --> L{All Validations Pass?}
    
    L -->|No| M[Generate Error Report]
    M --> N[Show Line-by-Line Errors]
    N --> O[Allow Partial Import]
    O --> P{Proceed with Valid Records?}
    P -->|No| B
    P -->|Yes| Q[Process Valid Records]
    
    L -->|Yes| Q
    Q --> R[Generate Login Tokens]
    R --> S[Encrypt Passwords]
    S --> T[Create Student Records]
    T --> U[Assign Default Instructors]
    U --> V[Generate Success Report]
    V --> W[Send Notification Emails]
    W --> X[Update Dashboard]
```

## Sequence Diagrams

### Student Creation and Authentication Setup
```mermaid
sequenceDiagram
    participant SA as Sub-Administrator
    participant UI as User Interface
    participant Validator as Data Validator
    participant Crypto as Crypto Service
    participant DB as User Database
    participant TokenGen as Token Generator
    participant Notifier as Notification Service
    
    SA->>UI: Create new student
    UI->>Validator: Validate student data
    Validator->>Validator: Check required fields
    Validator->>DB: Check for duplicate ID
    DB->>Validator: Return duplicate check result
    
    alt No duplicates found
        Validator->>Crypto: Encrypt password with SHA256
        Crypto->>TokenGen: Generate unique login token
        TokenGen->>DB: Store student record
        DB->>Notifier: Trigger welcome notification
        Notifier->>SA: Confirm student created
    else Duplicate found
        Validator->>UI: Return duplicate error
        UI->>SA: Show error message
    end
```

### Bulk Import Processing Sequence
```mermaid
sequenceDiagram
    participant SA as Sub-Administrator
    participant Upload as Upload Service
    participant Parser as File Parser
    participant Validator as Bulk Validator
    participant Processor as Batch Processor
    participant DB as Database
    participant Reporter as Report Generator
    
    SA->>Upload: Upload CSV/Excel file
    Upload->>Parser: Parse file content
    Parser->>Validator: Validate all records
    
    loop For each record
        Validator->>Validator: Validate individual record
        Validator->>DB: Check for duplicates
    end
    
    Validator->>Processor: Send valid records
    Processor->>DB: Batch insert students
    DB->>Reporter: Generate processing results
    Reporter->>SA: Return import summary
    
    alt Errors found
        Validator->>Reporter: Generate error report
        Reporter->>SA: Show detailed error list
    end
```

### Instructor-Student Assignment Sequence
```mermaid
sequenceDiagram
    participant SA as Sub-Administrator
    participant UI as Assignment Interface
    participant WorkloadCalc as Workload Calculator
    participant DB as Database
    participant AccessControl as Access Control
    participant Instructor as Instructor Interface
    
    SA->>UI: Assign students to instructor
    UI->>WorkloadCalc: Calculate current workload
    WorkloadCalc->>DB: Query current assignments
    DB->>WorkloadCalc: Return assignment counts
    WorkloadCalc->>UI: Show workload analysis
    
    SA->>UI: Confirm assignments
    UI->>DB: Update student-instructor relationships
    DB->>AccessControl: Update access permissions
    AccessControl->>Instructor: Refresh instructor interface
    Instructor->>AccessControl: Confirm updated access
    AccessControl->>UI: Report assignment complete
    UI->>SA: Show assignment success
```

## State Diagrams

### Student Account States
```mermaid
stateDiagram-v2
    [*] --> Created: Admin Creates Student
    Created --> Active: Account Activated
    Created --> PendingVerification: Email Verification Required
    
    PendingVerification --> Active: Email Verified
    PendingVerification --> Expired: Verification Timeout
    
    Active --> Learning: Student Logs In
    Active --> Suspended: Admin Suspension
    Active --> Inactive: Long Period No Activity
    
    Learning --> Active: Session Ends
    Learning --> Suspended: Violation Detected
    
    Suspended --> Active: Admin Reactivation
    Inactive --> Active: Student Returns
    Expired --> [*]: Account Deleted
    
    Active --> [*]: Admin Deletion
    Suspended --> [*]: Permanent Ban
```

### Instructor Account States
```mermaid
stateDiagram-v2
    [*] --> Created: Admin Creates Instructor
    Created --> Active: Account Activated
    Active --> Teaching: Students Assigned
    Active --> Idle: No Students Assigned
    
    Teaching --> Active: Teaching Session Ends
    Teaching --> Overloaded: Too Many Students
    Idle --> Teaching: Students Assigned
    
    Overloaded --> Teaching: Students Redistributed
    Active --> Suspended: Admin Action
    Teaching --> Suspended: Policy Violation
    
    Suspended --> Active: Reactivation
    Suspended --> [*]: Account Terminated
    Active --> [*]: Admin Deletion
    Teaching --> [*]: End of Contract
```

### Bulk Import Process States
```mermaid
stateDiagram-v2
    [*] --> Uploaded: File Uploaded
    Uploaded --> Parsing: Parse File Content
    Parsing --> Validating: Validate Records
    Parsing --> ParseError: File Format Error
    
    Validating --> Processing: All Valid
    Validating --> PartialValid: Some Errors Found
    Validating --> AllInvalid: All Records Invalid
    
    Processing --> Completed: All Records Processed
    PartialValid --> Completed: Valid Records Processed
    
    ParseError --> [*]: Process Terminated
    AllInvalid --> [*]: Process Terminated
    Completed --> [*]: Import Finished
```

## Activity Diagrams

### Daily User Management Workflow
```mermaid
flowchart TD
    Start([Start Daily User Management]) --> CheckRequests{New User Requests?}
    CheckRequests -->|Yes| ProcessRequests[Process New Requests]
    CheckRequests -->|No| ReviewAccounts[Review Existing Accounts]
    
    ProcessRequests --> ValidateRequests[Validate Request Data]
    ValidateRequests --> CreateAccounts[Create User Accounts]
    CreateAccounts --> AssignInstructors[Assign to Instructors]
    AssignInstructors --> SendNotifications[Send Welcome Notifications]
    
    ReviewAccounts --> CheckActivity[Check User Activity]
    CheckActivity --> IdentifyIssues{Issues Found?}
    IdentifyIssues -->|Yes| ResolveIssues[Resolve Account Issues]
    IdentifyIssues -->|No| AnalyzeUsage[Analyze Usage Patterns]
    
    ResolveIssues --> UpdateAccounts[Update Account Status]
    AnalyzeUsage --> OptimizeAssignments[Optimize Instructor Assignments]
    
    SendNotifications --> GenerateReports[Generate Daily Reports]
    UpdateAccounts --> GenerateReports
    OptimizeAssignments --> GenerateReports
    GenerateReports --> PlanImprovements[Plan System Improvements]
    PlanImprovements --> End([End Daily Workflow])
```

### Student Onboarding Process
```mermaid
flowchart TD
    NewStudent([New Student Request]) --> ValidateInfo[Validate Student Information]
    ValidateInfo --> CheckDuplicates{Duplicate Check}
    CheckDuplicates -->|Duplicate Found| HandleDuplicate[Handle Duplicate Case]
    CheckDuplicates -->|No Duplicate| CreateAccount[Create Student Account]
    
    HandleDuplicate --> ContactAdmin[Contact Administrator]
    ContactAdmin --> ResolveConflict[Resolve Identity Conflict]
    ResolveConflict --> CreateAccount
    
    CreateAccount --> GenerateCredentials[Generate Login Credentials]
    GenerateCredentials --> EncryptPassword[Encrypt Password SHA256]
    EncryptPassword --> CreateToken[Generate Login Token]
    CreateToken --> AssignInstructor[Assign Default Instructor]
    
    AssignInstructor --> SetupAccess[Setup Learning Access]
    SetupAccess --> SendWelcome[Send Welcome Email]
    SendWelcome --> NotifyInstructor[Notify Assigned Instructor]
    NotifyInstructor --> MonitorOnboarding[Monitor Initial Activity]
    MonitorOnboarding --> Complete([Onboarding Complete])
```

## Use Case Diagrams

### Sub-Administrator User Management Use Cases
```mermaid
graph LR
    SubAdmin((Sub-Administrator))
    
    SubAdmin --> UC1[Create Individual Students]
    SubAdmin --> UC2[Bulk Import Students]
    SubAdmin --> UC3[Edit Student Profiles]
    SubAdmin --> UC4[Manage Instructors]
    SubAdmin --> UC5[Assign Students to Instructors]
    SubAdmin --> UC6[View Learning Progress]
    SubAdmin --> UC7[Generate User Reports]
    SubAdmin --> UC8[Manage User Access]
    
    UC1 --> StudentSystem[Student Management System]
    UC2 --> BulkSystem[Bulk Import System]
    UC3 --> ProfileSystem[Profile Management System]
    UC4 --> InstructorSystem[Instructor Management System]
    UC5 --> AssignmentSystem[Assignment System]
    UC6 --> ProgressSystem[Progress Tracking System]
    UC7 --> ReportSystem[Reporting System]
    UC8 --> AccessSystem[Access Control System]
```

### Instructor Limited Access Use Cases
```mermaid
graph LR
    Instructor((Instructor))
    
    Instructor --> UC1[View Assigned Students Only]
    Instructor --> UC2[Monitor Student Progress]
    Instructor --> UC3[Access Learning Reports]
    Instructor --> UC4[Update Student Notes]
    
    UC1 --> FilteredView[Filtered Student View]
    UC2 --> ProgressMonitor[Progress Monitoring]
    UC3 --> LimitedReports[Limited Reporting]
    UC4 --> StudentNotes[Student Notes System]
    
    FilteredView --> AccessControl[Access Control System]
    ProgressMonitor --> AccessControl
    LimitedReports --> AccessControl
    StudentNotes --> AccessControl
    
    style Instructor fill:#4ecdc4
    style AccessControl fill:#ff8b94
```

## Component Interaction Diagrams

### User Management Architecture
```mermaid
graph TB
    subgraph "User Interface Layer"
        StudentMgmtUI[Student Management UI]
        InstructorMgmtUI[Instructor Management UI]
        BulkImportUI[Bulk Import UI]
        ProgressUI[Progress Monitoring UI]
    end
    
    subgraph "Business Logic Layer"
        UserManager[User Manager]
        BulkProcessor[Bulk Processor]
        PasswordManager[Password Manager]
        TokenGenerator[Token Generator]
        WorkloadBalancer[Workload Balancer]
    end
    
    subgraph "Security Layer"
        Authenticator[Authenticator]
        Encryptor[SHA256 Encryptor]
        AccessController[Access Controller]
        AuditLogger[Audit Logger]
    end
    
    subgraph "Data Layer"
        UserDB[(User Database)]
        ProgressDB[(Progress Database)]
        AuditDB[(Audit Database)]
        SessionDB[(Session Database)]
    end
    
    StudentMgmtUI --> UserManager
    InstructorMgmtUI --> UserManager
    BulkImportUI --> BulkProcessor
    ProgressUI --> UserManager
    
    UserManager --> PasswordManager
    UserManager --> TokenGenerator
    BulkProcessor --> UserManager
    UserManager --> WorkloadBalancer
    
    PasswordManager --> Encryptor
    TokenGenerator --> Authenticator
    UserManager --> AccessController
    AccessController --> AuditLogger
    
    UserManager --> UserDB
    UserManager --> ProgressDB
    AuditLogger --> AuditDB
    Authenticator --> SessionDB
```

### Multi-Tenant User Isolation
```mermaid
graph LR
    subgraph "Organization A"
        AdminA[Sub-Admin A]
        StudentsA[Students A]
        InstructorsA[Instructors A]
    end
    
    subgraph "Organization B"
        AdminB[Sub-Admin B]
        StudentsB[Students B]
        InstructorsB[Instructors B]
    end
    
    subgraph "Isolation Layer"
        OrgFilter[Organization Filter]
        DataPartition[Data Partitioning]
        AccessValidator[Access Validator]
    end
    
    subgraph "Shared Services"
        UserService[User Service]
        AuthService[Authentication Service]
        AuditService[Audit Service]
    end
    
    AdminA --> OrgFilter
    AdminB --> OrgFilter
    StudentsA --> OrgFilter
    StudentsB --> OrgFilter
    InstructorsA --> OrgFilter
    InstructorsB --> OrgFilter
    
    OrgFilter --> DataPartition
    DataPartition --> AccessValidator
    AccessValidator --> UserService
    AccessValidator --> AuthService
    AccessValidator --> AuditService
    
    style OrgFilter fill:#ffe66d
    style DataPartition fill:#ff8b94
    style AccessValidator fill:#4ecdc4
```