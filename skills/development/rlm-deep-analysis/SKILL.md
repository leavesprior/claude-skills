# RLM Deep Analysis

Use Recursive Language Models for complex, multi-step reasoning tasks that require decomposition and recursive problem-solving.

## When to Use

Invoke this skill when:
- Analyzing large codebases or documents that exceed context limits
- Complex reasoning requiring step-by-step decomposition
- Questions that benefit from recursive exploration
- Deep quality analysis where thoroughness matters more than speed
- Tasks requiring programmatic examination of inputs

## Trigger Phrases

- "deep analysis"
- "recursive reasoning"
- "thorough investigation"
- "decompose this problem"
- "RLM analysis"

## Usage

### Quick Analysis (via CLI wrapper)
```bash
# Simple query
neoma-rlm "Analyze the architecture of this codebase"

# With file context
neoma-rlm --file /path/to/code.py "Explain the data flow"

# Increased recursion depth (default: 3)
neoma-rlm --depth 5 "Complex multi-step analysis"
```

### Python Direct Usage
```python
from rlm import RLM
import os

rlm = RLM(
    backend="anthropic",
    backend_kwargs={
        "model_name": "claude-sonnet-4-20250514",
        "api_key": os.getenv("ANTHROPIC_API_KEY"),
    },
    environment="local",
    max_depth=3,
    verbose=True,
)

result = rlm.completion("Your complex query here")
print(result.response)
```

## How RLM Works

1. **Decomposition**: Breaks complex tasks into manageable sub-problems
2. **Recursive Calls**: Spawns sub-LM calls to handle each piece
3. **REPL Environment**: Code execution in sandboxed Python
4. **Synthesis**: Combines results from recursive calls

## Configuration

RLM is installed at: `/media/granny/larger SSD/Neoma_project/rlm`

Environment variables needed:
- `ANTHROPIC_API_KEY` - For Claude backend

## Best Practices

1. **Be specific** about what depth of analysis you need
2. **Provide context** - file paths or code snippets help
3. **Set appropriate depth** - higher = more thorough but slower
4. For **codebase analysis**, combine with `tldr` MCP for structure first

## Integration with Other Tools

- **tldr-mcp**: Get codebase structure, then RLM for deep analysis
- **grepai**: Find relevant code semantically, then RLM to understand it
- **zeroshot**: Use RLM for planning, zeroshot for validated implementation
