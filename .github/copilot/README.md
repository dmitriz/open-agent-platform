# GitHub Copilot Rules

This file documents the rules configured in `.github/copilot/command-settings.json` that govern AI assistant behavior in this repository.

## Mandatory Behaviors

1. **Always check for warnings and errors after every file edit**
   - AI assistants must use tools like `get_errors` to verify changes are error-free
   
2. **Fix all warnings before completing tasks**
   - No task should be considered complete until all warnings are resolved
   
3. **Preserve existing content**
   - Never remove or truncate content unless explicitly instructed
   - Focus on fixing syntax and formatting without content loss
   
4. **Report unfixable warnings**
   - If a warning cannot be fixed, provide a clear explanation to the user
   
5. **Prevent duplicate content**
   - Documentation should never contain duplicated sections
   
6. **Enforce code block standards**
   - All code blocks must have appropriate language specifiers
   - Proper indentation and syntax highlighting must be maintained

## Implementation

These rules are enforced through the configuration in `.github/copilot/command-settings.json`. This ensures that:

- All contributions maintain code and documentation quality
- Linting standards are consistently applied
- Content is preserved while fixing formatting issues
- Warnings are always addressed before task completion

Refer to the [`.linting-rules.md`](.linting-rules.md) file for specific linting rules and quality standards.
