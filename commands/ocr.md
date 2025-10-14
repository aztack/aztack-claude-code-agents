---
description: Analyzes a UI image and generates a UI Component Tree JSON.
argument-hint: <image-path>, --ref <ref-ui-json>, --prd <prd-md>, --style <true|false>
---

You are tasked with analyzing a UI design image and generating a structured UI Component Tree JSON file.

## Instructions

1. **Identify the image path**: If the user provides an image path, use it. If not, ask for the image path.

2. **Use the d2c-generator agent**: Launch the d2c-generator agent with the Task tool to perform OCR analysis on the provided image.

3. **Pass relevant options**:
   - If there's a reference UI JSON (e.g., layout-frame.ui.json), pass it via `options.ref`
   - If there's a PRD markdown file, pass it via `options.prd`
   - Set `options.style` to control whether to generate style props

4. **Output**: The agent will generate a `<image-name>.ui.json` file beside the image following the UI Component Tree Schema defined in the d2c-generator agent.

## Usage Examples

```
/ocr page1.png
/ocr layout-frame.png
/ocr page2.png --ref layout-frame.ui.json
/ocr dashboard.png --prd requirements.md
```

## What the agent will do

- Identify the layout type from the predefined layout list
- Recognize Semi Design components that can be reused
- Identify custom components that need implementation
- Generate a stable `identity` object for each component (category, variant, structure)
- Output a JSON file with the complete component tree structure
- Record necessary props sufficient to implement React components

Launch the d2c-generator agent now to analyze the UI image.
