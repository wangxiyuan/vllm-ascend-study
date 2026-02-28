---
name: code-summary
description: Generate comprehensive HTML code walkthrough documentation for vLLM and vllm-ascend codebase. Use this skill when the user wants to trace code execution from a specific method entry point, understand call chains, or create detailed line-by-line code analysis documentation. Triggers on requests like "trace this method", "walk through code from X", "generate code walkthrough for", "analyze code flow from entry point".
---

# Code Summary Skill

A skill for generating comprehensive, line-by-line code walkthrough HTML documentation for the vLLM and vllm-ascend codebases.

## Overview

This skill takes a method entry point specified by the user and performs deep code analysis, tracing the execution flow through vLLM and vllm-ascend codebases until reaching the lowest level implementation. Other dependency libraries are treated as black boxes.

## Input Requirements

The user must provide:
1. **Entry Point Method**: The fully qualified method name or a clear description of the starting point (e.g., `DPEngineCoreProc.run_busy_loop`, `EngineCore.step`, or "the main inference loop")

Optional:
2. **Output Directory**: Where to generate HTML files (defaults to current working directory)
3. **Custom Title**: A title for the documentation series

## Workflow

### Phase 1: Entry Point Discovery

1. Locate the entry point method in the codebase:
   - Search in vllm-ascend first (current working directory)
   - Then search in vllm (typically at same level as vllm-ascend or ask user for location)

2. Confirm the exact method signature and file location with the user if ambiguous.

### Phase 2: Call Chain Analysis

1. Read and analyze the entry point method completely
2. Identify all method calls within the analyzed code
3. For each call, determine:
   - Is it a vLLM core method? → Continue tracing
   - Is it a vllm-ascend method? → Continue tracing
   - Is it an external library? → Mark as black box, document the interface
   - Is it a Python builtin? → Skip

4. Build a call tree, noting:
   - File paths and line numbers
   - Class hierarchy relationships
   - Async/sync patterns
   - Data flow between methods

### Phase 3: Code Documentation Generation

For each discovered method in the call chain:

1. **Extract the source code** with proper context
2. **Analyze the logic**:
   - Purpose and responsibility
   - Key data structures used
   - Important algorithmic decisions
   - Ascend-specific adaptations (if applicable)

3. **Document the interfaces**:
   - Input parameters and their purposes
   - Return values and their structures
   - Side effects and state changes

4. **Create diagrams**:
   - ASCII diagrams for architecture visualization
   - Mermaid sequence diagrams for call flows
   - Data flow diagrams where appropriate

### Phase 4: HTML Generation

Generate HTML files following this structure:

```
output_dir/
├── index.html          # Overview and navigation
├── 01-<chapter>.html   # First major component
├── 02-<chapter>.html   # Second major component
└── ...                 # Additional chapters
```

The output_dir name should be the like `<class_name>_<chapter_number>` provided by the user.

### Phase 5: Update Index page

Add a new item to the navigation list in the `vllm-ascend-study/index.html` file.

## HTML Template Structure

Each HTML page must follow this exact structure for consistency:

### styles.css Content

The css content can found at `vllm-ascend-study/styles.css`

### HTML Page Template

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>[Chapter Number] - [Chapter Title] - [Main Title]</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/styles/github-dark.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/highlight.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/languages/python.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
    <link rel="stylesheet" href="../styles.css">
</head>
<body>
    <div class="container">
        <nav class="sidebar">
            <a href="../index.html" class="back-to-home">← 返回主页</a>
            <div class="sidebar-header">
                <h1>[Main Title]</h1>
                <p>[Subtitle]</p>
            </div>
            <div class="nav-section">
                <div class="nav-section-title">Documentation</div>
                <!-- Navigation items -->
            </div>
        </nav>

        <main class="main-content">
            <h1>[Chapter Title]</h1>

            <!-- Content sections -->

            <div class="chapter-nav">
                <!-- Previous/Next navigation -->
            </div>
        </main>
    </div>

    <script>
        hljs.highlightAll();
        mermaid.initialize({
            startOnLoad: true,
            theme: 'dark',
            themeVariables: {
                darkMode: true,
                background: '#161b22',
                primaryColor: '#58a6ff',
                primaryTextColor: '#c9d1d9',
                primaryBorderColor: '#30363d',
                lineColor: '#8b949e',
                secondaryColor: '#21262d',
                tertiaryColor: '#161b22'
            }
        });
    </script>
</body>
</html>
```

## Content Requirements

### For Each Code Section

1. **File Location Badge**: Show the relative file path
   ```html
   <div class="file-location">vllm/v1/engine/core.py:123</div>
   ```

2. **Code Blocks**: Use syntax-highlighted code blocks
   ```html
   <pre><code class="language-python">
   def method_name(self, param: Type) -> ReturnType:
       """Brief docstring."""
       # Implementation
   </code></pre>
   ```

3. **Architecture Diagrams**: Use ASCII diagrams in `.diagram` class. The content in the diagram must be English to make the structure align with the code.
   ```html
   <div class="diagram">┌─────────────┐
   │   Class A   │
   └──────┬──────┘
          │
          ▼
   ┌─────────────┐
   │   Class B   │
   └─────────────┘</div>
   ```

4. **Sequence Diagrams**: Use Mermaid syntax
   ```html
   <div class="mermaid">
   sequenceDiagram
       participant A as Actor
       participant B as Backend
       A->>B: Request
       B-->>A: Response
   </div>
   ```

5. **Important Notes**: Use highlight boxes
   ```html
   <div class="highlight-box">
       <h4>Key Point</h4>
       <p>Explanation of important concept.</p>
   </div>
   ```

6. **Language**: All the output must be in Chinese, except for diagrams.

## Code Analysis Guidelines

### Depth of Analysis

When analyzing code, focus on:

1. **Control Flow**: How does execution proceed through the method?
2. **Data Structures**: What are the key data structures and their purposes?
3. **State Changes**: What state modifications occur?
4. **Error Handling**: How are errors and edge cases handled?
5. **Performance Considerations**: Any notable optimization patterns?
6. **Ascend Adaptations**: For vllm-ascend code, highlight Ascend-specific modifications

### Tracing Rules

- **Always trace into**:
  - vLLM core methods (classes in `vllm/` directory)
  - vllm-ascend methods (classes in `vllm_ascend/` directory)
  - Custom implementations in either codebase

- **Stop tracing at**:
  - Python standard library calls
  - Third-party libraries (torch, numpy, etc.)
  - System calls and builtins

- **Document the interface** for black-box calls:
  - What parameters are passed
  - What the expected return value is
  - Why this external call is needed

## Output Organization

Organize the generated HTML files by logical grouping:

1. **index.html**: Overview, navigation, and high-level summary
2. **Numbered chapters**: Each major component or layer in the call chain
3. **Final chapter**: Complete flow summary and conclusions

Chapter naming convention: `NN-short-kebab-case-title.html`

## Project Context

This skill operates in the context of:
- **vllm-ascend**: The Ascend NPU adaptation of vLLM (current working directory)
- **vllm**: The upstream vLLM project (typically at the same level as vllm-ascend)

When analyzing code, consider:
- The relationship between vLLM core and vllm-ascend adaptation layer
- Hardware-specific implementations (Ascend NPU)
- The v1 architecture of vLLM

## Example Usage

```
User: Trace the code from EngineCore.step method

AI will:
1. Locate EngineCore.step in vllm/v1/engine/core.py
2. Analyze the method and identify all calls
3. Trace into scheduler.schedule()
4. Trace into model_executor.execute_model()
5. Continue through worker, model_runner, and model layers
6. Document vllm-ascend adaptations where applicable
7. Generate comprehensive HTML documentation
```

## Important Notes

- Always maintain the dark theme color scheme across all pages
- Ensure all code blocks have proper syntax highlighting
- Include file paths and line numbers for easy reference
- Create clear navigation between chapters
- Use diagrams to visualize complex relationships
- Write comprehensive explanations for each code section
- Highlight Ascend-specific adaptations clearly
- Cross-reference between related sections

## Execution Steps

1. **Receive entry point** from user
2. **Locate entry point** in codebase
3. **Build call tree** through recursive analysis
4. **Generate outline** showing chapter structure
5. **Confirm outline** with user before proceeding
6. **Generate styles.css** first
7. **Generate index.html** with overview
8. **Generate chapter HTML files** in sequence
9. **Verify all links** work correctly
10. **Report completion** with output directory location