# Multi-Administrator Management Flows and Diagrams

## User Flow Diagrams

### Sub-Administrator Creation Flow
```mermaid
flowchart TD
    A[Super Admin Login] --> B[Access Admin Management]
    B --> C[Click Create Sub-Administrator]
    C --> D[Fill Organization Details]
    D --> E[Set Initial Permissions]
    E --> F[Generate Admin Credentials]
    F --> G{Validation Passed?}
    G -->|No| H[Show Error Messages] --> D
    G -->|Yes| I[Create Organizational Scope]
    I --> J[Setup Database Partitions]
    J --> K[Send Welcome Email]
    K --> L[Display Success Message]
    L --> M[Add to Sub-Admin List]
```

### Data Isolation Verification Flow
```mermaid
flowchart TD
    A[Sub-Admin Login] --> B[System Validates Org ID]
    B --> C[Apply Data Filters]
    C --> D[Load Organization Dashboard]
    D --> E[Display Org-Specific Data]
    E --> F{User Attempts Cross-Org Access?}
    F -->|Yes| G[Block Access] --> H[Log Access Event]
    F -->|No| I[Continue Normal Operation]
    H --> J[Notify Super Admin]
    I --> K[Track User Activity]
```

### Super Administrator Oversight Flow
```mermaid
flowchart TD
    A[Super Admin Dashboard] --> B[Select Oversight Function]
    B --> C{Choose Action}
    C -->|View All Data| D[Aggregate Cross-Org Data]
    C -->|Manage Permissions| E[Global Permission Panel]
    C -->|Monitor Usage| F[System Analytics]
    C -->|Audit Logs| G[Security Audit Panel]
    
    D --> H[Display Unified Dashboard]
    E --> I[Update Global Settings]
    F --> J[Show Usage Metrics]
    G --> K[Review Security Events]
```

## System Architecture Diagrams

### Multi-Tenant Data Architecture
```mermaid
graph TB
    subgraph "Application Layer"
        A[Super Admin Interface]
        B[Sub-Admin Interface 1]
        C[Sub-Admin Interface 2]
        D[Sub-Admin Interface N]
    end
    
    subgraph "Business Logic Layer"
        E[Authentication Service]
        F[Authorization Service]
        G[Data Access Layer]
    end
    
    subgraph "Data Layer"
        H[(Shared Tables)]
        I[(Org 1 Partition)]
        J[(Org 2 Partition)]
        K[(Org N Partition)]
    end
    
    A --> E
    B --> E
    C --> E
    D --> E
    
    E --> F
    F --> G
    
    G --> H
    G --> I
    G --> J
    G --> K
```

### Permission Hierarchy Diagram
```mermaid
graph TD
    A[Super Administrator] --> B[Global System Access]
    A --> C[All Organizations Data]
    A --> D[Sub-Admin Management]
    A --> E[System Configuration]
    
    F[Sub-Administrator] --> G[Organization Scope Only]
    F --> H[Own Users Management]
    F --> I[Own Content Management]
    F --> J[Limited System Settings]
    
    B --> K[Override All Permissions]
    G --> L[Cannot Access Other Orgs]
    
    style A fill:#ff6b6b
    style F fill:#4ecdc4
    style K fill:#ffe66d
    style L fill:#ff8b94
```

## Sequence Diagrams

### Sub-Administrator Creation Sequence
```mermaid
sequenceDiagram
    participant SA as Super Admin
    participant UI as Admin Interface
    participant API as Backend API
    participant DB as Database
    participant Email as Email Service
    
    SA->>UI: Click "Create Sub-Admin"
    UI->>SA: Show creation form
    SA->>UI: Submit organization details
    UI->>API: POST /api/sub-admins
    API->>DB: Validate organization name
    DB->>API: Return validation result
    
    alt Organization name available
        API->>DB: Create organization record
        API->>DB: Create sub-admin user
        API->>DB: Setup data partitions
        DB->>API: Return success
        API->>Email: Send welcome email
        API->>UI: Return success response
        UI->>SA: Show success message
    else Organization name exists
        API->>UI: Return error response
        UI->>SA: Show error message
    end
```

### Data Access Control Sequence
```mermaid
sequenceDiagram
    participant SubAdmin as Sub-Administrator
    participant UI as Interface
    participant Auth as Auth Service
    participant API as Backend API
    participant DB as Database
    
    SubAdmin->>UI: Request student list
    UI->>Auth: Validate session token
    Auth->>UI: Return org_id and permissions
    UI->>API: GET /api/students (with org_id)
    API->>DB: SELECT * FROM students WHERE org_id = ?
    DB->>API: Return filtered results
    API->>UI: Return student list
    UI->>SubAdmin: Display organization students only
```

## State Diagrams

### Sub-Administrator Account States
```mermaid
stateDiagram-v2
    [*] --> Pending: Create Account
    Pending --> Active: Super Admin Approval
    Pending --> Rejected: Super Admin Rejection
    Active --> Suspended: Violation/Non-payment
    Active --> Inactive: Manual Deactivation
    Suspended --> Active: Issue Resolved
    Inactive --> Active: Manual Reactivation
    Rejected --> [*]: Account Deleted
    Inactive --> [*]: Account Purged
```

### Data Isolation States
```mermaid
stateDiagram-v2
    [*] --> Isolated: Organization Created
    Isolated --> Accessible: Valid Org Context
    Accessible --> Filtered: Apply Data Filters
    Filtered --> Displayed: Render UI
    Displayed --> Isolated: Session End
    
    Accessible --> Blocked: Invalid Access Attempt
    Blocked --> Logged: Access Event
    Logged --> Isolated: Return to Safe State
```

## Use Case Diagrams

### Super Administrator Use Cases
```mermaid
graph LR
    SuperAdmin((Super Administrator))
    
    SuperAdmin --> UC1[Create Sub-Administrator]
    SuperAdmin --> UC2[Manage Organizations]
    SuperAdmin --> UC3[View All Data]
    SuperAdmin --> UC4[Configure Global Settings]
    SuperAdmin --> UC5[Monitor System Usage]
    SuperAdmin --> UC6[Audit Security Events]
    SuperAdmin --> UC7[Manage Permissions]
    
    UC1 --> System[System]
    UC2 --> System
    UC3 --> System
    UC4 --> System
    UC5 --> System
    UC6 --> System
    UC7 --> System
```

### Sub-Administrator Constraints
```mermaid
graph LR
    SubAdmin((Sub-Administrator))
    
    SubAdmin --> UC1[Manage Own Students]
    SubAdmin --> UC2[Manage Own Instructors]
    SubAdmin --> UC3[Configure Own Content]
    SubAdmin --> UC4[View Own Analytics]
    SubAdmin --> UC5[Upload Organization Logo]
    
    UC1 --> OrgScope[Organization Scope Only]
    UC2 --> OrgScope
    UC3 --> OrgScope
    UC4 --> OrgScope
    UC5 --> OrgScope
    
    SubAdmin -.-> Blocked1[Cannot Access Other Orgs]
    SubAdmin -.-> Blocked2[Cannot Modify Global Settings]
    SubAdmin -.-> Blocked3[Cannot Create Sub-Admins]
    
    style Blocked1 fill:#ff8b94
    style Blocked2 fill:#ff8b94
    style Blocked3 fill:#ff8b94
```