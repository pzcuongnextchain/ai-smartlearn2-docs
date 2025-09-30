# AI Smart Learning Renewal - Database Schema (ERD)

## Entity Relationship Diagram

### Core Entities Overview

Based on docs.txt specifications, the system requires the following core entities:

1. **Users** - Super Admin, Sub-Admin, Instructors, Students (with company isolation)
2. **Content** - Videos, Sessions, Assignments
3. **AI Tools** - Practice tools and configurations
4. **Conversations** - AI conversation history per assignment/session
5. **File Uploads** - File management system

## Detailed Entity Definitions

### 1. Users Table (Multi-role with company isolation)

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    company_name VARCHAR(255), -- Sub-admin's company (for isolation)
    sub_admin_id BIGINT, -- Instructor belongs to sub-admin, NULL for super_admin and sub_admin
    name VARCHAR(255) NOT NULL,
    username VARCHAR(100) NOT NULL UNIQUE,
    email VARCHAR(255),
    password_hash VARCHAR(64) NOT NULL, -- SHA256 hash
    role ENUM('super_admin', 'sub_admin', 'instructor', 'student') NOT NULL,
    login_token VARCHAR(255), -- Required for students per docs.txt
    course_schedule DATE, -- For students
    assigned_instructor_id BIGINT, -- For students
    logo_url VARCHAR(500), -- Sub-admin logo (displayed to users)
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE,

    FOREIGN KEY (sub_admin_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (assigned_instructor_id) REFERENCES users(id) ON DELETE SET NULL,

    INDEX idx_company_role (company_name, role),
    INDEX idx_sub_admin (sub_admin_id),
    INDEX idx_login_token (login_token),
    INDEX idx_instructor (assigned_instructor_id),
    INDEX idx_role (role)
);
```

### 2. Practice Sessions Table (1-40 sessions per docs.txt)

```sql
CREATE TABLE practice_sessions (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    sub_admin_id BIGINT NOT NULL, -- Owner sub-admin
    session_number INT NOT NULL CHECK (session_number BETWEEN 1 AND 40),
    title VARCHAR(255) NOT NULL,
    web_url VARCHAR(500) NOT NULL, -- Different URL per sub-administrator
    pre_practice_message TEXT, -- HTML content for pre-practice guidance
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE,

    FOREIGN KEY (sub_admin_id) REFERENCES users(id) ON DELETE CASCADE,

    UNIQUE KEY unique_session_admin (session_number, sub_admin_id),
    INDEX idx_admin_session (sub_admin_id, session_number)
);
```

### 3. Assignments Table (1-10 assignments per docs.txt)

```sql
CREATE TABLE assignments (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    sub_admin_id BIGINT NOT NULL, -- Owner sub-admin
    assignment_number INT NOT NULL CHECK (assignment_number BETWEEN 1 AND 10),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    web_url VARCHAR(500) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE,

    FOREIGN KEY (sub_admin_id) REFERENCES users(id) ON DELETE CASCADE,

    UNIQUE KEY unique_assignment_admin (assignment_number, sub_admin_id),
    INDEX idx_admin_assignment (sub_admin_id, assignment_number)
);
```

### 4. AI Tools Master Table

```sql
CREATE TABLE ai_tools_master (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    provider VARCHAR(100) NOT NULL, -- 'eden_ai', 'openai', 'naver', 'kakao', etc.
    tool_type VARCHAR(100) NOT NULL, -- 'chat', 'image_generation', 'translation', etc.
    api_endpoint VARCHAR(500),
    description TEXT,
    is_enabled_globally BOOLEAN DEFAULT TRUE, -- Super admin control
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    INDEX idx_provider (provider),
    INDEX idx_type (tool_type),
    INDEX idx_enabled (is_enabled_globally)
);
```

### 5. Sub-Admin AI Tool Configuration

```sql
CREATE TABLE sub_admin_ai_tools (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    sub_admin_id BIGINT NOT NULL,
    ai_tool_id BIGINT NOT NULL,
    is_enabled BOOLEAN DEFAULT FALSE, -- Sub-admin control
    api_key_encrypted VARCHAR(500), -- Encrypted API key per sub-admin
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (sub_admin_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (ai_tool_id) REFERENCES ai_tools_master(id) ON DELETE CASCADE,

    UNIQUE KEY unique_admin_tool (sub_admin_id, ai_tool_id),
    INDEX idx_admin_enabled (sub_admin_id, is_enabled)
);
```

### 6. Session AI Tool Configuration

```sql
CREATE TABLE session_ai_tools (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    practice_session_id BIGINT NOT NULL,
    ai_tool_id BIGINT NOT NULL,
    is_enabled BOOLEAN DEFAULT TRUE,
    display_order INT DEFAULT 0,

    FOREIGN KEY (practice_session_id) REFERENCES practice_sessions(id) ON DELETE CASCADE,
    FOREIGN KEY (ai_tool_id) REFERENCES ai_tools_master(id) ON DELETE CASCADE,

    UNIQUE KEY unique_session_tool (practice_session_id, ai_tool_id),
    INDEX idx_session_order (practice_session_id, display_order)
);
```

### 7. Assignment AI Tool Configuration

```sql
CREATE TABLE assignment_ai_tools (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    assignment_id BIGINT NOT NULL,
    ai_tool_id BIGINT NOT NULL,
    is_enabled BOOLEAN DEFAULT TRUE,
    display_order INT DEFAULT 0,

    FOREIGN KEY (assignment_id) REFERENCES assignments(id) ON DELETE CASCADE,
    FOREIGN KEY (ai_tool_id) REFERENCES ai_tools_master(id) ON DELETE CASCADE,

    UNIQUE KEY unique_assignment_tool (assignment_id, ai_tool_id),
    INDEX idx_assignment_order (assignment_id, display_order)
);
```

### 8. Recommended Videos Table

```sql
CREATE TABLE recommended_videos (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    sub_admin_id BIGINT NOT NULL, -- Owner sub-admin
    session_number INT NOT NULL CHECK (session_number BETWEEN 1 AND 40),
    title VARCHAR(255) NOT NULL,
    video_url VARCHAR(500) NOT NULL,
    thumbnail_url VARCHAR(500),
    script_content TEXT, -- For AI recommendations
    hashtags TEXT, -- Comma-separated tags for AI matching
    display_order INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE,

    FOREIGN KEY (sub_admin_id) REFERENCES users(id) ON DELETE CASCADE,

    INDEX idx_admin_session (sub_admin_id, session_number),
    INDEX idx_active_order (is_active, display_order),
    FULLTEXT idx_content_search (title, script_content, hashtags)
);
```

### 9. File Uploads Table (Based on your TypeORM entity)

```sql
CREATE TABLE file_uploads (
    id VARCHAR(36) PRIMARY KEY, -- UUID
    uploader_id BIGINT,
    url VARCHAR(500) NOT NULL,
    file_name VARCHAR(255),
    file_type VARCHAR(50),
    file_size INT,
    access_mode VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (uploader_id) REFERENCES users(id) ON DELETE SET NULL,

    INDEX idx_uploader (uploader_id),
    INDEX idx_file_type (file_type),
    INDEX idx_access_mode (access_mode)
);
```

### 10. Conversations Table (One conversation per assignment/session)

```sql
CREATE TABLE conversations (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    student_id BIGINT NOT NULL,
    conversation_type ENUM('practice_session', 'assignment', 'video_recommendation') NOT NULL,
    session_id BIGINT, -- practice_session_id or assignment_id
    ai_tool_id BIGINT NOT NULL,
    title VARCHAR(255),
    started_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_message_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE,

    FOREIGN KEY (student_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (ai_tool_id) REFERENCES ai_tools_master(id) ON DELETE CASCADE,

    UNIQUE KEY unique_student_session (student_id, conversation_type, session_id),
    INDEX idx_student_type (student_id, conversation_type),
    INDEX idx_last_message (last_message_at)
);
```

### 11. Conversation Messages Table (Chat history)

```sql
CREATE TABLE conversation_messages (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    conversation_id BIGINT NOT NULL,
    message_type ENUM('user_message', 'ai_response', 'system_message') NOT NULL,
    message_content TEXT NOT NULL,
    prompt_analysis TEXT, -- GPT API analysis of user prompts
    improvement_suggestions TEXT, -- AI feedback on prompts
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (conversation_id) REFERENCES conversations(id) ON DELETE CASCADE,

    INDEX idx_conversation_time (conversation_id, created_at),
    INDEX idx_message_type (message_type),
    FULLTEXT idx_message_search (message_content)
);
```

### 12. Student Video Recommendations Table

```sql
CREATE TABLE student_video_recommendations (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    student_id BIGINT NOT NULL,
    session_number INT NOT NULL,
    recommended_video_id BIGINT NOT NULL,
    conversation_id BIGINT, -- Reference to the conversation that generated this recommendation
    recommendation_reason TEXT, -- AI explanation for recommendation
    is_selected BOOLEAN DEFAULT FALSE, -- Student selected this video
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (student_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (recommended_video_id) REFERENCES recommended_videos(id) ON DELETE CASCADE,
    FOREIGN KEY (conversation_id) REFERENCES conversations(id) ON DELETE SET NULL,

    INDEX idx_student_session (student_id, session_number),
    INDEX idx_selected (is_selected)
);
```

## Entity Relationships Summary

### Primary Relationships

1. **Users** (Sub-Admin) → (N) **Practice Sessions** - Session management per sub-admin
2. **Users** (Sub-Admin) → (N) **Assignments** - Assignment management per sub-admin
3. **Users** (Sub-Admin) → (N) **Recommended Videos** - Video content per sub-admin
4. **Users** (Student) → (N) **Conversations** - Student conversation tracking
5. **Conversations** → (N) **Conversation Messages** - Chat history per conversation
6. **AI Tools Master** → (N) **Sub-Admin AI Tools** - Tool configuration per sub-admin
7. **Practice Sessions** → (N) **Session AI Tools** - Tools available per session
8. **Assignments** → (N) **Assignment AI Tools** - Tools available per assignment

### Key Multi-Tenant Considerations

1. **Company Isolation**: All user data isolated by sub-admin's company
2. **Super Admin Access**: Super admin queries can access all data across companies
3. **Sub-Admin Restrictions**: All sub-admin queries filtered by their company
4. **Conversation-Based Chat Logs**: Each assignment/session has one conversation with multiple messages

### Critical Indexes for Performance

```sql
-- Multi-tenant queries
CREATE INDEX idx_company_users ON users(company_name, role, is_active);
CREATE INDEX idx_admin_sessions ON practice_sessions(sub_admin_id, is_active);
CREATE INDEX idx_admin_assignments ON assignments(sub_admin_id, is_active);

-- Conversation queries (critical for super admin chat log access)
CREATE INDEX idx_student_conversations ON conversations(student_id, conversation_type, is_active);
CREATE INDEX idx_conversation_messages ON conversation_messages(conversation_id, created_at);

-- AI tool configuration
CREATE INDEX idx_admin_tools ON sub_admin_ai_tools(sub_admin_id, is_enabled);
CREATE INDEX idx_session_tools ON session_ai_tools(practice_session_id, is_enabled);
```

## Data Access Patterns

### Super Administrator Queries

```sql
-- Access all student conversations across all companies
SELECT c.*, u.name as student_name, u.company_name, at.name as ai_tool_name
FROM conversations c
JOIN users u ON c.student_id = u.id
JOIN ai_tools_master at ON c.ai_tool_id = at.id
WHERE u.role = 'student'
ORDER BY c.last_message_at DESC;

-- View all assignments across all companies
SELECT a.*, u.name as sub_admin_name, u.company_name
FROM assignments a
JOIN users u ON a.sub_admin_id = u.id
WHERE a.is_active = TRUE;

-- Get all chat messages for a specific assignment across all companies
SELECT cm.*, c.title as conversation_title, u.name as student_name, u.company_name
FROM conversation_messages cm
JOIN conversations c ON cm.conversation_id = c.id
JOIN users u ON c.student_id = u.id
WHERE c.conversation_type = 'assignment' AND c.session_id = ?
ORDER BY cm.created_at;
```

### Sub-Administrator Queries (with company filter)

```sql
-- View students in their company only
SELECT * FROM users
WHERE company_name = ? AND role = 'student' AND is_active = TRUE;

-- View instructors that belong to this sub-admin
SELECT * FROM users
WHERE sub_admin_id = ? AND role = 'instructor' AND is_active = TRUE;

-- View conversations for their company's students only
SELECT c.*, u.name as student_name
FROM conversations c
JOIN users u ON c.student_id = u.id
WHERE u.company_name = ? AND u.role = 'student';
```

### Instructor Queries (restricted to their students only)

```sql
-- Instructor can only view their assigned students (same company)
SELECT s.* FROM users s
JOIN users i ON s.company_name = i.company_name
WHERE s.assigned_instructor_id = ? AND s.role = 'student' AND i.role = 'instructor';

-- Instructor can view conversations of their assigned students only
SELECT c.*, u.name as student_name
FROM conversations c
JOIN users u ON c.student_id = u.id
WHERE u.assigned_instructor_id = ? AND u.role = 'student';
```

### Student Learning View (학습보기) Query

```sql
-- Get AI-recommended videos for a student
SELECT rv.*, svr.recommendation_reason
FROM recommended_videos rv
LEFT JOIN student_video_recommendations svr ON rv.id = svr.recommended_video_id
    AND svr.student_id = ?
JOIN users sa ON rv.sub_admin_id = sa.id
JOIN users s ON s.company_name = sa.company_name
WHERE s.id = ? AND rv.session_number = ?
ORDER BY rv.display_order;
```

### Assignment View (과제보기) Query - Conversation-based

```sql
-- Get conversation and all messages for a specific assignment
SELECT
    c.id as conversation_id,
    c.title as conversation_title,
    cm.message_type,
    cm.message_content,
    cm.prompt_analysis,
    cm.improvement_suggestions,
    cm.created_at,
    at.name as ai_tool_name
FROM conversations c
JOIN conversation_messages cm ON c.id = cm.conversation_id
JOIN ai_tools_master at ON c.ai_tool_id = at.id
WHERE c.student_id = ?
    AND c.conversation_type = 'assignment'
    AND c.session_id = ?
ORDER BY cm.created_at;
```

### Video Recommendation via AI Chatbot Query

```sql
-- Get conversation that led to video recommendations
SELECT
    c.*,
    svr.recommended_video_id,
    rv.title as video_title,
    svr.recommendation_reason
FROM conversations c
LEFT JOIN student_video_recommendations svr ON c.id = svr.conversation_id
LEFT JOIN recommended_videos rv ON svr.recommended_video_id = rv.id
WHERE c.student_id = ?
    AND c.conversation_type = 'video_recommendation'
    AND c.session_id = ?;
```

## Key Design Benefits

### 1. Simplified Multi-Tenant Architecture

- No separate organizations table - company isolation via `company_name` in users
- Sub-admin's company determines data access scope
- Logo stored directly in user table for sub-admins

### 2. Conversation-Based Chat Logs

- One conversation per student per assignment/session
- Easy to get complete chat history for each assignment
- Supports prompt analysis and AI feedback per message
- Super admin can easily access all conversations

### 3. File Upload Integration

- Based on your TypeORM entity structure
- UUID primary key for better distribution
- Supports various file types and access modes
- Links to uploader for tracking

### 4. Removed Unauthorized Features

- ❌ No organizations table (not mentioned in docs.txt)
- ❌ No operation logs table (not mentioned in docs.txt)
- ❌ No system configuration table (not mentioned in docs.txt)
- ❌ No separate branding table (logo in user table as per docs.txt)

This simplified schema provides:

1. **Exact compliance with docs.txt** - only features mentioned are included
2. **Conversation-based chat tracking** - easier to get assignment chat history
3. **Company isolation without complex organization structure**
4. **File upload support** matching your TypeORM pattern
5. **Super admin access to all data** across companies
6. **Clean, maintainable structure** without unnecessary complexity
