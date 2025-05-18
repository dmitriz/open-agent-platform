# GitHub Copilot Instructions for This Repository

These instructions guide GitHub Copilot (and other AI assistants) to ensure every contribution is clean, consistent, and free of warnings.

## Core Rules

1. **Always run lint and error checks after edits**  
   Use the `get_errors` tool (or `npm run lint`) to verify that all modified files pass without errors or warnings.

2. **Fix all warnings before completing any task**  
   Do not consider a task complete if any linting or formatting warnings remain.

3. **Preserve existing content**  
   Never remove or truncate sections unless explicitly instructed by the user. Focus on correcting syntax, formatting, and structure.

4. **Report unfixable warnings**  
   If a warning cannot be resolved, clearly explain the issue and why it cannot be fixed.

5. **Prevent duplicate documentation**  
   Avoid copying or repeating sections. Merge or refactor content instead of duplicating.

6. **Enforce code block conventions**  
   - Always specify the correct language for fenced code blocks (`bash`, `javascript`, `json`, `text`, etc.).  
   - Maintain proper indentation and blank lines around code blocks.

## Workflow Enforcement

- Check for errors after each edit using the `get_errors` tool.  
- Only finalize changes when all checks pass.  
- If you cannot resolve a warning, notify the user and await further instructions.

## Reference Configuration

These behaviors are supported by the linting rules in `.linting-rules.md`.

Ensure all contributions adhere to these instructions to maintain high-quality standards across the project.
