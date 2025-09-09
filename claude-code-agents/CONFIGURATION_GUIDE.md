# Claude Code Sub-Agents Configuration Guide

This guide provides comprehensive instructions for configuring and using the three Claude Code sub-agents for spec-driven development.

## Overview

The three sub-agents work together to provide a complete spec-driven development workflow:

1. **Requirements Specialist** (`requirements-specialist`) - Creates requirements.md
2. **Design Architect** (`design-architect`) - Creates design.md  
3. **Implementation Task Agent** (`implementation-task`) - Executes tasks.md

## Quick Start

### 1. Copy Agent Files

Copy the agent configuration files to your Claude Code agents directory:

```bash
# Copy all agent files
cp claude-code-agents/*.md ~/.config/claude-code/agents/

# Or copy individual agents
cp claude-code-agents/requirements-specialist.md ~/.config/claude-code/agents/
cp claude-code-agents/design-architect.md ~/.config/claude-code/agents/
cp claude-code-agents/implementation-task.md ~/.config/claude-code/agents/
```

### 2. Copy Template Files (Optional but Recommended)

The templates are used by the agents to generate consistent documents. You have two options:

#### Option A: Copy to Claude Code Configuration (Global)
```bash
# Create templates directory in Claude Code config
mkdir -p ~/.config/claude-code/templates

# Copy all templates
cp -r claude-code-agents/templates/* ~/.config/claude-code/templates/

# This creates:
# ~/.config/claude-code/templates/requirements/template.md
# ~/.config/claude-code/templates/design/template.md
# ~/.config/claude-code/templates/tasks/template.md
# ~/.config/claude-code/templates/product/template.md
# ~/.config/claude-code/templates/tech/template.md
# ~/.config/claude-code/templates/structure/template.md
```

#### Option B: Copy to Project Directory (Project-Specific)
```bash
# Navigate to your project directory
cd /path/to/your/project

# Create .spec-workflow directory structure
mkdir -p .spec-workflow/templates

# Copy templates to project
cp -r claude-code-agents/templates/* .spec-workflow/templates/

# This creates:
# .spec-workflow/templates/requirements/template.md
# .spec-workflow/templates/design/template.md
# .spec-workflow/templates/tasks/template.md
# .spec-workflow/templates/product/template.md
# .spec-workflow/templates/tech/template.md
# .spec-workflow/templates/structure/template.md
```

### 3. Copy Steering Files (Optional but Recommended)

Steering files provide project context for the agents. Copy them to your project:

```bash
# Navigate to your project directory
cd /path/to/your/project

# Create steering directory
mkdir -p .spec-workflow/steering

# Copy steering documents
cp claude-code-agents/steering/* .spec-workflow/steering/

# This creates:
# .spec-workflow/steering/product.md
# .spec-workflow/steering/tech.md
# .spec-workflow/steering/structure.md
```

### 4. Configure Claude Code

Add the agents to your Claude Code configuration:

```json
{
  "agents": {
    "requirements-specialist": {
      "enabled": true,
      "config": {
        "templatePath": "~/.config/claude-code/templates",
        "steeringDocumentPath": ".spec-workflow/steering"
      }
    },
    "design-architect": {
      "enabled": true,
      "config": {
        "templatePath": "~/.config/claude-code/templates",
        "codeAnalysisDepth": "deep"
      }
    },
    "implementation-task": {
      "enabled": true,
      "config": {
        "testFramework": "auto",
        "codeStyle": "standard"
      }
    }
  }
}
```

## File Organization Options

### Global vs Project-Specific Setup

#### Global Setup (Recommended for Multiple Projects)
- **Templates**: Stored in `~/.config/claude-code/templates/`
- **Agents**: Stored in `~/.config/claude-code/agents/`
- **Benefits**: Share templates across all projects
- **Best for**: Developers working on multiple similar projects

#### Project-Specific Setup (Recommended for Project-Specific Needs)
- **Templates**: Stored in `.spec-workflow/templates/`
- **Steering**: Stored in `.spec-workflow/steering/`
- **Benefits**: Custom templates per project
- **Best for**: Projects with unique requirements or team standards

### Complete Directory Structure

After following the setup instructions, your directories will look like:

#### Claude Code Configuration (Global)
```
~/.config/claude-code/
├── agents/
│   ├── requirements-specialist.md
│   ├── design-architect.md
│   └── implementation-task.md
└── templates/
    ├── requirements/
    │   └── template.md
    ├── design/
    │   └── template.md
    ├── tasks/
    │   └── template.md
    ├── product/
    │   └── template.md
    ├── tech/
    │   └── template.md
    └── structure/
        └── template.md
```

#### Project Directory (Project-Specific)
```
your-project/
├── .spec-workflow/
│   ├── templates/
│   │   ├── requirements/
│   │   │   └── template.md
│   │   ├── design/
│   │   │   └── template.md
│   │   ├── tasks/
│   │   │   └── template.md
│   │   ├── product/
│   │   │   └── template.md
│   │   ├── tech/
│   │   │   └── template.md
│   │   └── structure/
│   │       └── template.md
│   ├── steering/
│   │   ├── product.md
│   │   ├── tech.md
│   │   └── structure.md
│   └── specs/
│       └── {feature-name}/
│           ├── requirements.md
│           ├── design.md
│           └── tasks.md
└── [your existing project files]
```

## Configuration Path Resolution

The agents will look for templates and steering files in this order:

1. **Project-specific paths** (if available):
   - Templates: `.spec-workflow/templates/{type}/template.md`
   - Steering: `.spec-workflow/steering/{document}.md`

2. **Global paths** (fallback):
   - Templates: `~/.config/claude-code/templates/{type}/template.md`
   - Steering: `.spec-workflow/steering/{document}.md`

3. **Built-in defaults** (if no templates found):
   - Agents will use built-in template structures

## Customizing Templates and Steering Files

### Customizing Templates
```bash
# Edit a specific template
nano ~/.config/claude-code/templates/requirements/template.md

# Or edit project-specific template
nano .spec-workflow/templates/requirements/template.md
```

### Customizing Steering Files
```bash
# Edit project steering documents
nano .spec-workflow/steering/product.md
nano .spec-workflow/steering/tech.md
nano .spec-workflow/steering/structure.md
```

### Template Customization Tips
- Keep the overall structure intact
- Modify placeholders and examples to match your project
- Add project-specific sections as needed
- Maintain consistency across templates

## Troubleshooting File Setup

### Common Issues

1. **Template Not Found**
   ```bash
   # Check if templates exist in expected location
   ls -la ~/.config/claude-code/templates/requirements/
   ls -la .spec-workflow/templates/requirements/
   ```

2. **Steering Document Not Found**
   ```bash
   # Check steering directory structure
   ls -la .spec-workflow/steering/
   ```

3. **Permission Issues**
   ```bash
   # Fix permissions
   chmod 755 ~/.config/claude-code/templates/
   chmod 755 .spec-workflow/
   ```

### Verification Commands
```bash
# Verify agent files are in place
ls -la ~/.config/claude-code/agents/

# Verify template structure
find ~/.config/claude-code/templates/ -name "template.md"

# Verify project structure
find .spec-workflow/ -name "*.md"
```

## Basic Usage

```bash
# Create requirements for a new feature
@requirements-specialist Create a spec for user authentication system

# Create technical design
@design-architect Design the user authentication system architecture

# Execute implementation tasks
@implementation-task Execute the next implementation task
```

## Detailed Configuration

### Requirements Specialist Configuration

#### Basic Configuration
```json
{
  "name": "requirements-specialist",
  "enabled": true,
  "config": {
    "templatePath": "~/.config/claude-code/templates/requirements",
    "steeringDocumentPath": ".spec-workflow/steering",
    "outputFormat": "markdown",
    "validationRules": "strict"
  }
}
```

#### Advanced Options
```json
{
  "config": {
    "autoGenerateUserStories": true,
    "userStoryTemplate": "EARS",
    "includeNonFunctional": true,
    "approvalWorkflow": "enabled",
    "stakeholderEmails": ["product@example.com", "tech-lead@example.com"]
  }
}
```

### Design Architect Configuration

#### Basic Configuration
```json
{
  "name": "design-architect",
  "enabled": true,
  "config": {
    "templatePath": "~/.config/claude-code/templates/design",
    "codeAnalysisDepth": "deep",
    "diagramFormat": "mermaid"
  }
}
```

#### Advanced Options
```json
{
  "config": {
    "architectureStandards": ["clean-architecture", "microservices"],
    "codeReuseAnalysis": true,
    "performanceAnalysis": true,
    "securityAnalysis": true,
    "generateDiagrams": true,
    "integrationPoints": ["database", "api", "external-services"]
  }
}
```

### Implementation Task Agent Configuration

#### Basic Configuration
```json
{
  "name": "implementation-task",
  "enabled": true,
  "config": {
    "testFramework": "auto",
    "codeStyle": "standard",
    "autoGenerateTests": true
  }
}
```

#### Advanced Options
```json
{
  "config": {
    "languageSupport": ["typescript", "javascript", "python", "java"],
    "testPatterns": {
      "unit": "auto",
      "integration": "conditional",
      "e2e": "manual"
    },
    "codeQuality": {
      "linting": true,
      "formatting": true,
      "typeChecking": true
    },
    "deployment": {
      "autoDeploy": false,
      "environment": "development"
    }
  }
}
```

## Project Setup

### 1. Initialize Project Structure

```bash
# Create .spec-workflow directory
mkdir -p .spec-workflow/specs
mkdir -p .spec-workflow/steering

# Create steering documents (optional but recommended)
touch .spec-workflow/steering/product.md
touch .spec-workflow/steering/tech.md
touch .spec-workflow/steering/structure.md
```

### 2. Example Steering Documents

#### product.md
```markdown
# Product Vision

## Purpose
Build a user authentication system for web applications

## Target Users
- Web developers who need secure authentication
- End users who need account management

## Key Features
1. User registration and login
2. Password reset functionality
3. Session management
4. Multi-factor authentication support

## Business Objectives
- Reduce development time for authentication features
- Improve security standards
- Support multiple projects
```

#### tech.md
```markdown
# Technical Standards

## Architecture
- Clean architecture pattern
- Separation of concerns
- Domain-driven design

## Technology Stack
- Backend: Node.js with TypeScript
- Database: PostgreSQL
- Authentication: JWT
- Testing: Jest + Supertest

## Coding Standards
- ESLint configuration
- Prettier formatting
- TypeScript strict mode
- Conventional commits

## Security Requirements
- Password hashing with bcrypt
- JWT token expiration
- Rate limiting
- Input validation
```

#### structure.md
```markdown
# Project Structure

## Directory Layout
```
src/
├── components/     # React components
├── services/      # Business logic
├── repositories/   # Data access
├── types/         # Type definitions
├── utils/         # Utility functions
└── tests/         # Test files
```

## Naming Conventions
- Components: PascalCase (e.g., UserProfile)
- Services: camelCase (e.g., userService)
- Types: PascalCase with suffix (e.g., UserInterface)
- Files: kebab-case (e.g., user-profile.ts)

## File Organization
- One component per file
- Related types in same directory
- Test files alongside implementation
```

## Workflow Examples

### Complete Feature Development

```bash
# 1. Create requirements
@requirements-specialist Create a comprehensive spec for user authentication including registration, login, password reset, and session management

# 2. Create technical design
@design-architect Design the user authentication system architecture with clean architecture patterns, security considerations, and database schema

# 3. Execute implementation
@implementation-task Execute the next implementation task

# Continue until all tasks are complete
@implementation-task Execute all remaining tasks
```

### Design-Only Workflow

```bash
# When requirements already exist
@design-architect Design the payment processing system based on existing requirements for handling transactions, refunds, and payment methods
```

### Implementation-Only Workflow

```bash
# When design is already complete
@implementation-task Execute task impl-1 for creating the payment service interface
@implementation-task Execute task impl-2 for implementing the transaction processor
```

## Agent Communication

### Inter-Agent Context Sharing

The agents automatically share context through the `.spec-workflow` directory structure:

```
.spec-workflow/
├── specs/
│   └── {feature-name}/
│       ├── requirements.md  # Created by Requirements Specialist
│       ├── design.md         # Created by Design Architect
│       └── tasks.md          # Updated by Implementation Agent
└── steering/
    ├── product.md          # Shared context
    ├── tech.md            # Shared context
    └── structure.md       # Shared context
```

### Status Tracking

Each agent updates the workflow status:

```bash
# Check workflow status
get-agent-workflow-status workflowId="user-authentication-1234567890"

# List all active workflows
list-agent-workflows
```

## Customization

### Custom Templates

Create custom templates for each agent:

#### Requirements Template
```markdown
# Requirements Template

## Introduction
{feature_description}

## Business Objectives
{business_objectives}

## User Stories
{user_stories}

## Technical Requirements
{technical_requirements}

## Constraints
{constraints}
```

#### Design Template
```markdown
# Design Template

## Overview
{overview}

## Architecture
{architecture_diagram}

## Components
{components}

## Data Models
{data_models}

## Security Considerations
{security_analysis}
```

### Custom Configuration Files

Create project-specific configuration:

```json
// .claude-code-config.json
{
  "agents": {
    "requirements-specialist": {
      "config": {
        "templatePath": "./templates/requirements",
        "projectSpecific": true
      }
    },
    "design-architect": {
      "config": {
        "architectureStandards": ["hexagonal-architecture"],
        "databaseType": "mongodb"
      }
    }
  }
}
```

## Troubleshooting

### Common Issues

#### Agent Not Found
```bash
# Check agent installation
ls ~/.config/claude-code/agents/

# Verify configuration
cat ~/.config/claude-code/config.json
```

#### Permission Issues
```bash
# Check file permissions
ls -la .spec-workflow/

# Fix permissions
chmod 755 .spec-workflow/
chmod 644 .spec-workflow/steering/*.md
```

#### Template Issues
```bash
# Check template paths
ls ~/.config/claude-code/templates/

# Verify template syntax
cat ~/.config/claude-code/templates/requirements/template.md
```

### Debug Mode

Enable debug logging:

```bash
export DEBUG=claude-code-agents:*
export LOG_LEVEL=debug
```

### Reset Configuration

Reset to default configuration:

```bash
# Backup current config
cp ~/.config/claude-code/config.json ~/.config/claude-code/config.json.backup

# Reset configuration
rm ~/.config/claude-code/config.json
```

## Best Practices

### 1. Project Organization
- Use consistent naming conventions
- Maintain clean directory structure
- Keep steering documents updated
- Version control your configuration

### 2. Agent Usage
- Use agents in sequence (requirements → design → implementation)
- Provide clear, specific requirements
- Review generated documents before proceeding
- Monitor progress regularly

### 3. Quality Assurance
- Test implementations thoroughly
- Follow established patterns
- Maintain code consistency
- Document exceptions and deviations

### 4. Team Collaboration
- Share configuration across team members
- Use consistent steering documents
- Establish review processes
- Track workflow status

## Integration with CI/CD

### GitHub Actions Example

```yaml
name: Agent Workflow Validation
on:
  push:
    paths:
      - '.spec-workflow/**'

jobs:
  validate-workflow:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Validate Requirements
        run: |
          if [ -f ".spec-workflow/specs/*/requirements.md" ]; then
            echo "Requirements document found"
          fi
      - name: Validate Design
        run: |
          if [ -f ".spec-workflow/specs/*/design.md" ]; then
            echo "Design document found"
          fi
      - name: Check Implementation Progress
        run: |
          if [ -f ".spec-workflow/specs/*/tasks.md" ]; then
            echo "Tasks document found"
          fi
```

## Advanced Features

### Multi-Project Support

Configure agents for multiple projects:

```json
{
  "agents": {
    "requirements-specialist": {
      "config": {
        "multiProject": true,
        "projectPaths": [
          "~/projects/web-app",
          "~/projects/mobile-app",
          "~/projects/api-service"
        ]
      }
    }
  }
}
```

### Custom Agent Commands

Extend agents with custom commands:

```json
{
  "agents": {
    "requirements-specialist": {
      "commands": {
        "create-epic": "Create epic-level requirements",
        "generate-user-stories": "Generate user stories from epic",
        "validate-requirements": "Validate requirements completeness"
      }
    }
  }
}
```

### Integration with External Tools

Connect with external project management tools:

```json
{
  "agents": {
    "implementation-task": {
      "config": {
        "jiraIntegration": {
          "enabled": true,
          "baseUrl": "https://your-domain.atlassian.net",
          "project": "PROJ"
        },
        "githubIntegration": {
          "enabled": true,
          "repository": "your-org/your-repo",
          "autoCreatePRs": true
        }
      }
    }
  }
}
```

## Support and Resources

### Documentation
- [Claude Code Documentation](https://docs.anthropic.com/claude-code)
- [Agent Configuration Reference](https://docs.anthropic.com/claude-code/agents)
- [Best Practices Guide](https://docs.anthropic.com/claude-code/best-practices)

### Community
- [Claude Code Community Forum](https://community.anthropic.com)
- [GitHub Issues](https://github.com/anthropics/claude-code/issues)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/claude-code)

### Getting Help
1. Check this configuration guide
2. Review agent-specific documentation
3. Search community forums
4. Create GitHub issue with detailed description

This configuration guide provides everything you need to successfully set up and use the Claude Code sub-agents for spec-driven development in your projects.