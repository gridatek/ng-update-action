# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a GitHub Action called `ng-update-action` that automatically updates Angular CLI workspaces to the latest version with validation and smart PR creation. It's implemented as a composite action using shell scripts, based on the nx-migrate-action pattern.

## Required Files to Implement

### Core Action Files
- `action.yml` - Main composite action configuration with inputs/outputs
- Shell scripts (typically embedded in action.yml steps) for:
  - Version detection and comparison
  - Angular CLI update execution
  - Validation command execution
  - Branch and PR management

### Documentation
- `README.md` - User-facing documentation with examples
- `package.json` - Basic Node.js metadata for the action

### Configuration Files
- `.yamllint.yml` - YAML linting configuration
- `.gitignore` - Standard Node.js gitignore
- `LICENSE` - MIT license

### Testing
- `.github/workflows/` - Matrix testing workflows for different package managers (npm, yarn, pnpm) and Node.js versions (18, 20, 22)

## Key Angular CLI Commands

### Version Detection
```bash
# npm
npm view @angular/core@latest version

# yarn
yarn info @angular/core@latest --json | jq -r '.data.version'

# pnpm
pnpm view @angular/core@latest version
```

### Update Commands
```bash
# Basic update
ng update @angular/core @angular/cli

# With specific version
ng update @angular/core@17.3.0 @angular/cli@17.3.0

# With additional packages
ng update @angular/core @angular/cli @angular/material
```

### Validation Commands
```bash
# Build validation
ng build --configuration production

# Test validation
ng test --watch=false --browsers=ChromeHeadless

# Lint validation
ng lint

# E2E validation
ng e2e
```

## Action Architecture

The action follows this workflow:
1. **Setup Dependencies** - Install dependencies if not skipped
2. **Check Angular Version** - Compare current vs target version
3. **Create Branch** - Handle dev-mode vs production mode branching
4. **Run ng update** - Execute update commands for specified packages
5. **Install Dependencies** - Reinstall dependencies after updates
6. **Commit Changes** - Commit package.json and other updates
7. **Validation** - Run Angular build/test/lint commands
8. **Smart Merge/PR** - Auto-merge or create PR based on validation results

## Branch Naming Convention
- Production mode: `ng-update-17.3.0`
- Dev mode: `ng-update-17.3.0-npm-node22-{run-id}-{job-id}`

## Key Differences from nx-migrate-action
- No migrations.json handling (Angular CLI handles migrations internally)
- Use `ng build`, `ng test` instead of `nx run-many` commands
- Target `@angular/core` and Angular ecosystem packages
- Simpler workflow without separate migration execution step
- Handle Angular CLI version requirements and Node.js compatibility

## Important Implementation Notes

### Package Manager Support
- Support npm, yarn, and pnpm detection and usage
- Use package manager-specific commands for version detection and installation

### Angular CLI Considerations
- Angular CLI requires specific Node.js versions for different Angular versions
- Some Angular updates require multiple steps (major version updates)
- Handle Angular CLI prompts with --force flag when needed
- Angular workspaces may have multiple projects requiring different validation strategies

### Environment Variables
- Use `GITHUB_TOKEN` environment variable (not input parameter)
- Follow the same GitHub API patterns as nx-migrate-action

### Error Handling
- Handle edge cases: no changes, validation failures, missing packages
- Provide comprehensive logging throughout the process
- Respect branch protection rules
- Create detailed PRs with update summary
- Handle validation failures gracefully

## Testing Strategy

Matrix testing should cover:
- Package managers: npm, yarn, pnpm
- Node.js versions: 18, 20, 22
- Angular versions: Test upgrading from older versions

Test workflow should:
1. Create temporary Angular workspace with older version
2. Install dependencies
3. Run the action
4. Verify successful update