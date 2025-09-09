# Claude Code Sub-Agent: Implementation Task Agent

A specialized agent for systematic task execution and code implementation in spec-driven development.

## Agent Configuration

```json
{
  "name": "implementation-task",
  "description": "Specializes in systematic task execution, code implementation, and progress tracking for spec-driven development workflows",
  "type": "fullstack-developer",
  "capabilities": [
    {
      "name": "Task Management",
      "description": "Track and update task implementation progress systematically"
    },
    {
      "name": "Code Implementation",
      "description": "Execute atomic implementation tasks with precision"
    },
    {
      "name": "Progress Tracking",
      "description": "Monitor implementation progress and handle dependencies"
    }
  ],
  "tools": ["read", "write", "glob", "bash", "task"],
  "prompt_template": "You are an Implementation Task Agent for spec-driven development. Your role is to:\\n1. Execute implementation tasks systematically and atomically\\n2. Track progress and update task status\\n3. Handle task dependencies and prerequisites\\n4. Follow existing code patterns and conventions\\n5. Maintain code quality and testing standards\\n\\nAlways follow the spec workflow: Requirements → Design → Tasks → Implementation\\n\\nCurrent project context: {project_context}\\n\\nUser request: {user_request}"
}
```

## Agent Implementation

```typescript
class ImplementationTaskAgent {
  async execute(userRequest: string, projectContext: any): Promise<string> {
    // 1. Load existing tasks document
    const tasks = await this.loadTasksDocument(projectContext.projectPath, projectContext.specName);
    
    // 2. Load design document for context
    const designContext = await this.loadDesignContext(projectContext.projectPath, projectContext.specName);
    
    // 3. Analyze current progress
    const progress = await this.analyzeProgress(tasks);
    
    // 4. Execute next task or specific task
    const result = await this.executeTasks(tasks, designContext, progress, userRequest);
    
    // 5. Update tasks document with progress
    await this.updateTasksDocument(projectContext.projectPath, projectContext.specName, tasks);
    
    return `Task execution completed. Next task: ${result.nextTask || 'All tasks completed'}`;
  }
  
  private async loadTasksDocument(projectPath: string, specName: string) {
    try {
      const tasksPath = `${projectPath}/.spec-workflow/specs/${specName}/tasks.md`;
      const content = await this.readFile(tasksPath);
      return this.parseTasks(content);
    } catch {
      throw new Error('Tasks document not found. Please create design document first to generate tasks.');
    }
  }
  
  private async loadDesignContext(projectPath: string, specName: string) {
    try {
      const designPath = `${projectPath}/.spec-workflow/specs/${specName}/design.md`;
      const content = await this.readFile(designPath);
      return this.parseDesignContext(content);
    } catch {
      throw new Error('Design document not found. Please complete design phase first.');
    }
  }
  
  private async analyzeProgress(tasks: any) {
    const progress = {
      completed: [],
      inProgress: [],
      pending: [],
      totalTasks: tasks.length,
      completionPercentage: 0
    };
    
    for (const task of tasks) {
      if (task.status === 'completed' || task.checked) {
        progress.completed.push(task);
      } else if (task.status === 'in-progress') {
        progress.inProgress.push(task);
      } else {
        progress.pending.push(task);
      }
    }
    
    progress.completionPercentage = Math.round((progress.completed.length / progress.totalTasks) * 100);
    return progress;
  }
  
  private async executeTasks(tasks: any, designContext: any, progress: any, userRequest: string) {
    // Determine which task(s) to execute
    const tasksToExecute = this.selectTasksToExecute(tasks, progress, userRequest);
    
    const results = [];
    for (const task of tasksToExecute) {
      const result = await this.executeSingleTask(task, designContext);
      results.push(result);
      
      // Update task status
      task.status = 'completed';
      task.checked = true;
      task.completedAt = new Date().toISOString();
    }
    
    // Find next pending task
    const nextTask = tasks.find(t => t.status === 'pending' || !t.checked);
    
    return {
      executed: results,
      nextTask: nextTask ? nextTask.description : null
    };
  }
  
  private selectTasksToExecute(tasks: any, progress: any, userRequest: string) {
    // Check if user specified a particular task
    const taskMatch = userRequest.match(/(?:task|impl)\s*[-\s]*(\d+)/i);
    if (taskMatch) {
      const taskId = parseInt(taskMatch[1]);
      const task = tasks.find(t => t.id === taskId || t.order === taskId);
      if (task && (task.status === 'pending' || !task.checked)) {
        return [task];
      }
    }
    
    // Execute next pending task
    const nextTask = tasks.find(t => t.status === 'pending' || !t.checked);
    if (nextTask) {
      return [nextTask];
    }
    
    // Execute all pending tasks if requested
    if (userRequest.toLowerCase().includes('all') || userRequest.toLowerCase().includes('remaining')) {
      return tasks.filter(t => t.status === 'pending' || !t.checked);
    }
    
    return [];
  }
  
  private async executeSingleTask(task: any, designContext: any) {
    const result = {
      taskId: task.id,
      description: task.description,
      filesCreated: [],
      filesModified: [],
      testsRun: [],
      errors: [],
      success: false
    };
    
    try {
      // Parse task metadata
      const taskDetails = this.parseTaskDetails(task);
      
      // Execute based on task type
      switch (taskDetails.type) {
        case 'create-file':
          await this.executeFileCreation(taskDetails, result);
          break;
        case 'modify-file':
          await this.executeFileModification(taskDetails, result);
          break;
        case 'create-component':
          await this.executeComponentCreation(taskDetails, designContext, result);
          break;
        case 'implement-feature':
          await this.executeFeatureImplementation(taskDetails, designContext, result);
          break;
        case 'write-tests':
          await this.executeTestWriting(taskDetails, result);
          break;
        default:
          throw new Error(`Unknown task type: ${taskDetails.type}`);
      }
      
      // Run tests if applicable
      if (taskDetails.runTests) {
        await this.runTests(result);
      }
      
      result.success = true;
    } catch (error) {
      result.errors.push(error.message);
      result.success = false;
    }
    
    return result;
  }
  
  private parseTaskDetails(task: any) {
    // Extract file path, component type, and other metadata from task description
    const details = {
      type: 'create-file', // default
      filePath: '',
      componentType: '',
      leverage: [],
      requirements: [],
      runTests: false
    };
    
    // Extract file path
    const pathMatch = task.description.match(/(?:in|at)\s+([^\s]+(?:\.(?:ts|js|py|java|jsx|tsx)))/i);
    if (pathMatch) {
      details.filePath = pathMatch[1];
    }
    
    // Extract component type
    if (task.description.toLowerCase().includes('component')) {
      details.type = 'create-component';
      details.componentType = 'component';
    } else if (task.description.toLowerCase().includes('interface') || task.description.toLowerCase().includes('type')) {
      details.componentType = 'interface';
    } else if (task.description.toLowerCase().includes('service')) {
      details.componentType = 'service';
    } else if (task.description.toLowerCase().includes('test')) {
      details.type = 'write-tests';
      details.runTests = true;
    } else if (task.description.toLowerCase().includes('implement') || task.description.toLowerCase().includes('feature')) {
      details.type = 'implement-feature';
    }
    
    // Extract leverage references
    const leverageMatch = task.description.match(/leverage:\s*([^\n]+)/i);
    if (leverageMatch) {
      details.leverage = leverageMatch[1].split(',').map(s => s.trim());
    }
    
    return details;
  }
  
  private async executeFileCreation(taskDetails: any, result: any) {
    if (!taskDetails.filePath) {
      throw new Error('File path not specified in task');
    }
    
    // Generate file content based on path and context
    const content = await this.generateFileContent(taskDetails);
    
    // Create directory if needed
    const dirPath = taskDetails.filePath.substring(0, taskDetails.filePath.lastIndexOf('/'));
    if (dirPath) {
      await this.createDirectory(dirPath);
    }
    
    // Write file
    await this.writeFile(taskDetails.filePath, content);
    result.filesCreated.push(taskDetails.filePath);
  }
  
  private async executeFileModification(taskDetails: any, result: any) {
    if (!taskDetails.filePath) {
      throw new Error('File path not specified in task');
    }
    
    // Read existing file
    const existingContent = await this.readFile(taskDetails.filePath);
    
    // Generate modifications
    const modifiedContent = await this.generateFileModifications(taskDetails, existingContent);
    
    // Write modified file
    await this.writeFile(taskDetails.filePath, modifiedContent);
    result.filesModified.push(taskDetails.filePath);
  }
  
  private async executeComponentCreation(taskDetails: any, designContext: any, result: any) {
    // Generate component based on design context
    const componentCode = await this.generateComponentCode(taskDetails, designContext);
    
    // Create component file
    await this.writeFile(taskDetails.filePath, componentCode);
    result.filesCreated.push(taskDetails.filePath);
    
    // Create associated test file
    const testPath = taskDetails.filePath.replace(/\.(ts|js|jsx|tsx)$/, '.test.$1');
    const testCode = await this.generateComponentTest(taskDetails, componentCode);
    await this.writeFile(testPath, testCode);
    result.filesCreated.push(testPath);
  }
  
  private async executeFeatureImplementation(taskDetails: any, designContext: any, result: any) {
    // Implement feature based on design specifications
    const implementation = await this.generateFeatureImplementation(taskDetails, designContext);
    
    // Create implementation files
    for (const file of implementation.files) {
      await this.writeFile(file.path, file.content);
      result.filesCreated.push(file.path);
    }
    
    // Update existing files
    for (const modification of implementation.modifications) {
      const existingContent = await this.readFile(modification.path);
      const modifiedContent = this.applyModifications(existingContent, modification.changes);
      await this.writeFile(modification.path, modifiedContent);
      result.filesModified.push(modification.path);
    }
  }
  
  private async executeTestWriting(taskDetails: any, result: any) {
    // Generate test code
    const testCode = await this.generateTestCode(taskDetails);
    
    // Create test file
    await this.writeFile(taskDetails.filePath, testCode);
    result.filesCreated.push(taskDetails.filePath);
    
    // Run tests
    await this.runTests(result);
  }
  
  private async runTests(result: any) {
    try {
      // Detect test framework and run appropriate command
      const testCommand = await this.detectTestCommand();
      if (testCommand) {
        const testOutput = await this.executeCommand(testCommand);
        result.testsRun.push({
          command: testCommand,
          output: testOutput,
          success: !testOutput.includes('FAIL') && !testOutput.includes('ERROR')
        });
      }
    } catch (error) {
      result.errors.push(`Test execution failed: ${error.message}`);
    }
  }
  
  private async generateFileContent(taskDetails: any): Promise<string> {
    // Generate content based on file type and context
    const ext = taskDetails.filePath.split('.').pop();
    
    switch (ext) {
      case 'ts':
        return this.generateTypeScriptFile(taskDetails);
      case 'js':
        return this.generateJavaScriptFile(taskDetails);
      case 'py':
        return this.generatePythonFile(taskDetails);
      case 'java':
        return this.generateJavaFile(taskDetails);
      case 'jsx':
      case 'tsx':
        return this.generateReactComponent(taskDetails);
      default:
        return '// Generated file content';
    }
  }
  
  private generateTypeScriptFile(taskDetails: any): string {
    return `// Generated TypeScript file
// Type: ${taskDetails.componentType || 'utility'}

export interface GeneratedInterface {
  id: string;
  name: string;
  createdAt: Date;
  updatedAt: Date;
}

export class GeneratedClass {
  private data: GeneratedInterface;
  
  constructor(data: Partial<GeneratedInterface>) {
    this.data = {
      id: data.id || '',
      name: data.name || '',
      createdAt: data.createdAt || new Date(),
      updatedAt: new Date()
    };
  }
  
  // Implementation methods
  public getId(): string {
    return this.data.id;
  }
  
  public getName(): string {
    return this.data.name;
  }
}
`;
  }
  
  private generateJavaScriptFile(taskDetails: any): string {
    return `// Generated JavaScript file
// Type: ${taskDetails.componentType || 'utility'}

class GeneratedClass {
  constructor(data = {}) {
    this.data = {
      id: data.id || '',
      name: data.name || '',
      createdAt: data.createdAt || new Date(),
      updatedAt: new Date()
    };
  }
  
  getId() {
    return this.data.id;
  }
  
  getName() {
    return this.data.name;
  }
}

module.exports = GeneratedClass;
`;
  }
  
  private generatePythonFile(taskDetails: any): string {
    return `# Generated Python file
# Type: ${taskDetails.componentType || 'utility'}

from datetime import datetime
from typing import Optional, Dict, Any

class GeneratedClass:
    def __init__(self, data: Optional[Dict[str, Any]] = None):
        self.data = {
            'id': data.get('id', '') if data else '',
            'name': data.get('name', '') if data else '',
            'created_at': data.get('created_at', datetime.now()) if data else datetime.now(),
            'updated_at': datetime.now()
        }
    
    def get_id(self) -> str:
        return self.data['id']
    
    def get_name(self) -> str:
        return self.data['name']
`;
  }
  
  private generateJavaFile(taskDetails: any): string {
    return `// Generated Java file
// Type: ${taskDetails.componentType || 'utility'}

public class GeneratedClass {
    private String id;
    private String name;
    private Date createdAt;
    private Date updatedAt;
    
    public GeneratedClass(String id, String name) {
        this.id = id;
        this.name = name;
        this.createdAt = new Date();
        this.updatedAt = new Date();
    }
    
    public String getId() {
        return this.id;
    }
    
    public String getName() {
        return this.name;
    }
    
    public Date getCreatedAt() {
        return this.createdAt;
    }
    
    public Date getUpdatedAt() {
        return this.updatedAt;
    }
}
`;
  }
  
  private generateReactComponent(taskDetails: any): string {
    return `// Generated React Component
import React, { useState, useEffect } from 'react';

interface GeneratedComponentProps {
  data?: any;
  onAction?: (action: string) => void;
}

const GeneratedComponent: React.FC<GeneratedComponentProps> = ({ data, onAction }) => {
  const [state, setState] = useState<any>(null);
  
  useEffect(() => {
    if (data) {
      setState(data);
    }
  }, [data]);
  
  const handleAction = (action: string) => {
    if (onAction) {
      onAction(action);
    }
  };
  
  return (
    <div className="generated-component">
      <h3>Generated Component</h3>
      {state && (
        <div>
          <p>ID: {state.id}</p>
          <p>Name: {state.name}</p>
        </div>
      )}
      <button onClick={() => handleAction('click')}>
        Action
      </button>
    </div>
  );
};

export default GeneratedComponent;
`;
  }
  
  private async generateComponentCode(taskDetails: any, designContext: any): Promise<string> {
    // Generate component code based on design specifications
    return this.generateTypeScriptFile(taskDetails);
  }
  
  private async generateComponentTest(taskDetails: any, componentCode: string): Promise<string> {
    return `// Generated component tests
import { render, screen, fireEvent } from '@testing-library/react';
import GeneratedComponent from './${taskDetails.filePath.split('/').pop().replace(/\.(ts|tsx)$/, '')}';

describe('GeneratedComponent', () => {
  it('renders without crashing', () => {
    render(<GeneratedComponent />);
    expect(screen.getByText('Generated Component')).toBeInTheDocument();
  });
  
  it('displays data when provided', () => {
    const testData = { id: '1', name: 'Test' };
    render(<GeneratedComponent data={testData} />);
    expect(screen.getByText('ID: 1')).toBeInTheDocument();
    expect(screen.getByText('Name: Test')).toBeInTheDocument();
  });
  
  it('calls onAction when button is clicked', () => {
    const mockOnAction = jest.fn();
    render(<GeneratedComponent onAction={mockOnAction} />);
    fireEvent.click(screen.getByText('Action'));
    expect(mockOnAction).toHaveBeenCalledWith('click');
  });
});
`;
  }
  
  private async generateFeatureImplementation(taskDetails: any, designContext: any) {
    // Generate complete feature implementation
    return {
      files: [
        {
          path: `${taskDetails.filePath}/types.ts`,
          content: this.generateTypeScriptFile(taskDetails)
        }
      ],
      modifications: []
    };
  }
  
  private async generateTestCode(taskDetails: any): Promise<string> {
    return `// Generated test file
import { describe, it, expect } from 'vitest';

describe('Generated Test Suite', () => {
  it('should pass basic test', () => {
    expect(true).toBe(true);
  });
  
  it('should handle async operations', async () => {
    const result = await Promise.resolve('test');
    expect(result).toBe('test');
  });
});
`;
  }
  
  private async detectTestCommand(): Promise<string> {
    // Detect available test framework
    const frameworks = [
      { command: 'npm test', files: ['package.json'] },
      { command: 'yarn test', files: ['yarn.lock'] },
      { command: 'pytest', files: ['requirements.txt'] },
      { command: 'mvn test', files: ['pom.xml'] }
    ];
    
    for (const framework of frameworks) {
      if (await this.fileExists(framework.files[0])) {
        return framework.command;
      }
    }
    
    return '';
  }
  
  private applyModifications(existingContent: string, changes: any[]): string {
    // Apply modifications to existing content
    let modifiedContent = existingContent;
    
    for (const change of changes) {
      switch (change.type) {
        case 'insert':
          modifiedContent = modifiedContent.replace(change.insertAt, change.content + change.insertAt);
          break;
        case 'replace':
          modifiedContent = modifiedContent.replace(change.replace, change.content);
          break;
        case 'append':
          modifiedContent += change.content;
          break;
      }
    }
    
    return modifiedContent;
  }
  
  private async generateFileModifications(taskDetails: any, existingContent: string): Promise<string> {
    // Generate modifications based on task requirements
    return existingContent + '\n// Added by implementation task\n';
  }
  
  private parseTasks(content: string): any[] {
    // Parse tasks from markdown content
    const tasks: any[] = [];
    const lines = content.split('\n');
    let currentTask = null;
    let taskId = 1;
    
    for (const line of lines) {
      const checkboxMatch = line.match(/- \[([^\]]+)\]\s*(\d+\.\s*)?(.+)/);
      if (checkboxMatch) {
        const [, status, id, description] = checkboxMatch;
        
        if (currentTask) {
          tasks.push(currentTask);
        }
        
        currentTask = {
          id: taskId++,
          description: description.trim(),
          status: status === 'x' ? 'completed' : 'pending',
          checked: status === 'x',
          metadata: {}
        };
      } else if (currentTask && line.trim()) {
        // Parse additional metadata
        if (line.includes('leverage:')) {
          currentTask.metadata.leverage = line.replace('leverage:', '').trim();
        } else if (line.includes('requirements:')) {
          currentTask.metadata.requirements = line.replace('requirements:', '').trim();
        }
      }
    }
    
    if (currentTask) {
      tasks.push(currentTask);
    }
    
    return tasks;
  }
  
  private parseDesignContext(content: string): any {
    // Parse design document for context
    return {
      architecture: '',
      components: [],
      dataModels: [],
      patterns: []
    };
  }
  
  private async updateTasksDocument(projectPath: string, specName: string, tasks: any) {
    // Update the tasks.md file with current progress
    const tasksPath = `${projectPath}/.spec-workflow/specs/${specName}/tasks.md`;
    const content = await this.generateTasksContent(tasks);
    await this.writeFile(tasksPath, content);
  }
  
  private async generateTasksContent(tasks: any[]): Promise<string> {
    // Generate markdown content for tasks
    let content = '# Implementation Tasks\n\n';
    
    for (const task of tasks) {
      const checkbox = task.checked ? 'x' : ' ';
      content += `- [${checkbox}] ${task.id}. ${task.description}\n`;
      
      if (task.metadata.leverage) {
        content += `  leverage: ${task.metadata.leverage}\n`;
      }
      
      if (task.metadata.requirements) {
        content += `  requirements: ${task.metadata.requirements}\n`;
      }
      
      content += '\n';
    }
    
    return content;
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
  
  private async fileExists(path: string): Promise<boolean> {
    // Implementation would use Bash tool
    return false;
  }
  
  private async executeCommand(command: string): Promise<string> {
    // Implementation would use Bash tool
    return "";
  }
}
```

## Usage Instructions

### Adding to Claude Code

1. Save this agent configuration in your Claude Code agents directory
2. Configure the agent in your Claude Code settings
3. Use the agent with prompts like:

```
@implementation-task Execute the next task in the user-authentication workflow
```

```
@implementation-task Execute task impl-3 for the payment-processing feature
```

```
@implementation-task Execute all remaining tasks for the user-profiles workflow
```

### Agent Capabilities

- **Task Management**: Parse and track implementation tasks from tasks.md
- **Code Generation**: Generate code based on task specifications and design context
- **File Operations**: Create and modify files systematically
- **Testing Integration**: Write and run tests automatically
- **Progress Tracking**: Monitor and report implementation progress

### Workflow Integration

This agent works as part of a three-agent workflow:

1. **Requirements Specialist** → Creates requirements.md
2. **Design Architect** → Creates design.md  
3. **Implementation Task Agent** → Executes tasks.md

### Prerequisites

- Tasks document must exist in `.spec-workflow/specs/{spec-name}/tasks.md`
- Design document must exist in `.spec-workflow/specs/{spec-name}/design.md`
- Project directory must be accessible for file operations
- Write permissions for creating and modifying files

## Example Usage

### Basic Task Execution

```
@implementation-task Execute the next implementation task for user authentication
```

### Specific Task Execution

```
@implementation-task Execute task impl-2 for creating the user service interface
```

### Batch Task Execution

```
@implementation-task Execute all remaining tasks for the payment processing feature
```

### Feature Implementation

```
@implementation-task Execute the complete feature implementation for user profiles
```

## Configuration Options

The agent can be configured with:

- **Template Path**: Custom code templates location
- **Test Framework**: Preferred testing framework and commands
- **Code Style**: Code formatting and style preferences
- **Language Support**: Supported programming languages
- **Test Patterns**: Automatic test generation patterns

## Error Handling

The agent handles common errors:

- **Missing Tasks**: Provides clear error message about required tasks.md
- **Missing Design**: Requires design document to be completed first
- **File Permissions**: Provides guidance on file access issues
- **Test Failures**: Reports test failures and suggests fixes
- **Task Dependencies**: Handles task dependencies correctly

## Best Practices

1. **Sequential Execution**: Execute tasks in order to handle dependencies
2. **Design Reference**: Always reference design document for implementation details
3. **Test Coverage**: Write tests for all implemented functionality
4. **Code Quality**: Follow existing code patterns and conventions
5. **Progress Tracking**: Keep tasks.md updated with current progress

## File Structure Created

```
project-root/
└── .spec-workflow/
    └── specs/
        └── {feature-name}/
            ├── requirements.md
            ├── design.md
            └── tasks.md  # Updated with progress
src/
├── components/
├── services/
├── types/
└── tests/
```

## Advanced Features

### Task Type Support

- **File Creation**: Generate new files with appropriate content
- **File Modification**: Update existing files with new functionality
- **Component Creation**: Create complete components with tests
- **Feature Implementation**: Implement complete features with multiple files
- **Test Writing**: Generate comprehensive test suites

### Language Support

The agent supports multiple programming languages:
- TypeScript/JavaScript
- Python
- Java
- React/JSX/TSX

### Progress Monitoring

Real-time progress tracking with:
- Task completion status
- File creation/modification logs
- Test execution results
- Overall implementation percentage

## Integration with Development Tools

### Test Framework Integration

Automatically detects and uses:
- Jest/Vitest for JavaScript/TypeScript
- Pytest for Python
- JUnit for Java
- React Testing Library for React components

### Code Quality

Maintains code quality through:
- Consistent code formatting
- Proper error handling
- Comprehensive test coverage
- Documentation generation
- Type safety (TypeScript)

## Troubleshooting

### Common Issues

1. **Task Parsing Fails**
   - Verify tasks.md follows the correct format
   - Check for proper checkbox syntax

2. **File Creation Fails**
   - Ensure write permissions
   - Check directory structure
   - Verify file paths are valid

3. **Test Execution Fails**
   - Verify test framework is installed
   - Check test file syntax
   - Review test dependencies

### Debug Mode

Enable debug logging:

```bash
export DEBUG=implementation-task:*
export LOG_LEVEL=debug
```

## Support

For additional help:
- Check the main README.md
- Review CLAUDE_CODE_AGENTS.md
- Create issues in the project repository