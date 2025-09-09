# Claude Code Sub-Agent: Requirements Specialist

A specialized agent for requirements gathering and business analysis in spec-driven development.

## Agent Configuration

```json
{
  "name": "requirements-specialist",
  "description": "Specializes in business analysis, requirements gathering, and user story creation for spec-driven development workflows",
  "type": "business-analyst",
  "capabilities": [
    {
      "name": "Requirements Analysis",
      "description": "Analyze user needs and business requirements"
    },
    {
      "name": "User Story Creation", 
      "description": "Create user stories with EARS criteria"
    },
    {
      "name": "Stakeholder Communication",
      "description": "Handle requirements validation and approval workflows"
    }
  ],
  "tools": ["read", "write", "glob", "bash"],
  "prompt_template": "You are a Requirements Specialist agent for spec-driven development. Your role is to:\n1. Analyze user needs and business objectives\n2. Create comprehensive requirements documents\n3. Generate user stories with EARS criteria\n4. Align requirements with project vision\n5. Handle requirements validation\n\nAlways follow the spec workflow: Requirements → Design → Tasks → Implementation\n\nCurrent project context: {project_context}\n\nUser request: {user_request}"
}
```

## Agent Implementation

```typescript
class RequirementsSpecialistAgent {
  async execute(userRequest: string, projectContext: any): Promise<string> {
    // 1. Analyze user request and project context
    const analysis = await this.analyzeRequirements(userRequest, projectContext);
    
    // 2. Check for existing steering documents
    const steeringContext = await this.loadSteeringContext(projectContext.projectPath);
    
    // 3. Generate requirements document
    const requirements = await this.generateRequirements(analysis, steeringContext);
    
    // 4. Create requirements.md file
    await this.createRequirementsDocument(projectContext.projectPath, requirements);
    
    return `Requirements document created successfully at: ${projectContext.projectPath}/.spec-workflow/specs/${projectContext.specName}/requirements.md`;
  }
  
  private async analyzeRequirements(request: string, context: any) {
    // Analyze the user's request and extract requirements
    return {
      featureName: this.extractFeatureName(request),
      userStories: this.generateUserStories(request),
      businessObjectives: this.extractBusinessObjectives(request),
      constraints: this.extractConstraints(request)
    };
  }
  
  private async loadSteeringContext(projectPath: string) {
    // Load existing steering documents if they exist
    const steeringPath = `${projectPath}/.spec-workflow/steering`;
    
    try {
      const product = await this.readFile(`${steeringPath}/product.md`);
      const tech = await this.readFile(`${steeringPath}/tech.md`);
      const structure = await this.readFile(`${steeringPath}/structure.md`);
      
      return { product, tech, structure, hasDocuments: true };
    } catch {
      return { hasDocuments: false };
    }
  }
  
  private async generateRequirements(analysis: any, steeringContext: any) {
    // Generate requirements content using template
    const template = `# Requirements Document

## Introduction

${analysis.featureDescription}

## Alignment with Product Vision

${steeringContext.product ? 'This feature aligns with the product vision documented in steering documents.' : 'Feature supports overall business objectives.'}

## Requirements

${analysis.userStories.map((story: string, index: number) => `
### Requirement ${index + 1}

**User Story:** ${story}

#### Acceptance Criteria

1. WHEN [user action] THEN [system response]
2. IF [precondition] THEN [system response]
3. WHEN [user action] AND [condition] THEN [system response]
`).join('\n')}

## Non-Functional Requirements

### Code Architecture and Modularity
- **Single Responsibility Principle**: Each file should have a single, well-defined purpose
- **Modular Design**: Components, utilities, and services should be isolated and reusable
- **Dependency Management**: Minimize interdependencies between modules
- **Clear Interfaces**: Define clean contracts between components and layers

### Performance
- [Performance requirements to be specified]

### Security
- [Security requirements to be specified]

### Reliability
- [Reliability requirements to be specified]

### Usability
- [Usability requirements to be specified]
`;
    
    return template;
  }
  
  private async createRequirementsDocument(projectPath: string, content: string) {
    // Create the .spec-workflow directory structure
    const specDir = `${projectPath}/.spec-workflow/specs/${this.specName}`;
    await this.createDirectory(specDir);
    
    // Write requirements.md
    await this.writeFile(`${specDir}/requirements.md`, content);
  }
  
  private extractFeatureName(request: string): string {
    // Extract feature name from user request
    const match = request.match(/(?:create|build|implement|develop)\s+(?:a\s+)?(?:feature\s+)?([a-zA-Z0-9-\s]+)/i);
    return match ? match[1].toLowerCase().replace(/\s+/g, '-') : 'new-feature';
  }
  
  private generateUserStories(request: string): string[] {
    // Generate user stories based on the request
    return [
      "As a user, I want to [primary functionality], so that [benefit]",
      "As a user, I want to [secondary functionality], so that [benefit]",
      "As a system administrator, I want to [admin functionality], so that [benefit]"
    ];
  }
  
  private extractBusinessObjectives(request: string): string[] {
    return [
      "Improve user experience",
      "Increase system efficiency",
      "Reduce manual processes"
    ];
  }
  
  private extractConstraints(request: string): string[] {
    return [
      "Must work with existing authentication system",
      "Should be responsive and mobile-friendly",
      "Must follow existing code patterns"
    ];
  }
  
  // Helper methods for file operations
  private async readFile(path: string): Promise<string> {
    // Implementation would use Read tool
    return "";
  }
  
  private async writeFile(path: string, content: string): Promise<void> {
    // Implementation would use Write tool
  }
  
  private async createDirectory(path: string): Promise<void> {
    // Implementation would use Bash tool
  }
}
```

## Usage Instructions

### Adding to Claude Code

1. Save this agent configuration in your Claude Code agents directory
2. Configure the agent in your Claude Code settings
3. Use the agent with prompts like:

```
@requirements-specialist Create a spec for user authentication system
```

```
@requirements-specialist Analyze requirements for payment processing feature
```

### Agent Capabilities

- **Requirements Analysis**: Analyze user requests and extract business requirements
- **User Story Creation**: Generate user stories with proper EARS criteria
- **Document Generation**: Create structured requirements.md documents
- **Steering Integration**: Leverage existing project steering documents
- **Template Following**: Use established requirements templates

### Workflow Integration

This agent works as part of a three-agent workflow:

1. **Requirements Specialist** → Creates requirements.md
2. **Design Architect** → Creates design.md  
3. **Implementation Task** → Executes tasks.md

### File Structure Created

```
project-root/
└── .spec-workflow/
    └── specs/
        └── {feature-name}/
            └── requirements.md
```

## Example Usage

### Basic Requirements Creation

```
@requirements-specialist Create a spec for user authentication system with login, registration, and password reset functionality
```

### Requirements Analysis

```
@requirements-specialist Analyze the following requirements for a payment processing system and create a comprehensive requirements document: [requirements details]
```

### Integration with Steering Documents

```
@requirements-specialist Create requirements for new feature, leveraging existing product vision and technical standards
```

## Configuration Options

The agent can be configured with:

- **Template Path**: Custom requirements template location
- **Steering Document Path**: Location of project steering documents
- **Output Format**: Markdown or other documentation formats
- **Validation Rules**: Requirements validation criteria
- **Approval Workflow**: Integration with approval processes

## Error Handling

The agent handles common errors:

- **Missing Project Structure**: Automatically creates .spec-workflow directory
- **Invalid Feature Names**: Validates and normalizes feature names
- **Template Issues**: Falls back to default templates
- **Permission Issues**: Provides guidance on file permissions

## Best Practices

1. **Clear Requirements**: Provide specific, measurable requirements
2. **User-Centric**: Focus on user needs and business value
3. **Feasible**: Ensure requirements are technically achievable
4. **Testable**: Make requirements verifiable with acceptance criteria
5. **Prioritized**: Order requirements by importance and dependencies