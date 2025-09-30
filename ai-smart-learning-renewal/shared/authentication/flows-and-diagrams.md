# Multi-Role Authentication Flows and Diagrams

## User Flow Diagrams

### Multi-Role Login Flow
```mermaid
flowchart TD
    A[User Access Login Page] --> B[Enter Credentials]
    B --> C[Submit Login Request]
    C --> D[System Validates Credentials]
    D --> E{Credentials Valid?}
    
    E -->|No| F[Increment Failed Attempts]
    F --> G{Attempts >= 5?}
    G -->|Yes| H[Lock Account 15 Minutes]
    G -->|No| I[Show Invalid Credentials Error]
    
    H --> J[Display Lockout Message]
    I --> B
    J --> K[Wait for Lockout Period]
    K --> B
    
    E -->|Yes| L[Determine User Role]
    L --> M{User Role}
    
    M -->|Super Administrator| N[Load Super Admin Dashboard]
    M -->|Sub-Administrator| O[Load Sub-Admin Dashboard]
    M -->|Instructor| P[Load Instructor Interface]
    M -->|Student| Q[Load Student Interface]
    
    N --> R[Apply Global Permissions]
    O --> S[Apply Organizational Scope]
    P --> T[Apply Student Filter]
    Q --> U[Apply Content Access]
    
    R --> V[Generate Session Token]
    S --> V
    T --> V
    U --> V
    V --> W[Set Session Timeout]
    W --> X[Log Successful Login]
    X --> Y[Redirect to Role Dashboard]
```

### Password Management Flow
```mermaid
flowchart TD
    A[Password Management Request] --> B{Request Type}
    
    B -->|Change Password| C[Current Password Verification]
    B -->|Reset Password| D[Password Reset Request]
    B -->|First Time Setup| E[Initial Password Setup]
    
    C --> F[Verify Current Password]
    F --> G{Current Password Valid?}
    G -->|No| H[Show Verification Error]
    G -->|Yes| I[Enter New Password]
    
    H --> C
    I --> J[Validate Password Strength]
    
    D --> K[Verify User Identity]
    K --> L[Send Reset Email]
    L --> M[User Clicks Reset Link]
    M --> N[Validate Reset Token]
    N --> O{Token Valid?}
    O -->|No| P[Show Token Error]
    O -->|Yes| Q[Enter New Password]
    
    P --> D
    Q --> J
    
    E --> R[Generate Temporary Password]
    R --> S[Send Setup Instructions]
    S --> T[User Sets New Password]
    T --> J
    
    J --> U{Password Meets Requirements?}
    U -->|No| V[Show Strength Requirements]
    U -->|Yes| W[Encrypt with SHA256]
    
    V --> I
    W --> X[Store Encrypted Password]
    X --> Y[Invalidate Old Sessions]
    Y --> Z[Confirm Password Updated]
```

### Organizational Context Setup Flow
```mermaid
flowchart TD
    A[User Authentication Success] --> B[Determine User Role]
    B --> C{Role Type}
    
    C -->|Super Administrator| D[Set Global Context]
    C -->|Sub-Administrator| E[Load Organization ID]
    C -->|Instructor| F[Load Organization and Student Filter]
    C -->|Student| G[Load Organization and Content Access]
    
    D --> H[Access All Organizations]
    E --> I[Query Organization Details]
    F --> J[Query Assigned Students]
    G --> K[Query Available Content]
    
    H --> L[Set Unlimited Permissions]
    I --> M[Set Organizational Scope]
    J --> N[Set Student Scope Filter]
    K --> O[Set Content Access Filter]
    
    L --> P[Generate Session Context]
    M --> P
    N --> P
    O --> P
    
    P --> Q[Store Context in Session]
    Q --> R[Apply Data Filters]
    R --> S[Configure UI Permissions]
    S --> T[Context Setup Complete]
```

## Sequence Diagrams

### Multi-Role Authentication Sequence
```mermaid
sequenceDiagram
    participant U as User
    participant UI as Login Interface
    participant Auth as Authentication Service
    participant DB as User Database
    participant Session as Session Manager
    participant Role as Role Manager
    
    U->>UI: Enter credentials
    UI->>Auth: Submit login request
    Auth->>DB: Validate credentials
    DB->>Auth: Return user profile and role
    
    alt Valid Credentials
        Auth->>Role: Determine user permissions
        Role->>Auth: Return role-based permissions
        Auth->>Session: Create session with role context
        Session->>Auth: Return session token
        Auth->>UI: Login successful with role
        UI->>U: Redirect to role-appropriate dashboard
    else Invalid Credentials
        Auth->>Auth: Increment failed attempts
        Auth->>UI: Return authentication error
        UI->>U: Show error message
    end
```

### Organizational Context Establishment Sequence
```mermaid
sequenceDiagram
    participant Auth as Authentication Service
    participant OrgMgr as Organization Manager
    participant DataFilter as Data Filter
    participant PermMgr as Permission Manager
    participant Session as Session Manager
    
    Auth->>OrgMgr: Request organizational context
    OrgMgr->>OrgMgr: Determine user's organization
    OrgMgr->>DataFilter: Set organizational filters
    DataFilter->>PermMgr: Configure data permissions
    PermMgr->>Session: Store context in session
    Session->>Auth: Confirm context established
    
    Note over DataFilter,PermMgr: All subsequent data queries filtered by organization
```

### Session Management Sequence
```mermaid
sequenceDiagram
    participant User as User
    participant Session as Session Manager
    participant Auth as Authentication Service
    participant Monitor as Security Monitor
    
    User->>Session: Make authenticated request
    Session->>Session: Validate session token
    Session->>Session: Check session expiration
    
    alt Session Valid
        Session->>Auth: Verify permissions
        Auth->>Session: Confirm access allowed
        Session->>User: Process request
    else Session Expired
        Session->>Monitor: Log session expiration
        Session->>User: Redirect to login
    else Suspicious Activity
        Session->>Monitor: Log security event
        Monitor->>Session: Terminate session
        Session->>User: Security logout
    end
```

## State Diagrams

### User Authentication States
```mermaid
stateDiagram-v2
    [*] --> LoggedOut: Initial State
    LoggedOut --> Authenticating: Login Attempt
    Authenticating --> Authenticated: Valid Credentials
    Authenticating --> Failed: Invalid Credentials
    Authenticating --> Locked: Too Many Failures
    
    Failed --> LoggedOut: Return to Login
    Locked --> LoggedOut: Lockout Period Expires
    
    Authenticated --> Active: Session Established
    Active --> Idle: User Inactive
    Idle --> Active: User Activity Detected
    
    Active --> Expired: Session Timeout
    Idle --> Expired: Idle Timeout
    Expired --> LoggedOut: Session Ended
    
    Active --> Terminated: Security Event
    Terminated --> LoggedOut: Forced Logout
    
    LoggedOut --> [*]: User Leaves
```

### Session Lifecycle States
```mermaid
stateDiagram-v2
    [*] --> Created: Authentication Success
    Created --> Active: User Activity
    Active --> Refreshed: Token Refresh
    Refreshed --> Active: Continue Session
    
    Active --> Warning: Approaching Timeout
    Warning --> Active: User Activity
    Warning --> Expired: No Activity
    
    Active --> Suspended: Suspicious Activity
    Suspended --> Active: Security Cleared
    Suspended --> Terminated: Security Threat
    
    Expired --> [*]: Session Cleanup
    Terminated --> [*]: Security Cleanup
```

### Password Security States
```mermaid
stateDiagram-v2
    [*] --> Temporary: Initial Setup
    Temporary --> Valid: User Sets Password
    Valid --> Expiring: Approaching Expiration
    Expiring --> Valid: Password Updated
    Expiring --> Expired: Time Limit Reached
    
    Valid --> Compromised: Security Breach
    Compromised --> Reset: Password Reset
    Reset --> Valid: New Password Set
    
    Expired --> Reset: Force Password Change
    Valid --> Reset: User Requests Change
    
    Reset --> [*]: Account Deactivated
    Expired --> [*]: Account Locked
```

## Activity Diagrams

### Daily Authentication Management Workflow
```mermaid
flowchart TD
    Start([Start Daily Auth Management]) --> CheckSecurity[Check Security Events]
    CheckSecurity --> ReviewFailures[Review Failed Login Attempts]
    ReviewFailures --> AnalyzePatterns[Analyze Attack Patterns]
    AnalyzePatterns --> SecurityThreats{Threats Detected?}
    
    SecurityThreats -->|Yes| InvestigateThreats[Investigate Security Threats]
    SecurityThreats -->|No| MonitorSessions[Monitor Active Sessions]
    
    InvestigateThreats --> CategorizeThreats[Categorize Threat Types]
    CategorizeThreats --> BruteForce{Brute Force Attacks?}
    BruteForce -->|Yes| ImplementRateLimit[Implement Rate Limiting]
    BruteForce -->|No| SuspiciousActivity{Suspicious Activity?}
    
    SuspiciousActivity -->|Yes| TerminateSessions[Terminate Suspicious Sessions]
    SuspiciousActivity -->|No| MonitorSessions
    
    ImplementRateLimit --> NotifyAdmins[Notify Administrators]
    TerminateSessions --> NotifyUsers[Notify Affected Users]
    
    MonitorSessions --> CheckExpiredSessions[Check Expired Sessions]
    CheckExpiredSessions --> CleanupSessions[Cleanup Expired Sessions]
    CleanupSessions --> ValidateTokens[Validate Active Tokens]
    ValidateTokens --> OptimizePerformance[Optimize Auth Performance]
    
    NotifyAdmins --> GenerateReports[Generate Security Reports]
    NotifyUsers --> GenerateReports
    OptimizePerformance --> GenerateReports
    
    GenerateReports --> UpdatePolicies[Update Security Policies]
    UpdatePolicies --> End([Complete Auth Management])
```

### User Onboarding Authentication Setup
```mermaid
flowchart TD
    NewUser([New User Created]) --> DetermineRole[Determine User Role]
    DetermineRole --> GenerateCredentials[Generate Initial Credentials]
    GenerateCredentials --> SetTempPassword[Set Temporary Password]
    SetTempPassword --> AssignOrganization[Assign to Organization]
    AssignOrganization --> ConfigurePermissions[Configure Role Permissions]
    ConfigurePermissions --> SendWelcomeEmail[Send Welcome Email]
    SendWelcomeEmail --> UserFirstLogin[User First Login]
    UserFirstLogin --> ForcePasswordChange[Force Password Change]
    ForcePasswordChange --> ValidateNewPassword[Validate New Password]
    ValidateNewPassword --> EncryptPassword[Encrypt with SHA256]
    EncryptPassword --> EstablishSession[Establish User Session]
    EstablishSession --> SetupComplete[Setup Complete]
```

## Use Case Diagrams

### Multi-Role Authentication Use Cases
```mermaid
graph LR
    AuthSystem((Authentication System))
    
    AuthSystem --> UC1[Authenticate Super Admin]
    AuthSystem --> UC2[Authenticate Sub-Admin]
    AuthSystem --> UC3[Authenticate Instructor]
    AuthSystem --> UC4[Authenticate Student]
    AuthSystem --> UC5[Manage Sessions]
    AuthSystem --> UC6[Handle Password Security]
    AuthSystem --> UC7[Enforce Organizational Isolation]
    
    UC1 --> GlobalAccess[Global Access Control]
    UC2 --> OrgAccess[Organizational Access Control]
    UC3 --> StudentAccess[Student-Filtered Access]
    UC4 --> ContentAccess[Content Access Control]
    UC5 --> SessionMgmt[Session Management]
    UC6 --> PasswordMgmt[Password Management]
    UC7 --> DataIsolation[Data Isolation Service]
```

### Security and Compliance Use Cases
```mermaid
graph TB
    SecuritySystem((Security System))
    
    SecuritySystem --> UC1[Monitor Login Attempts]
    SecuritySystem --> UC2[Detect Suspicious Activity]
    SecuritySystem --> UC3[Enforce Account Lockouts]
    SecuritySystem --> UC4[Manage Password Policies]
    SecuritySystem --> UC5[Audit Authentication Events]
    SecuritySystem --> UC6[Handle Security Incidents]
    
    UC1 --> LoginMonitor[Login Monitor]
    UC2 --> ThreatDetector[Threat Detector]
    UC3 --> AccountLocker[Account Locker]
    UC4 --> PolicyEnforcer[Policy Enforcer]
    UC5 --> AuditLogger[Audit Logger]
    UC6 --> IncidentHandler[Incident Handler]
```

## Component Interaction Diagrams

### Authentication System Architecture
```mermaid
graph TB
    subgraph "Authentication Layer"
        LoginHandler[Login Handler]
        PasswordManager[Password Manager]
        TokenGenerator[Token Generator]
        SessionManager[Session Manager]
    end
    
    subgraph "Authorization Layer"
        RoleManager[Role Manager]
        PermissionEngine[Permission Engine]
        OrgContextManager[Org Context Manager]
        DataFilter[Data Filter]
    end
    
    subgraph "Security Layer"
        Encryptor[SHA256 Encryptor]
        SecurityMonitor[Security Monitor]
        AuditLogger[Audit Logger]
        ThreatDetector[Threat Detector]
    end
    
    subgraph "Data Layer"
        UserDB[(User Database)]
        SessionDB[(Session Database)]
        AuditDB[(Audit Database)]
        SecurityDB[(Security Database)]
    end
    
    LoginHandler --> RoleManager
    PasswordManager --> Encryptor
    TokenGenerator --> SessionManager
    SessionManager --> PermissionEngine
    
    RoleManager --> OrgContextManager
    PermissionEngine --> DataFilter
    OrgContextManager --> SecurityMonitor
    DataFilter --> AuditLogger
    
    Encryptor --> UserDB
    SecurityMonitor --> SessionDB
    AuditLogger --> AuditDB
    ThreatDetector --> SecurityDB
```

### Multi-Tenant Authentication Flow
```mermaid
graph LR
    subgraph "User Request"
        UserLogin[User Login]
        CredentialValidation[Credential Validation]
        RoleIdentification[Role Identification]
    end
    
    subgraph "Organizational Context"
        OrgDetermination[Organization Determination]
        ContextSetup[Context Setup]
        DataScoping[Data Scoping]
    end
    
    subgraph "Session Establishment"
        TokenGeneration[Token Generation]
        PermissionAssignment[Permission Assignment]
        SessionCreation[Session Creation]
    end
    
    UserLogin --> CredentialValidation
    CredentialValidation --> RoleIdentification
    RoleIdentification --> OrgDetermination
    
    OrgDetermination --> ContextSetup
    ContextSetup --> DataScoping
    DataScoping --> TokenGeneration
    
    TokenGeneration --> PermissionAssignment
    PermissionAssignment --> SessionCreation
    SessionCreation --> UserLogin
```

### Security Monitoring Integration
```mermaid
graph TB
    subgraph "Authentication Events"
        LoginAttempts[Login Attempts]
        PasswordChanges[Password Changes]
        SessionActivity[Session Activity]
    end
    
    subgraph "Security Analysis"
        PatternAnalyzer[Pattern Analyzer]
        ThreatAssessment[Threat Assessment]
        RiskCalculator[Risk Calculator]
    end
    
    subgraph "Response Actions"
        AlertGenerator[Alert Generator]
        AccountLocker[Account Locker]
        SessionTerminator[Session Terminator]
    end
    
    LoginAttempts --> PatternAnalyzer
    PasswordChanges --> ThreatAssessment
    SessionActivity --> RiskCalculator
    
    PatternAnalyzer --> AlertGenerator
    ThreatAssessment --> AccountLocker
    RiskCalculator --> SessionTerminator
    
    AlertGenerator --> LoginAttempts
    AccountLocker --> PasswordChanges
    SessionTerminator --> SessionActivity
```