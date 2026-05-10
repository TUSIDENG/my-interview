# Project: My Interview Notes

This project is a collection of interview questions and technical documentation, built using the [Hugo Book](https://github.com/alex-shpak/hugo-book) theme.

## Standards and Conventions

### 1. Content Format
- All content must be written in **Markdown**.
- Adhere to Hugo Book's organizational structure (content resides in the `content/` directory).
- Use descriptive filenames and front matter (YAML) for metadata.

### 2. Diagramming
We use two primary tools for visualization:

#### Mermaid (In-line)
- Use **Mermaid** for standard diagrams (flowcharts, sequence diagrams, state diagrams, etc.) that can be rendered directly in the browser.
- Mermaid code blocks should be wrapped in Hugo shortcodes or standard markdown code blocks as supported by the Hugo Book theme.

#### Python Diagrams (Architectural)
- Use the [Python Diagrams](https://diagrams.mingrammer.com/) library for complex system architectures, cloud infrastructure, and detailed technical mappings.
- **Workflow**:
  - Source code for these diagrams should be stored in the `/scripts/diagrams/` directory.
  - Generated image files (PNG/SVG) should be stored in `/static/images/diagrams/`.
  - Link the generated images in Markdown using standard syntax: `![Description](/images/diagrams/filename.png)`.

### 3. Technical Standards
- Maintain clean, hierarchical navigation.
- Ensure all diagrams have appropriate alt-text and descriptions.

## AI Assistance Instructions
When assisting with this project (Gemini CLI, GitHub Copilot, Codex, etc.):
- **Context Awareness**: Always consider the Hugo Book theme constraints.
- **Visualization**:
    - Default to **Mermaid** for logical flow and simple relationships.hugo-book Support Mermaid.
    - Use **Python Diagrams** when representing complex infrastructure or when precise control over icons (AWS, K8s, etc.) is needed.
- **Consistency**: Follow existing naming conventions for files and image assets.
