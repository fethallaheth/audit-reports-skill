# Audit Reports Skill for Claude Code

A Claude Code skill for generating formatted security audit findings for Web3 smart contract audit contests including Sherlock, Code4rena, and Cantina. Create professional vulnerability reports with proper severity classification, PoC formatting, and platform-specific templates.

## What are Skills?

Skills are markdown files that give AI agents specialized knowledge and workflows for specific tasks. When you add this skill to your project, Claude Code can recognize when you're working on security audit findings and apply the right formatting and judging criteria.

## Available Skills

| Skill | Description | Triggers |
|-------|-------------|----------|
| audit-reports | Generate formatted Web3 security audit findings | "audit finding," "sherlock," "code4rena," "cantina," "vulnerability report," "security audit" |

## Installation

### Option 1: CLI Install (Recommended)

Use add-skill to install skills directly:

```bash
# Install the skill
npx skills add fethallaheth/audit-reports-skill
```

This automatically installs to your .claude/skills/ directory.


### Option 2: Clone and Copy

Clone the repo and copy to your skills directory:

```bash
git clone https://github.com/fethallaheth/audit-reports-skill.git
cp -r audit-reports ~/.config/claude-code/skills/
```

### Option 3: Direct Copy

Copy the skill folder directly:

```bash
cp -r /path/to/audit-reports ~/.config/claude-code/skills/
```

## Usage

Once installed, just ask Claude Code to help with audit findings:

```
"Generate a Sherlock finding for missing access control in pause function"
→ Uses audit-reports skill

"Format this vulnerability for Code4rena"
→ Uses audit-reports skill

"Create a Cantina report for reentrancy in Vault.sol:230 "
→ Uses audit-reports skill

"What's the severity threshold for High vs Medium?"
→ Uses audit-reports skill
```

You can also invoke the skill directly:

```
/audit-reports
```

## Supported Platforms

| Platform | Format | Severity Levels |
|----------|--------|-----------------|
| **Sherlock** | GitHub Issues | HIGH, MEDIUM |
| **Code4rena** | Web Form | High (3), Medium (2), QA (1) |
| **Cantina** | LightChaser | High, Medium, Low, Info |

## Contributing

Contributions are welcomed! Feel free to:

- Add new platform templates
- Update judging criteria as platforms evolve
- Improve finding examples
- Fix bugs or improve documentation

## License

MIT - Use this however you want. 

## About

Audit Reports helps Web3 security researchers generate properly formatted findings for smart contract audit contests.
