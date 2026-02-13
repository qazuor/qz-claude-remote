# qz-claude-remote

## Project Overview

- **Name**: qz-claude-remote
- **Type**: Tool/Utility
- **Description**: Remote Claude Code connector
- **Language**: TBD
- **Package Manager**: TBD

## Architecture

<!-- Add architecture overview here -->

## Development Guidelines

### Code Style

- Follow the conventions established in the global CLAUDE.md
- Use English for all code, comments, and documentation
- Keep files under 500 lines

### File Organization

```
/                    - Project root
  logs/              - Application logs
  .claude/           - Claude Code configuration
    agents/          - Agent definitions
    commands/        - Custom commands
    skills/          - Custom skills
    docs/            - Documentation
    reports/         - Generated reports
```

## Coding Standards

<!-- Add language-specific standards once the tech stack is defined -->

## Workflow

### Available Commands

- `/init-project` .. Initialize project configuration
- `/spec` .. Create a specification from requirements
- `/auto-loop` .. Start autonomous task processing
- `/next-task` .. Find and start the next available task

### Development Process

1. Create a spec with `/spec`
2. Review and approve the generated tasks
3. Use `/auto-loop` or `/next-task` to process tasks
4. Review changes and commit

## Context

<!-- Add project-specific context, architectural decisions, and constraints here -->
