---
description: Implements React components from a UI JSON file.
argument-hint: <ui-json-path>, <output-dir-or-file>
---

You are tasked with implementing React components based on a UI Component Tree JSON file.

## Instructions

1. **Identify the UI JSON file**: If the user provides a `.ui.json` file path, use it. If not, ask for the file path.

2. **Identify the output directory/file**: If the user specifies where to output the component code, use it. Otherwise, infer a sensible location (e.g., `src/components/<component-name>/`).

3. **Use the d2c-generator agent**: Launch the d2c-generator agent with the Task tool to implement React components from the UI JSON.

4. **Component generation requirements**:
   - Generate functional React components with TypeScript
   - Create accompanying SCSS files using BEM methodology
   - Import appropriate components from Semi Design (`@douyinfe/semi-ui`, `@douyinfe/semi-icons`)
   - Infer Props interface from component structure
   - Implement state management, event handlers, and effects as needed
   - Use lucide-icons MCP to search for best-matching icons when needed
   - Review and optimize import statements to avoid duplication
   - Ensure light/dark theme compatibility

5. **Output structure**:
   ```
   <component-name>/
   ├── index.tsx       # React component with Props interface
   └── index.scss      # BEM-style SCSS
   ```

## Usage Examples

```
/implComponents page1.ui.json
/implComponents layout-frame.ui.json src/layouts/
/implComponents dashboard.ui.json ./components/Dashboard
```

## What the agent will do

- Parse the UI JSON file and extract component definitions
- Generate TypeScript React component files with:
  - Proper Props interfaces extending React.HTMLAttributes
  - State management (useState, useEffect, etc.)
  - Event handlers
  - Semi Design component imports
  - Icon components (from Semi Design or custom)
- Generate SCSS files with:
  - BEM methodology (block__element--modifier)
  - Theme-aware styles using CSS variables
  - Responsive design considerations
- Review all imports for correctness and availability
- Update the UI JSON file's `src` and `impl` fields after implementation

Launch the d2c-generator agent now to implement the components.
