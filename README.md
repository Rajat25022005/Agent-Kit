<div align="center">

# Agent Skills

**A curated collection of 66 agent skills across 16 domains,**
**designed for autonomous AI agents and assistants.**

---

`autonomous-ai-agents` | `creative` | `data-science` | `devops` | `software-development`
`research` | `productivity` | `mlops` | `github` | `media` | `email` | `smart-home` | `social-media` | `note-taking` | `dogfood` | `yuanbao`

---

</div>

## Overview

This repository is a structured library of reusable agent skills -- modular capabilities that can be composed, extended, and executed by AI-powered systems. Each skill is self-contained within its own directory and includes documentation, references, and implementation details.

Skills are organized into **domain-specific categories**, making it easy to discover, contribute, and integrate new capabilities.

---

## Skill Categories

### AI and Automation

| Category | Skills | Description |
|:---|:---:|:---|
| [autonomous-ai-agents](./autonomous-ai-agents/) | 5 | Building, managing, and orchestrating autonomous AI agents |
| [software-development](./software-development/) | 9 | Code generation, debugging, testing, and engineering workflows |
| [mlops](./mlops/) | 4 | Model deployment, monitoring, and ML pipeline management |
| [devops](./devops/) | 2 | CI/CD, infrastructure provisioning, and deployment automation |

### Content and Design

| Category | Skills | Description |
|:---|:---:|:---|
| [creative](./creative/) | 16 | ASCII art, visual design, diagrams, music, and generative tools |
| [media](./media/) | 4 | Audio, video, and image processing capabilities |
| [social-media](./social-media/) | 1 | Content creation and social platform management |

### Knowledge and Communication

| Category | Skills | Description |
|:---|:---:|:---|
| [research](./research/) | 5 | Web search, information synthesis, and deep research |
| [email](./email/) | 1 | Email generation, parsing, and inbox management |
| [note-taking](./note-taking/) | 1 | Integration with note-taking systems and knowledge bases |

### Productivity and Data

| Category | Skills | Description |
|:---|:---:|:---|
| [productivity](./productivity/) | 8 | Task management, scheduling, and personal workflows |
| [data-science](./data-science/) | 1 | Data processing, analysis, and visualization |
| [github](./github/) | 6 | Repository management, code review, and issue tracking |

### Integrations

| Category | Skills | Description |
|:---|:---:|:---|
| [smart-home](./smart-home/) | 1 | Home automation and IoT device control |
| [yuanbao](./yuanbao/) | -- | Platform-specific integrations |
| [dogfood](./dogfood/) | 2 | Internal testing and self-hosting utilities |

---

## Structure

```
agent-skills/
    <category>/
        DESCRIPTION.md          # Category overview
        <skill-name>/
            SKILL.md            # Skill definition and instructions
            references/         # Supporting documentation
            ...
```

Each category contains a `DESCRIPTION.md` with a high-level overview. Individual skills live in subdirectories and follow a consistent structure with a `SKILL.md` entry point.

---

<div align="center">

*Each skill is a building block. Compose them to create something greater.*

</div>
