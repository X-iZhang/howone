# HowOne Workflow Types

HowOne AI generates agentic workflows based on your app requirements.

## Common Workflow Patterns

### 1. Content Generation Workflow

```
User Input → Prompt Builder → LLM Node → Output Formatter → Display
```

Used for: Blog generators, email writers, social media content

### 2. Analysis Workflow

```
User Input → Data Parser → Analysis Agent → Report Generator → Display/Export
```

Used for: Contract analysis, data insights, document review

### 3. Multi-Step Processing Workflow

```
Input → Step 1 Agent → Step 2 Agent → Step 3 Agent → Final Output
```

Used for: Complex transformations, multi-stage content creation

### 4. Interactive Workflow

```
User Input → Processing → User Confirmation → Final Action
```

Used for: Approval workflows, iterative refinement

## Node Types

| Node | Function |
|------|----------|
| **Input Node** | Receives user input (text, file, image) |
| **LLM Node** | Calls language model for generation/analysis |
| **Tool Node** | Executes specific tools (search, calculator) |
| **Condition Node** | Branches based on conditions |
| **Output Node** | Formats and displays results |
| **API Node** | Calls external APIs |

## Editing Workflows

### Edit Single Node
1. Click on the node in the workflow canvas
2. Enter modification instructions in "Edit Node"
3. AI will update that specific node

### Edit Entire Workflow
1. Right-click blank area in workflow canvas
2. Click "Edit Workflow"
3. Describe the changes you want

### Version Management
- View all versions in "Version History"
- Rollback to any previous version
- Self-evolution creates new versions automatically

## Self-Evolution

Workflows can improve automatically through:

1. **Auto Evolution**: After first generation, AI evaluates and optimizes
2. **Manual Evolution**: Select 2-5 input/output pairs, rate them, trigger optimization

### Evolution Modes

| Mode | Rounds | Use Case |
|------|--------|----------|
| Quick Try | 1 | Fast test |
| Standard | 3 | Regular optimization |
| Deep | 5 | Complex workflows |

## Best Practices

1. **Start simple**: Generate basic version first, then iterate
2. **Use version history**: Keep track of what works
3. **Test with real data**: Use actual user inputs for evolution
4. **Check logs**: Use workflow logs to debug issues