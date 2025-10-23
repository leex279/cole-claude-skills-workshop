# Claude Code Skills - Complete Workshop Guide

Welcome to the Claude Code Skills showcase! This project provides a hands-on introduction to **Claude Code Skills** - a powerful way to extend Claude's capabilities with your own custom workflows and expertise.

## Table of Contents

- [What Are Claude Code Skills?](#what-are-claude-code-skills)
- [Why Use Skills?](#why-use-skills)
- [How Skills Work](#how-skills-work)
- [Anatomy of a Skill](#anatomy-of-a-skill)
- [Project vs Personal Skills](#project-vs-personal-skills)
---

## What Are Claude Code Skills?

**Skills** are modular capabilities that extend Claude Code's functionality. Think of them as specialized "expertise modules" that teach Claude how to handle specific tasks or workflows.

### Key Characteristics

- **Model-Invoked**: Claude automatically decides when to use a skill based on context (unlike slash commands which you manually trigger)
- **Modular**: Each skill is a self-contained folder with instructions and resources
- **Shareable**: Skills can be shared via git with your team or community
- **Composable**: Multiple skills can work together to handle complex tasks

### Real-World Analogy

If Claude Code is a smart assistant, **Skills** are like specialized training manuals that teach the assistant new capabilities:
- A "PDF Extraction" skill teaches Claude how to parse and extract data from PDFs
- An "API Scraper" skill teaches Claude how to fetch and process web data
- A "Test Automation" skill teaches Claude your preferred testing patterns

---

## Why Use Skills?

### 1. **Reduce Repetitive Prompting**
Instead of explaining the same workflow every time, encode it once as a skill.

**Without Skills:**
```
You: "Please extract the data from this PDF, format it as JSON, and validate the fields..."
(Repeat this explanation every time you work with PDFs)
```

**With Skills:**
```
You: "Process this PDF file"
Claude: (Automatically activates pdf-extractor skill and follows your predefined workflow)
```

### 2. **Share Team Expertise**
Capture your team's best practices and workflows in skills that everyone can use.

**Example:** Your team's testing conventions, code generation templates, or API integration patterns.

### 3. **Extend Claude's Capabilities**
Add specialized knowledge that Claude doesn't have by default.

**Examples:**
- Company-specific workflows
- Domain-specific data processing
- Custom code generation patterns
- Integration with proprietary tools

### 4. **Compose Complex Workflows**
Combine multiple skills to handle sophisticated tasks.

**Example:** Use `api-scraper` to fetch data, `data-processor` to transform it, and `report-generator` to create output.

---

## How Skills Work

### Model-Invoked Behavior

Skills are **automatically activated** by Claude based on the context of your request. You don't need to explicitly call them (though you can reference them if needed).

**How Claude Decides:**
1. You make a request: "Extract tables from this PDF"
2. Claude reads all available skill descriptions
3. Claude finds `pdf-extractor` skill has a description mentioning "PDF" and "tables"
4. Claude activates the skill and follows its instructions

### Skill Activation Flow

```
User Request → Claude Analyzes Context → Matches Skill Description → Activates Skill → Follows Instructions
```

### Progressive Disclosure

Claude reads the main `SKILL.md` file initially, but only loads additional resources (templates, examples, scripts) when needed. This keeps context efficient.

![Progressive Context Loading](assets/image.png)
*Source: [Dani Avila on X](https://x.com/dani_avila7/status/1981394535155438029)*

**How progressive loading works:**

1. **Main Context** - Always loaded: Your project's CLAUDE.md and basic context
2. **Skill Discovery** - When you make a request, Claude scans all skill descriptions to find matches
3. **Skill Activation** - Once a skill is selected, Claude loads it in three stages:
   - **SKILL.md** - Always loaded immediately when the skill activates
   - **Referenced files** - Loaded only when needed (examples.md, reference.md, etc.)
   - **Supporting files** - NOT pre-loaded; accessed directly when executed (scripts, templates)

This progressive approach means skills can include extensive resources without overwhelming Claude's context window.

---

## Anatomy of a Skill

Every skill is a **directory** containing at minimum a `SKILL.md` file with YAML frontmatter.

### Minimal Skill Structure

```
my-skill/
└── SKILL.md
```

### Complete Skill Structure

```
my-skill/
├── SKILL.md           # Required: Main skill file with frontmatter and instructions
├── examples.md        # Optional: Example usage and outputs
├── reference.md       # Optional: Additional documentation
├── templates/         # Optional: Template files
│   └── template.txt
└── scripts/           # Optional: Helper scripts
    └── helper.py
```

### SKILL.md Format

Every `SKILL.md` must start with YAML frontmatter:

```yaml
---
name: my-skill-name
description: Brief description of what this skill does and when to use it. Include trigger keywords.
allowed-tools: Read, Write, Grep, Glob  # Optional: restrict which tools Claude can use
---

# Skill Instructions

Your detailed instructions go here in markdown format.

## What This Skill Does

Explain the skill's purpose and capabilities.

## When to Use

Explain the scenarios where this skill should be activated.

## Instructions

Step-by-step instructions for Claude to follow.

## Examples

Show example inputs and expected outputs.
```

### YAML Frontmatter Fields

| Field | Required | Format | Description |
|-------|----------|--------|-------------|
| `name` | ✅ Yes | lowercase-with-hyphens | Unique identifier (max 64 chars) |
| `description` | ✅ Yes | Plain text | What the skill does + when to use it (max 1024 chars) |
| `allowed-tools` | ❌ No | Comma-separated list | Restrict which tools Claude can use when skill is active |

### Writing Effective Descriptions

The `description` field is **critical** - it's how Claude determines when to activate your skill.

**Good Description (Specific):**
```yaml
description: Extract text and tables from PDF files, fill PDF forms, and merge PDF documents. Use when the user mentions PDFs, forms, document extraction, or table parsing.
```

**Bad Description (Too Generic):**
```yaml
description: Helps with documents
```

**Tips:**
- Include the **what** (what does it do?)
- Include the **when** (trigger keywords)
- Be specific about capabilities
- Mention file types, use cases, or domain terms

---

## Project vs Personal Skills

Skills can be stored in **two locations**, each with different purposes:

### Project Skills: `.claude/skills/`

**Location:** `.claude/skills/my-skill/` (in your project repository)

**Purpose:** Team-shared skills that are version-controlled with your project

**Use When:**
- Building team workflows
- Sharing best practices across your organization
- Creating project-specific conventions
- Skills that should be available to all team members

**Benefits:**
- ✅ Automatically available to all team members
- ✅ Version controlled via git
- ✅ Shared across the entire team
- ✅ Changes can be reviewed via pull requests

**Example:**
```bash
my-project/
├── .claude/
│   └── skills/
│       ├── company-api-client/   # How to interact with company APIs
│       ├── test-patterns/        # Team's testing conventions
│       └── code-standards/       # Code generation templates
└── src/
```

### Personal Skills: `~/.claude/skills/`

**Location:** `~/.claude/skills/my-skill/` (in your home directory)

**Purpose:** Individual workflows available across all your projects

**Use When:**
- Personal productivity workflows
- Experimental skills you're testing
- Skills specific to your work style
- Tools you use across many projects

**Benefits:**
- ✅ Available in every project you work on
- ✅ No need to commit to project repository
- ✅ Personal customization
- ✅ Quick iteration and experimentation

**Example:**
```bash
~/.claude/
└── skills/
    ├── my-commit-style/      # Personal git commit conventions
    ├── my-code-review/       # Personal code review checklist
    └── my-documentation/     # Personal doc writing style
```

### Which Should I Use?

| Scenario | Location | Reason |
|----------|----------|--------|
| Team coding standards | Project | Everyone should follow the same standards |
| Your personal git commit style | Personal | Individual preference |
| Company API integration patterns | Project | Shared knowledge for all developers |
| Your favorite debugging workflow | Personal | Personal productivity tool |
| Project-specific testing patterns | Project | Consistent testing across the team |
| Experimental new workflow | Personal | Test before sharing with team |

---

## Installing Skills from the Community

The Claude Code community has created a growing library of ready-to-use skills that you can install with a single command. Visit [aitmpl.com/skills](https://www.aitmpl.com/skills) to browse available skills.

### How to Install Community Skills

1. **Browse Available Skills**
   - Visit [https://www.aitmpl.com/skills](https://www.aitmpl.com/skills)
   - Explore skills by category: Development, Productivity, Data Processing, etc.
   - Read the skill description to understand what it does

2. **Install a Skill**

   Each skill on aitmpl.com provides a one-line installation command. Simply copy and run it:

   ```bash
   # Example: Installing the skill-creator skill
   npx claude-code-templates@latest --skill=development/skill-creator --yes
   ```

   The `--yes` flag automatically installs the skill without additional prompts.

3. **Verify Installation**

   After installation, the skill files will be added to your `.claude/skills/` directory and will be automatically available to Claude Code.

### Popular Community Skills

Here are some of the most downloaded skills from the community (19 skills available):

**Development:**
| Skill | Description | Install Command |
|-------|-------------|-----------------|
| **skill-creator** | Guide for creating effective skills | `npx claude-code-templates@latest --skill=development/skill-creator --yes` |
| **webapp-testing** | Toolkit for testing web applications using Playwright | `npx claude-code-templates@latest --skill=development/webapp-testing --yes` |
| **git-commit-helper** | Generate descriptive commit messages by analyzing git diffs | `npx claude-code-templates@latest --skill=development/git-commit-helper --yes` |
| **mcp-builder** | Guide for creating MCP (Model Context Protocol) servers | `npx claude-code-templates@latest --skill=development/mcp-builder --yes` |
| **artifacts-builder** | Create multi-component HTML artifacts using modern web technologies | `npx claude-code-templates@latest --skill=development/artifacts-builder --yes` |

**Document Processing:**
| Skill | Description | Install Command |
|-------|-------------|-----------------|
| **docx** | Create, edit, and analyze Word documents with tracked changes | `npx claude-code-templates@latest --skill=document-processing/docx --yes` |
| **xlsx** | Spreadsheet creation, editing, and analysis with formulas | `npx claude-code-templates@latest --skill=document-processing/xlsx --yes` |
| **pdf-processing-pro** | Production-ready PDF processing with forms, tables, OCR | `npx claude-code-templates@latest --skill=document-processing/pdf-processing-pro --yes` |
| **pptx** | Presentation creation, editing, and analysis | `npx claude-code-templates@latest --skill=document-processing/pptx --yes` |

**Creative Design:**
| Skill | Description | Install Command |
|-------|-------------|-----------------|
| **canvas-design** | Create visual art in PNG and PDF documents | `npx claude-code-templates@latest --skill=creative-design/canvas-design --yes` |
| **algorithmic-art** | Create algorithmic art using p5.js | `npx claude-code-templates@latest --skill=creative-design/algorithmic-art --yes` |

**Enterprise Communication:**
| Skill | Description | Install Command |
|-------|-------------|-----------------|
| **excel-analysis** | Analyze Excel spreadsheets, create pivot tables, generate charts | `npx claude-code-templates@latest --skill=enterprise-communication/excel-analysis --yes` |
| **email-composer** | Draft professional emails for various contexts | `npx claude-code-templates@latest --skill=enterprise-communication/email-composer --yes` |

### Managing Installed Skills

Once installed, skills are stored in your `.claude/skills/` directory. You can:

- **View installed skills**: Check the `.claude/skills/` directory in your project
- **Update a skill**: Re-run the installation command to get the latest version
- **Remove a skill**: Delete the skill directory from `.claude/skills/`
- **Browse categories**: Visit aitmpl.com/skills and filter by Development, Document Processing, Creative Design, or Enterprise Communication

---
