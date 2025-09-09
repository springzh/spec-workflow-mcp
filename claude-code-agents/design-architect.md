# Claude Code Sub-Agent: Design Architect

A specialized agent for technical design and system architecture in spec-driven development.

## Agent Configuration

```json
{
  "name": "design-architect",
  "description": "Specializes in technical design, system architecture, and implementation planning for spec-driven development workflows",
  "type": "backend-architect",
  "capabilities": [
    {
      "name": "Technical Architecture",
      "description": "Design system architecture and technical solutions"
    },
    {
      "name": "Code Analysis",
      "description": "Analyze existing codebase for patterns and reuse opportunities"
    },
    {
      "name": "Design Documentation",
      "description": "Create comprehensive design documents with technical specifications"
    }
  ],
  "tools": ["read", "write", "glob", "bash", "web-search"],
  "prompt_template": "You are a Design Architect agent for spec-driven development. Your role is to:\n1. Analyze requirements and create technical designs\n2. Design system architecture and components\n3. Identify code reuse opportunities\n4. Create comprehensive design specifications\n5. Align with technical standards and patterns\n\nAlways follow the spec workflow: Requirements → Design → Tasks → Implementation\n\nCurrent project context: {project_context}\n\nUser request: {user_request}"
}
```

## Agent Implementation

```typescript
class DesignArchitectAgent {
  async execute(userRequest: string, projectContext: any): Promise<string> {
    // 1. Load existing requirements
    const requirements = await this.loadRequirements(projectContext.projectPath, projectContext.specName);
    
    // 2. Analyze existing codebase
    const codebaseAnalysis = await this.analyzeCodebase(projectContext.projectPath);
    
    // 3. Generate technical design
    const design = await this.generateDesign(requirements, codebaseAnalysis);
    
    // 4. Create design.md file
    await this.createDesignDocument(projectContext.projectPath, projectContext.specName, design);
    
    return `Design document created successfully at: ${projectContext.projectPath}/.spec-workflow/specs/${projectContext.specName}/design.md`;
  }
  
  private async loadRequirements(projectPath: string, specName: string) {
    try {
      const requirementsPath = `${projectPath}/.spec-workflow/specs/${specName}/requirements.md`;
      const content = await this.readFile(requirementsPath);
      return this.parseRequirements(content);
    } catch {
      throw new Error('Requirements document not found. Please create requirements first.');
    }
  }
  
  private async analyzeCodebase(projectPath: string) {
    const analysis = {
      summary: {
        totalFiles: 0,
        languages: [],
        frameworks: [],
        patterns: []
      },
      components: [],
      services: [],
      utilities: [],
      existingPatterns: {
        architecture: [],
        designPatterns: [],
        codeOrganization: []
      },
      reuseOpportunities: []
    };

    // Scan project structure
    const srcPath = `${projectPath}/src`;
    if (await this.directoryExists(srcPath)) {
      const files = await this.scanDirectory(srcPath);
      analysis.summary.totalFiles = files.length;
      
      // Analyze file types and patterns
      for (const file of files) {
        this.analyzeFile(file, analysis);
      }
    }

    return analysis;
  }
  
  private async generateDesign(requirements: any, codebaseAnalysis: any) {
    const template = `# Design Document

## Overview

${this.generateOverview(requirements)}

## Steering Document Alignment

### Technical Standards (tech.md)
${this.generateTechnicalStandardsAlignment(codebaseAnalysis)}

### Project Structure (structure.md)
${this.generateProjectStructureAlignment(codebaseAnalysis)}

## Code Reuse Analysis

${this.generateCodeReuseAnalysis(codebaseAnalysis)}

### Existing Components to Leverage
${this.generateExistingComponents(codebaseAnalysis)}

### Integration Points
${this.generateIntegrationPoints(codebaseAnalysis)}

## Architecture

${this.generateArchitectureSection(requirements, codebaseAnalysis)}

### Modular Design Principles
- **Single File Responsibility**: Each file handles one specific concern or domain
- **Component Isolation**: Create small, focused components rather than large monolithic files
- **Service Layer Separation**: Separate data access, business logic, and presentation layers
- **Utility Modularity**: Break utilities into focused, single-purpose modules

### Architecture Diagram
\`\`\`mermaid
graph TD
    A[User Interface] --> B[API Layer]
    B --> C[Business Logic]
    C --> D[Data Access]
    D --> E[Database]
\`\`\`

## Components and Interfaces

${this.generateComponentsSection(requirements, codebaseAnalysis)}

## Data Models

${this.generateDataModelsSection(requirements)}

## Error Handling

${this.generateErrorHandlingSection(requirements)}

## Testing Strategy

${this.generateTestingStrategy(requirements)}
`;
    
    return template;
  }
  
  private async createDesignDocument(projectPath: string, specName: string, content: string) {
    const specDir = `${projectPath}/.spec-workflow/specs/${specName}`;
    await this.createDirectory(specDir);
    
    const designPath = `${specDir}/design.md`;
    await this.writeFile(designPath, content);
  }
  
  private parseRequirements(content: string): any {
    // Parse requirements markdown content
    const requirements = {
      featureDescription: '',
      userStories: [],
      businessObjectives: [],
      constraints: []
    };
    
    // Extract feature description
    const introMatch = content.match(/## Introduction\n\n([\s\S]*?)\n##/);
    if (introMatch) {
      requirements.featureDescription = introMatch[1].trim();
    }
    
    // Extract user stories
    const storyMatches = content.match(/### Requirement \d+\n\n\*\*User Story:\*\* ([^\n]+)/g);
    if (storyMatches) {
      requirements.userStories = storyMatches.map(match => 
        match.replace(/### Requirement \d+\n\n\*\*User Story:\*\* /, '')
      );
    }
    
    return requirements;
  }
  
  private async scanDirectory(dirPath: string): Promise<string[]> {
    try {
      const files: string[] = [];
      const entries = await this.listFiles(dirPath);
      
      for (const entry of entries) {
        const fullPath = `${dirPath}/${entry}`;
        if (await this.isDirectory(fullPath)) {
          files.push(...await this.scanDirectory(fullPath));
        } else {
          files.push(fullPath);
        }
      }
      
      return files;
    } catch {
      return [];
    }
  }
  
  private analyzeFile(filePath: string, analysis: any) {
    // Analyze individual file and update analysis
    if (filePath.endsWith('.ts') || filePath.endsWith('.js')) {
      analysis.summary.languages.push('TypeScript/JavaScript');
    } else if (filePath.endsWith('.py')) {
      analysis.summary.languages.push('Python');
    } else if (filePath.endsWith('.java')) {
      analysis.summary.languages.push('Java');
    }
    
    // Detect frameworks and patterns
    if (filePath.includes('react') || filePath.includes('jsx')) {
      analysis.summary.frameworks.push('React');
    } else if (filePath.includes('vue')) {
      analysis.summary.frameworks.push('Vue');
    } else if (filePath.includes('express')) {
      analysis.summary.frameworks.push('Express');
    }
  }
  
  private generateOverview(requirements: any): string {
    return requirements.featureDescription || 'Technical design for the specified feature.';
  }
  
  private generateTechnicalStandardsAlignment(codebaseAnalysis: any): string {
    if (codebaseAnalysis.summary.frameworks.length > 0) {
      return `Design aligns with existing technology stack: ${codebaseAnalysis.summary.frameworks.join(', ')}`;
    }
    return 'Design follows established patterns and best practices.';
  }
  
  private generateProjectStructureAlignment(codebaseAnalysis: any): string {
    return 'Implementation follows project structure conventions and maintains consistency with existing codebase.';
  }
  
  private generateCodeReuseAnalysis(codebaseAnalysis: any): string {
    if (codebaseAnalysis.reuseOpportunities.length > 0) {
      return codebaseAnalysis.reuseOpportunities.map((opp: any) => `- **${opp.component}**: ${opp.description}`).join('\n');
    }
    return 'No existing components identified for reuse. New implementation required.';
  }
  
  private generateExistingComponents(codebaseAnalysis: any): string {
    // Generate list of existing components that can be leveraged
    return '';
  }
  
  private generateIntegrationPoints(codebaseAnalysis: any): string {
    // Generate integration points with existing systems
    return '';
  }
  
  private generateArchitectureSection(requirements: any, codebaseAnalysis: any): string {
    return `[Describe the overall architecture and design patterns used]

Consider:
- Overall system architecture
- Design patterns to be used
- Technology choices and rationale
- Performance considerations
- Security requirements
- Scalability requirements`;
  }
  
  private generateComponentsSection(requirements: any, codebaseAnalysis: any): string {
    return `### Component 1
- **Purpose:** [What this component does]
- **Interfaces:** [Public methods/APIs]
- **Dependencies:** [What it depends on]
- **Reuses:** [Existing components/utilities it builds upon]

### Component 2
- **Purpose:** [What this component does]
- **Interfaces:** [Public methods/APIs]
- **Dependencies:** [What it depends on]
- **Reuses:** [Existing components/utilities it builds upon]`;
  }
  
  private generateDataModelsSection(requirements: any): string {
    return `### Model 1
\`\`\`
[Define the structure of Model1 in your language]
- id: [unique identifier type]
- name: [string/text type]
- [Additional properties as needed]
\`\`\`

### Model 2
\`\`\`
[Define the structure of Model2 in your language]
- id: [unique identifier type]
- [Additional properties as needed]
\`\`\``;
  }
  
  private generateErrorHandlingSection(requirements: any): string {
    return `### Error Scenarios
1. **Scenario 1:** [Description]
   - **Handling:** [How to handle]
   - **User Impact:** [What user sees]

2. **Scenario 2:** [Description]
   - **Handling:** [How to handle]
   - **User Impact:** [What user sees]`;
  }
  
  private generateTestingStrategy(requirements: any): string {
    return `### Unit Testing
- [Unit testing approach]
- [Key components to test]

### Integration Testing
- [Integration testing approach]
- [Key flows to test]

### End-to-End Testing
- [E2E testing approach]
- [User scenarios to test]`;
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
  
  private async directoryExists(path: string): Promise<boolean> {
    // Implementation would use Bash tool
    return false;
  }
  
  private async listFiles(path: string): Promise<string[]> {
    // Implementation would use Bash tool
    return [];
  }
  
  private async isDirectory(path: string): Promise<boolean> {
    // Implementation would use Bash tool
    return false;
  }
}
```

## Usage Instructions

### Adding to Claude Code

1. Save this agent configuration in your Claude Code agents directory
2. Configure the agent in your Claude Code settings
3. Use the agent with prompts like:

```
@design-architect Create a technical design for the user authentication system
```

```
@design-architect Design the payment processing feature architecture
```

### Agent Capabilities

- **Requirements Analysis**: Parse and analyze existing requirements
- **Codebase Analysis**: Scan existing code for patterns and reuse opportunities
- **Architecture Design**: Create system architecture and component designs
- **Documentation Generation**: Generate comprehensive design documents
- **Pattern Detection**: Identify and leverage existing design patterns

### Workflow Integration

This agent works as part of a three-agent workflow:

1. **Requirements Specialist** → Creates requirements.md
2. **Design Architect** → Creates design.md  
3. **Implementation Task** → Executes tasks.md

### Prerequisites

- Requirements document must exist in `.spec-workflow/specs/{spec-name}/requirements.md`
- Project directory must be accessible for codebase analysis
- Write permissions for creating design documents

## Example Usage

### Basic Design Creation

```
@design-architect Create a technical design for the user authentication system based on the existing requirements
```

### Architecture Design

```
@design-architect Design the system architecture for payment processing, considering existing codebase patterns and technical standards
```

### Component Design

```
@design-architect Design the component structure and data models for the new reporting feature
```

## Configuration Options

The agent can be configured with:

- **Template Path**: Custom design template location
- **Codebase Scan Depth**: How deep to scan for code patterns
- **Architecture Standards**: Preferred architecture patterns
- **Documentation Format**: Markdown or other formats
- **Integration Rules**: How to integrate with existing systems

## Error Handling

The agent handles common errors:

- **Missing Requirements**: Provides clear error message
- **Inaccessible Codebase**: Works with available information
- **Template Issues**: Falls back to default templates
- **Permission Issues**: Provides guidance on file permissions

## Best Practices

1. **Requirements First**: Always ensure requirements exist before design
2. **Codebase Awareness**: Leverage existing patterns and components
3. **Modular Design**: Design for maintainability and scalability
4. **Documentation**: Create comprehensive design documentation
5. **Standards Compliance**: Follow established technical standards

## File Structure Created

```
project-root/
└── .spec-workflow/
    └── specs/
        └── {feature-name}/
            ├── requirements.md
            └── design.md
```