# AI Smart Learning Renewal

## Project Overview

AI Smart Learning Renewal is a multi-tenant SaaS learning management platform that enables organizations to create and manage their own learning environments with AI-powered tools and video recommendations.

## Core Features

### Multi-Tenant Administrator Structure

- **Super Administrator**: Manages all data across all organizations
- **Sub-Administrators**: Manage only their organization's data (students, instructors, content)
- **Organizational Isolation**: Each sub-administrator sees only their own organization's data
- **Custom Branding**: Each organization can display their own logo to administrators and users

### AI Practice Tool Integration

- **Eden AI Integration**: Primary AI tool provider via https://www.edenai.co/
- **Additional AI APIs**: Integration with other AI tools for features not provided by Eden (e.g., video generation)
- **Assistant UI**: Modern practice tool interface via https://www.assistant-ui.com/
- **API Key Management**: Sub-administrators can register their own API keys for token cost handling

### AI-Powered Video Recommendations

- **AI Chatbot Integration**: Students receive personalized video recommendations through AI conversations
- **RAG Methodology**: Recommended videos based on script content, hashtags, and student interactions
- **Personalized Learning**: AI analyzes student conversations to suggest relevant lectures

## System Architecture

### User Roles and Access

#### Super Administrator

- Access to ALL data across ALL organizations
- Can override any sub-administrator settings
- Manages global AI tool configurations
- Views all student chat logs and AI tool usage
- Performs cross-organizational operations

#### Sub-Administrator

- Manages only their organization's data
- Cannot access other organizations' data
- Configures AI tools for their organization
- Manages students, instructors, and content within organizational scope

#### Instructor

- Views only their own students
- Limited menu access
- Basic student management capabilities

#### Student

- Accesses AI-powered learning tools
- Receives personalized video recommendations
- Uses AI practice tools for assignments and sessions

### Content Management Structure

#### Practice Sessions (1-40)

- Configurable AI practice tools per session
- Pre-practice message configuration
- Session-specific AI tool availability
- Different URLs per sub-administrator

#### Assignments (1-10)

- AI tool usage pages for assignment work
- Chat log tracking for all AI interactions
- Assignment viewing through AI tool interface

#### Video Management

- AI-recommended video lectures
- Script content and hashtag integration
- Personalized recommendations via AI chatbot

## Technical Specifications

### Authentication & Security

- **Password Encryption**: SHA256 encryption required
- **Login Tokens**: Required for all students
- **Organizational Data Isolation**: Strict separation between organizations
- **Role-Based Access Control**: Different access levels per user role

### AI Tool Integration

- **100+ AI Tools**: Comprehensive AI tool library
- **Master Control**: Super admin controls global tool availability
- **Organizational Override**: Sub-admins can enable/disable tools for their organization
- **API Key Management**: Flexible API key configuration per organization

### Data Management

- **Multi-Tenant Database**: Organizational data partitioning
- **Cross-Organizational Operations**: Super admin can transfer data between organizations
- **Chat Log Storage**: Complete AI conversation history tracking
- **Bulk Operations**: Mass user and content management capabilities

## User Interface Features

### Student Learning Interface

1. **AI Video Recommendation Process**
   - Initial AI chatbot conversation
   - Personalized video recommendations based on job/experience
   - Customizable lecture combinations (minimum 5 minutes)
   - Progress tracking through recommended content

2. **AI Practice Tool Interface**
   - Pre-practice guidance messages
   - AI tool selection (administrator-configured)
   - Real-time prompt analysis and feedback
   - Chat log recording for all interactions

### Administrator Interface

1. **Content Management**
   - Video and session management
   - AI tool configuration per session
   - Bulk content operations
   - Cross-organizational content access (Super Admin)

2. **User Management**
   - Student and instructor CRUD operations
   - Simplified user profiles (Name, ID, Password, Instructor, Course Schedule, Login Token)
   - Bulk user import/export
   - Cross-organizational user management (Super Admin)

3. **AI Tool Management**
   - Global AI tool library management
   - API key configuration
   - Usage monitoring and chat log access
   - Cost tracking and token management

## Data Access Patterns

### Student Work Viewing

- **Learning View (학습보기)**: Access to AI-recommended video lectures
- **Assignment View (과제보기)**: View AI tool usage and complete chat logs
- **Practice Session Monitoring**: Track AI tool conversations across all sessions
- **Chat Log Search**: Filter and search all student AI interactions

### Organizational Data Management

- **Complete Data Access**: Super admin sees all organizational data
- **Data Transfer**: Move students, instructors, or content between organizations
- **Bulk Operations**: Mass updates across multiple organizations
- **Chat Log Monitoring**: Access all student AI conversations globally

## Removed Features

The following features from the previous system have been removed:

- Session-based video management
- O/X Simulation tools
- VR O/X Simulation
- Pre-survey functionality
- Task instruction and task report pages (replaced with direct AI tool usage)

## Installation & Setup

### Prerequisites

- Multi-tenant database system
- AI API integrations (Eden AI, OpenAI, etc.)
- Web server with organizational routing capabilities
- SHA256 password encryption support

### Configuration

1. **Database Setup**: Configure multi-tenant database with organizational partitioning
2. **AI Integration**: Set up Eden AI and additional AI tool APIs
3. **Authentication**: Implement SHA256 password encryption and login token system
4. **Organizational Setup**: Configure sub-administrator creation and data isolation
5. **Branding**: Set up custom logo display per organization

## API Integration

### Eden AI Integration

- Primary AI tool provider
- Comprehensive AI tool library
- API key management per organization
- Usage tracking and cost monitoring

### Additional AI APIs

- Video generation tools
- Specialized AI services not available through Eden AI
- Custom integration support
- Fallback mechanisms for service availability

## Development Guidelines

### Multi-Tenant Considerations

- All database queries must include organizational context
- UI components must respect organizational data boundaries
- API endpoints must validate organizational access rights
- Chat logs must be stored with organizational association

### Security Requirements

- SHA256 password encryption mandatory
- Login token generation and validation
- Organizational data isolation enforcement
- Role-based access control implementation

### AI Tool Integration

- Modular AI tool architecture
- Configurable tool availability per session
- Chat log recording for all AI interactions
- Error handling for AI service failures

## Monitoring & Maintenance

### System Monitoring

- Basic system health checks
- AI tool availability monitoring
- Database performance tracking
- Cross-organizational operation logging

### Data Management

- Regular backup procedures
- Chat log archival policies
- Cross-organizational data integrity checks
- User activity monitoring

## Support & Documentation

### User Documentation

- Administrator setup guides
- AI tool configuration instructions
- Student learning interface guides
- Troubleshooting documentation

### Technical Documentation

- API integration guides
- Database schema documentation
- Multi-tenant architecture overview
- Security implementation guidelines

## Future Enhancements

### Planned Features

- Enhanced AI tool library expansion
- Improved cross-organizational collaboration tools
- Advanced chat log analysis capabilities
- Mobile application support

### Integration Opportunities

- Additional AI service providers
- Learning management system integrations
- Third-party authentication systems
- Advanced analytics platforms (if required)

## License & Support

This project is designed as a comprehensive multi-tenant learning management system with AI integration capabilities. For technical support and implementation guidance, refer to the detailed requirements and flow documentation in the respective feature directories.

## Project Structure

```
ai-smart-learning-renewal/
├── README.md
├── docs.txt
├── directory-structure.md
├── super-admin/
│   ├── multi-admin-management/
│   ├── global-data-management/
│   ├── ai-api-master-control/
│   └── global-branding-management/
├── sub-admin/
│   ├── user-management/
│   ├── content-management/
│   ├── ai-practice-tool-management/
│   └── branding-customization/
├── shared/
│   └── ai-integrations/
└── migration/
    └── data-migration-plan.md
```

Each directory contains detailed requirements and flow diagrams for the respective features, ensuring comprehensive documentation for development and implementation.
