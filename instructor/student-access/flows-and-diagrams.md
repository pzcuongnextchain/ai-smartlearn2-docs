# Instructor Student Access Flows and Diagrams

## User Flow Diagrams

### Instructor Login and Access Flow
```mermaid
flowchart TD
    A[Instructor Login] --> B[System Validates Credentials]
    B --> C[Determine Instructor Role]
    C --> D[Load Assigned Students List]
    D --> E[Apply Access Restrictions]
    E --> F[Hide Other System Menus]
    F --> G[Display Student Access Interface]
    G --> H[Show Only Assigned Students]
    H --> I{Select Student Action}
    
    I -->|View Student Info| J[Display Student Details]
    I -->|Learning View| K[Show AI Video Recommendations]
    I -->|Assignment View| L[Show Practice Tool & Assignment Work]
    
    J --> M[Student Information Display]
    K --> N[AI-Recommended Video List]
    L --> O["Use AI Tool" Page Content]
    
    M --> P[Return to Student List]
    N --> P
    O --> P
    P --> I
```

### Student Assignment Access Flow
```mermaid
flowchart TD
    A[Instructor Accesses Students] --> B[System Loads Assignment Data]
    B --> C[Filter by Instructor Assignment]
    C --> D[Query Assigned Students Only]
    D --> E[Display Filtered Student List]
    E --> F{Student Selection}
    
    F -->|Select Student| G[Load Student Details]
    F -->|Search Students| H[Search Within Assigned Students]
    F -->|Bulk Operations| I[Bulk Actions on Assigned Students]
    
    G --> J[Display Student Information]
    H --> K[Show Search Results]
    I --> L[Execute Bulk Operations]
    
    J --> M[Student Detail Interface]
    K --> N[Filtered Results Display]
    L --> O[Bulk Operation Results]
    
    M --> P[Return to Student List]
    N --> P
    O --> P
```

### Access Restriction Enforcement Flow
```mermaid
flowchart TD
    A[Instructor Attempts Action] --> B[System Checks User Role]
    B --> C[Validate Instructor Permissions]
    C --> D{Action Permitted?}
    
    D -->|Student Access| E[Check Student Assignment]
    D -->|Other Features| F[Block Access]
    
    E --> G{Student Assigned?}
    G -->|Yes| H[Allow Access]
    G -->|No| I[Deny Access]
    
    F --> J[Show Access Restriction Message]
    I --> K[Show Student Not Assigned Message]
    
    H --> L[Execute Permitted Action]
    J --> M[Redirect to Allowed Area]
    K --> M
    
    L --> N[Log Successful Access]
    M --> O[Log Access Attempt]
    N --> P[Continue Operation]
    O --> P
```

## Sequence Diagrams

### Instructor Authentication and Setup Sequence
```mermaid
sequenceDiagram
    participant I as Instructor
    participant Auth as Authentication System
    participant Access as Access Control
    participant DB as Database
    participant UI as User Interface
    
    I->>Auth: Login with credentials
    Auth->>DB: Validate instructor credentials
    DB->>Auth: Return instructor profile
    Auth->>Access: Set instructor permissions
    Access->>DB: Query assigned students
    DB->>Access: Return assigned student list
    Access->>UI: Configure restricted interface
    UI->>I: Display student access interface
    
    Note over Access,UI: Only student access features visible
    Note over DB,Access: Data filtered to assigned students only
```

### Student Data Access Sequence
```mermaid
sequenceDiagram
    participant I as Instructor
    participant UI as User Interface
    participant Access as Access Control
    participant DB as Database
    participant Logger as Audit Logger
    
    I->>UI: Request student information
    UI->>Access: Validate student access
    Access->>DB: Query with instructor filter
    DB->>Access: Return filtered student data
    Access->>Logger: Log data access
    Logger->>Access: Confirm logging
    Access->>UI: Provide student information
    UI->>I: Display student details
    
    Note over DB,Access: Only assigned students returned
    Note over Logger: All access attempts logged
```

### Assignment Change Propagation Sequence
```mermaid
sequenceDiagram
    participant SA as Sub-Administrator
    participant UserMgmt as User Management
    participant Access as Access Control
    participant InstructorUI as Instructor Interface
    participant Cache as Cache Manager
    
    SA->>UserMgmt: Change student assignments
    UserMgmt->>Access: Update instructor permissions
    Access->>Cache: Invalidate instructor cache
    Cache->>InstructorUI: Notify of assignment changes
    InstructorUI->>Access: Request updated student list
    Access->>InstructorUI: Return new assigned students
    InstructorUI->>Instructor: Update interface with new students
    
    Note over Access,InstructorUI: Real-time permission updates
```

## State Diagrams

### Instructor Session States
```mermaid
stateDiagram-v2
    [*] --> LoggedOut: Initial State
    LoggedOut --> Authenticating: Login Attempt
    Authenticating --> Authenticated: Valid Credentials
    Authenticating --> LoggedOut: Invalid Credentials
    
    Authenticated --> StudentAccess: Load Student Interface
    StudentAccess --> ViewingStudent: Select Student
    ViewingStudent --> StudentAccess: Return to List
    
    StudentAccess --> AccessDenied: Attempt Restricted Access
    ViewingStudent --> AccessDenied: Unauthorized Action
    AccessDenied --> StudentAccess: Return to Allowed Interface
    
    StudentAccess --> LoggedOut: Logout
    ViewingStudent --> LoggedOut: Session Timeout
    AccessDenied --> LoggedOut: Security Logout
```

### Student Assignment States
```mermaid
stateDiagram-v2
    [*] --> Unassigned: Default State
    Unassigned --> Assigned: Sub-Admin Assigns Student
    Assigned --> Accessible: Instructor Can Access
    Accessible --> Unassigned: Student Reassigned
    
    Accessible --> Viewing: Instructor Views Student
    Viewing --> Accessible: Return to List
    Viewing --> Managing: Instructor Manages Student
    Managing --> Accessible: Complete Management
    
    Assigned --> [*]: Student Removed
    Accessible --> [*]: Instructor Removed
```

## Activity Diagrams

### Daily Instructor Workflow
```mermaid
flowchart TD
    Start([Instructor Starts Day]) --> Login[Login to System]
    Login --> CheckStudents[Review Assigned Students]
    CheckStudents --> SelectStudent{Choose Student}
    
    SelectStudent -->|Student A| ViewA[View Student A Details]
    SelectStudent -->|Student B| ViewB[View Student B Details]
    SelectStudent -->|Student N| ViewN[View Student N Details]
    
    ViewA --> CheckProgress[Check Learning Progress]
    ViewB --> CheckProgress
    ViewN --> CheckProgress
    
    CheckProgress --> ReviewWork[Review Student Work]
    ReviewWork --> UpdateNotes[Update Student Notes]
    UpdateNotes --> NextStudent{More Students?}
    
    NextStudent -->|Yes| SelectStudent
    NextStudent -->|No| GenerateReport[Generate Student Report]
    GenerateReport --> EndSession[End Session]
    EndSession --> End([Complete Daily Work])
```

### Student Information Access Workflow
```mermaid
flowchart TD
    AccessRequest([Request Student Information]) --> ValidateAccess[Validate Instructor Access]
    ValidateAccess --> CheckAssignment{Student Assigned?}
    
    CheckAssignment -->|Yes| LoadStudentData[Load Student Information]
    CheckAssignment -->|No| DenyAccess[Deny Access Request]
    
    LoadStudentData --> FilterData[Apply Data Filters]
    FilterData --> DisplayInfo[Display Student Information]
    DisplayInfo --> LogAccess[Log Access Event]
    
    DenyAccess --> LogDenial[Log Access Denial]
    LogDenial --> ShowError[Show Access Error]
    
    LogAccess --> Complete([Access Complete])
    ShowError --> Complete
```

## Use Case Diagrams

### Instructor Student Access Use Cases
```mermaid
graph LR
    Instructor((Instructor))
    
    Instructor --> UC1[Login with Restricted Access]
    Instructor --> UC2[View Assigned Students]
    Instructor --> UC3[Access Student Information]
    Instructor --> UC4[Manage Student Data]
    Instructor --> UC5[Generate Student Reports]
    
    UC1 --> AuthSystem[Authentication System]
    UC2 --> AccessControl[Access Control System]
    UC3 --> StudentData[Student Data System]
    UC4 --> StudentMgmt[Student Management]
    UC5 --> ReportSystem[Reporting System]
    
    style Instructor fill:#4ecdc4
    style AccessControl fill:#ff8b94
```

### System Access Control Use Cases
```mermaid
graph TB
    AccessControl((Access Control System))
    
    AccessControl --> UC1[Validate Instructor Role]
    AccessControl --> UC2[Filter Student Data]
    AccessControl --> UC3[Restrict Menu Access]
    AccessControl --> UC4[Enforce Assignment Scope]
    AccessControl --> UC5[Log Access Attempts]
    
    UC1 --> RoleSystem[Role Management]
    UC2 --> DataFilter[Data Filtering Service]
    UC3 --> UIController[UI Controller]
    UC4 --> AssignmentValidator[Assignment Validator]
    UC5 --> AuditSystem[Audit System]
```

## Component Interaction Diagrams

### Instructor Access Architecture
```mermaid
graph TB
    subgraph "Instructor Interface Layer"
        InstructorUI[Instructor Interface]
        StudentList[Student List View]
        StudentDetail[Student Detail View]
    end
    
    subgraph "Access Control Layer"
        RoleValidator[Role Validator]
        AssignmentFilter[Assignment Filter]
        MenuRestrictor[Menu Restrictor]
        AccessLogger[Access Logger]
    end
    
    subgraph "Data Layer"
        StudentDB[(Student Database)]
        AssignmentDB[(Assignment Database)]
        AuditDB[(Audit Database)]
    end
    
    InstructorUI --> RoleValidator
    StudentList --> AssignmentFilter
    StudentDetail --> AssignmentFilter
    
    RoleValidator --> AccessLogger
    AssignmentFilter --> StudentDB
    AssignmentFilter --> AssignmentDB
    MenuRestrictor --> InstructorUI
    
    AccessLogger --> AuditDB
    
    style RoleValidator fill:#ff8b94
    style AssignmentFilter fill:#ff8b94
    style MenuRestrictor fill:#ff8b94
```

### Real-time Assignment Updates
```mermaid
graph LR
    subgraph "Assignment Changes"
        SubAdminAction[Sub-Admin Assignment Change]
        AssignmentUpdate[Assignment Update]
        PermissionChange[Permission Change]
    end
    
    subgraph "Propagation System"
        ChangeDetector[Change Detector]
        CacheInvalidator[Cache Invalidator]
        UIUpdater[UI Updater]
    end
    
    subgraph "Instructor Interface"
        StudentListRefresh[Student List Refresh]
        AccessUpdate[Access Update]
        InterfaceSync[Interface Sync]
    end
    
    SubAdminAction --> ChangeDetector
    AssignmentUpdate --> CacheInvalidator
    PermissionChange --> UIUpdater
    
    ChangeDetector --> StudentListRefresh
    CacheInvalidator --> AccessUpdate
    UIUpdater --> InterfaceSync
    
    StudentListRefresh --> SubAdminAction
    AccessUpdate --> AssignmentUpdate
    InterfaceSync --> PermissionChange
```

### Security and Audit Integration
```mermaid
graph TB
    subgraph "Access Attempts"
        LoginAttempt[Login Attempt]
        StudentAccess[Student Access]
        UnauthorizedAccess[Unauthorized Access]
    end
    
    subgraph "Security Validation"
        AuthValidator[Authentication Validator]
        PermissionChecker[Permission Checker]
        AccessEnforcer[Access Enforcer]
    end
    
    subgraph "Audit and Logging"
        AuditLogger[Audit Logger]
        SecurityMonitor[Security Monitor]
        AlertSystem[Alert System]
    end
    
    LoginAttempt --> AuthValidator
    StudentAccess --> PermissionChecker
    UnauthorizedAccess --> AccessEnforcer
    
    AuthValidator --> AuditLogger
    PermissionChecker --> SecurityMonitor
    AccessEnforcer --> AlertSystem
    
    AuditLogger --> LoginAttempt
    SecurityMonitor --> StudentAccess
    AlertSystem --> UnauthorizedAccess
```