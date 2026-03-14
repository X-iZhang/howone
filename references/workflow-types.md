# HowOne Workflow Types

HowOne AI generates agentic workflows based on your app requirements. Understanding these patterns helps when reviewing or editing generated apps.

## Common Workflow Patterns

### 1. Content Generation

```
User Input -> Prompt Builder -> LLM Node -> Output Formatter -> Display
```

Used for: Blog generators, email writers, social media content

### 2. Analysis

```
User Input -> Data Parser -> Analysis Agent -> Report Generator -> Display/Export
```

Used for: Contract analysis, data insights, document review

### 3. Multi-Step Processing

```
Input -> Step 1 Agent -> Step 2 Agent -> Step 3 Agent -> Final Output
```

Used for: Complex transformations, multi-stage content creation

### 4. Interactive

```
User Input -> Processing -> User Confirmation -> Final Action
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

## Editing Workflows via Playwright CLI

The workflow editor is in the **right panel** of the project page. To edit workflows using playwright-cli:

### Edit a Single Node
```bash
# Snapshot the project page
playwright-cli snapshot

# Find and click the target node in the workflow panel
# Nodes appear as interactive elements in the right panel
playwright-cli click <ref-of-node>

# Snapshot to see the edit panel that appears
playwright-cli snapshot

# Find the edit/instruction input and enter changes
playwright-cli fill <ref-of-edit-input> "Add error handling to this node"
playwright-cli click <ref-of-save-button>
```

### Edit Entire Workflow via Chat
```bash
# Use the chat panel (left side) to request workflow changes
playwright-cli snapshot
playwright-cli fill <ref-of-chat-input> "Add a data validation step before the LLM node"
playwright-cli click <ref-of-send-button>

# Wait for the AI to process and update the workflow
playwright-cli snapshot
```

## Version Management

- Each edit creates a new version automatically
- View version history in the project settings
- Self-evolution (auto-optimization) also creates new versions

## Self-Evolution

Workflows can improve automatically:

1. **Auto Evolution**: After first generation, AI evaluates and optimizes
2. **Manual Evolution**: Provide 2-5 input/output examples, rate them, trigger optimization

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
