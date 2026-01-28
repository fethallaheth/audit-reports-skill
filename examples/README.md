# Finding Report Examples

Real-world examples of well-written audit findings for each platform. Use these as references when writing your own reports.

## Available Examples

| Platform | File | Description |
|----------|------|-------------|
| [Sherlock](./sherlock.md) | Missing access control in reward distribution |
| [Code4rena](./code4rena.md) | Flash loan price manipulation via TWAP oracle |
| [Cantina](./cantina.md) | Share price inflation attack |

## Tips for Writing Good Findings

1. **Start with impact** - Judges need to immediately understand the severity
2. **Be specific about root cause** - Don't just describe symptoms; explain the underlying bug
3. **Provide executable PoC** - Tests should compile and pass/fail clearly
4. **Include file:line references** - Make it easy to locate the vulnerable code
5. **Suggest concrete fixes** - Provide code, not just descriptions
6. **Use clear formatting** - Bold key values, use code blocks, organize with headers
7. **Avoid assumptions** - Don't assume complex external conditions for HIGH severity
