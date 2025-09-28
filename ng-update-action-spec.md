# ng-update-action Specification

This document specifies the requirements for creating an Angular CLI update GitHub Action based on the nx-migrate-action pattern (https://github.com/gridatek/nx-migrate-action).

## Project Overview

Create a GitHub Action that automatically updates Angular CLI workspaces to the latest version with validation and smart PR creation. The action should be implemented as a composite action using shell scripts, following the same patterns as nx-migrate-action.

## Core Functionality

### Command Mapping
- Replace `nx migrate latest` → `ng update @angular/core @angular/cli`
- Replace `nx run-many --target=build` → `ng build` (for all projects) or specific project builds
- Replace `nx run-many --target=test` → `ng test` (for all projects) or specific project tests
- No migrations.json handling needed (Angular CLI handles migrations internally)

### Version Detection
- Target package: `@angular/core` (default)
- Support version tags: `latest`, `next`, `rc`
- Support specific versions: `17.3.0`, `18.0.0`, etc.
- Use same package manager detection logic as nx-migrate-action

### Branch Naming
- Production mode: `ng-update-17.3.0`
- Dev mode: `ng-update-17.3.0-npm-node22-{run-id}-{job-id}`

## Required Files

### 1. action.yml
Composite action with these inputs:

```yaml
inputs:
  angular-package:
    description: 'The Angular package to check for updates'
    required: false
    default: '@angular/core'

  angular-version:
    description: 'Angular version to use (latest, next, rc, or specific version like 17.3.0)'
    required: false
    default: 'latest'

  additional-packages:
    description: 'Additional Angular packages to update (comma-separated)'
    required: false
    default: '@angular/cli'

  package-manager:
    description: 'Package manager to use (npm, yarn, pnpm)'
    required: false
    default: 'npm'

  validation-commands:
    description: 'Commands to run for validation (comma-separated)'
    required: false
    default: 'build'

  affected:
    description: 'Only validate affected projects (true) or all projects (false)'
    required: false
    default: 'true'

  merge-strategy:
    description: 'Merge strategy after validation (auto-merge, always-pr)'
    required: false
    default: 'auto-merge'

  pr-labels:
    description: 'Labels to add to PRs (comma-separated)'
    required: false
    default: 'ng-update-action'

  commit-message-prefix:
    description: 'Prefix for commit messages'
    required: false
    default: 'build'

  target-branch:
    description: 'Target branch for merging changes'
    required: false
    default: 'main'

  working-directory:
    description: 'Working directory'
    required: false
    default: '.'

  skip-initial-install:
    description: 'Skip initial dependency installation'
    required: false
    default: 'false'

  dev-mode:
    description: 'Enable dev mode for testing (creates unique branches with matrix info)'
    required: false
    default: 'false'

outputs:
  updated:
    description: 'Whether Angular was updated'

  current-version:
    description: 'Current Angular version before update'

  latest-version:
    description: 'Latest Angular version'

  validation-result:
    description: 'Result of validation tests'

  pr-url:
    description: 'URL of created PR (if any)'
```

### 2. Action Steps (based on nx-migrate-action patterns)

1. **Setup Dependencies** - Install dependencies if not skipped
2. **Check Angular Version** - Compare current vs target version using package manager commands
3. **Create Branch** - Handle dev-mode vs production mode branching
4. **Run ng update** - Execute `ng update` commands for specified packages
5. **Install Dependencies** - Reinstall dependencies after updates
6. **Commit Changes** - Commit package.json and other updates
7. **Validation** - Run Angular build/test/lint commands
8. **Smart Merge/PR** - Auto-merge or create PR based on validation results

### 3. Angular-Specific Logic

#### Version Detection
```bash
# For npm
npm view @angular/core@latest version

# For yarn
yarn info @angular/core@latest --json | jq -r '.data.version'

# For pnpm
pnpm view @angular/core@latest version
```

#### Update Commands
```bash
# Basic update
ng update @angular/core @angular/cli

# With specific version
ng update @angular/core@17.3.0 @angular/cli@17.3.0

# With additional packages
ng update @angular/core @angular/cli @angular/material
```

#### Validation Commands
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

### 4. README.md Structure

#### Sections to include:
- **Features** - Angular CLI update automation, validation, smart PR creation
- **Quick Start** - Basic usage examples
- **Configuration Options** - All inputs with descriptions
- **Usage Examples** - Different package managers, strategies, Angular packages
- **How It Works** - Flowchart showing Angular update process
- **Troubleshooting** - Common Angular CLI issues

#### Example Usage:
```yaml
- uses: gridatek/ng-update-action@v0
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    package-manager: npm
    validation-commands: 'build,test'
    additional-packages: '@angular/material,@angular/cdk'
```

### 5. Test Workflows

Create matrix testing for:
- Package managers: npm, yarn, pnpm
- Node.js versions: 18, 20, 22
- Angular versions: Test upgrading from older versions

Test setup should:
1. Create temporary Angular workspace with older version
2. Install dependencies
3. Run the action
4. Verify successful update

### 6. Supporting Files

- **package.json** - Basic Node.js metadata
- **.yamllint.yml** - YAML linting configuration
- **CLAUDE.md** - Development guidelines specific to Angular CLI
- **LICENSE** - MIT license
- **.gitignore** - Standard Node.js gitignore

## Key Differences from nx-migrate-action

1. **No migrations.json handling** - Angular CLI handles migrations internally
2. **Different validation commands** - Use `ng build`, `ng test` instead of `nx run-many`
3. **Package focus** - Target `@angular/core` and Angular ecosystem packages
4. **Simpler workflow** - No separate migration execution step needed
5. **Angular CLI compatibility** - Handle Angular CLI version requirements

## Angular CLI Considerations

- Angular CLI requires specific Node.js versions for different Angular versions
- Some Angular updates require multiple steps (e.g., major version updates)
- Angular CLI may prompt for user input during updates (handle with --force flag)
- Angular workspaces may have multiple projects requiring different validation strategies

## Branch Protection & PR Handling

Follow same patterns as nx-migrate-action:
- Respect branch protection rules
- Create detailed PRs with update summary
- Handle validation failures gracefully
- Support auto-merge for successful updates

## Environment Variables

Use `GITHUB_TOKEN` environment variable (not input parameter) following the same pattern as nx-migrate-action.

## Implementation Notes

- Use composite action approach (shell scripts in action.yml)
- Support all three major package managers
- Handle edge cases (no changes, validation failures, missing packages)
- Provide comprehensive logging throughout the process
- Follow semantic versioning for action releases

This specification should provide enough detail for Claude to generate a complete ng-update-action following the established patterns from nx-migrate-action.