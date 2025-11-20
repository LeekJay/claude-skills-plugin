---
name: frontend-architecture-design
description: Review and redesign frontend project architecture, analyzing performance, maintainability, scalability, engineering, and type safety. Supports React, Vue, and other tech stacks. Use when architecture review, code structure analysis, architecture redesign, performance optimization, or architectural improvements are needed. Also use when the user mentions architecture is unreasonable, needs restructuring, wants to refactor, or improve code organization.
allowed-tools: Read, Grep, Glob
---

# Frontend Architecture Design & Review

Comprehensive frontend project architecture review from five key dimensions: performance, maintainability, scalability, engineering, and type safety.

## Core Review Dimensions

### 1. Performance Optimization
- Code splitting and lazy loading strategies
- Resource loading and caching mechanisms
- Runtime performance optimization
- Build output analysis

See [PERFORMANCE.md](PERFORMANCE.md) for details

### 2. Maintainability
- Directory structure and code organization
- Modularization and separation of concerns
- Design pattern applications
- Code reuse strategies

See [MAINTAINABILITY.md](MAINTAINABILITY.md) for details

### 3. Scalability
- Plugin architecture design
- Micro-frontend architecture
- Component library and utility library design
- API design and abstraction layers

See [SCALABILITY.md](SCALABILITY.md) for details

### 4. Engineering
- Build tool configuration
- Code standards and quality control
- CI/CD integration
- Developer experience optimization

See [ENGINEERING.md](ENGINEERING.md) for details

### 5. Type Safety (TypeScript)
- TypeScript configuration
- Type system design
- Type safety practices
- Type utility usage

See [TYPESCRIPT.md](TYPESCRIPT.md) for details

## Architecture Review Process

### Step 1: Project Overview
1. Use Glob to find project configuration files:
   - `package.json` - Dependencies and scripts
   - `tsconfig.json` - TypeScript configuration
   - `vite.config.*` / `webpack.config.*` - Build configuration
   - `.eslintrc.*` / `prettier.config.*` - Code standards

2. Use Read to examine configuration files and understand:
   - Tech stack selection (React/Vue/other)
   - Main dependencies and versions
   - Build tools and plugins
   - TypeScript strictness level

3. Use Glob to review directory structure:
   - `src/**/*` - Source code organization
   - `public/**/*` - Static assets
   - `tests/**/*` - Test coverage

### Step 2: Dimension-Specific Deep Analysis

Choose relevant dimensions based on project needs for in-depth analysis:

**Performance Analysis**:
- Check routing configuration for lazy loading usage
- Analyze dependency sizes in `package.json`
- Find performance optimization code (memo, useMemo, computed)
- Review static asset handling strategies

**Maintainability Analysis**:
- Evaluate directory structure rationality
- Check component/module responsibility separation
- Find code duplication and refactoring opportunities
- Assess state management approach

**Scalability Analysis**:
- Check for plugin systems or extension mechanisms
- Evaluate API abstraction layer design
- Review component configurability and reusability
- Analyze coupling between modules

**Engineering Analysis**:
- Check build configuration rationality
- Evaluate code standard tool configuration
- Review Git hooks and CI configuration
- Analyze development script completeness

**Type Safety Analysis**:
- Check tsconfig.json strictness
- Find usage of `any` type
- Evaluate type definition completeness
- Review type utility usage

### Step 3: Generate Review Report

Organize findings and improvement suggestions in the following structure:

```markdown
# Frontend Architecture Review Report

## Project Overview
- Tech Stack: [React/Vue/...]
- TypeScript Version: [version]
- Main Dependencies: [list key dependencies]

## Architecture Scores

| Dimension | Score | Notes |
|-----------|-------|-------|
| Performance | ⭐⭐⭐⭐☆ | Brief explanation |
| Maintainability | ⭐⭐⭐☆☆ | Brief explanation |
| Scalability | ⭐⭐⭐⭐⭐ | Brief explanation |
| Engineering | ⭐⭐⭐⭐☆ | Brief explanation |
| Type Safety | ⭐⭐⭐☆☆ | Brief explanation |

## Issues Found

### Performance Related
1. **Issue Description**
   - Location: `src/path/to/file.ts:123`
   - Impact: Medium
   - Recommendation: Specific improvement approach

### Maintainability Related
[...]

### Scalability Related
[...]

### Engineering Related
[...]

### Type Safety Related
[...]

## Improvement Priorities

### P0 (Must fix immediately)
- [ ] Issue description and location

### P1 (Should fix short-term)
- [ ] Issue description and location

### P2 (Medium to long-term optimization)
- [ ] Issue description and location

## Architecture Recommendations

### Short-term Improvements
1. Specific recommendations with implementation steps

### Medium-term Optimizations
1. Specific recommendations with implementation steps

### Long-term Planning
1. Specific recommendations with implementation steps
```

## Review Best Practices

### Systematic Analysis
- Don't just look at surface issues, analyze root causes
- Focus on architectural decision trade-offs
- Consider team size and project stage
- Provide recommendations aligned with business needs

### Objective Evaluation
- Use specific code locations and examples
- Provide quantifiable metrics (bundle size, build time)
- Distinguish issue severity levels
- Offer actionable improvement plans

### Progressive Improvement
- Don't propose too many improvements at once
- Prioritize issues with highest impact
- Consider risk and cost of changes
- Provide phased implementation plans

### Framework-Agnostic Principles
- Use universal architectural principles and patterns
- Provide framework-specific practices when needed
- Focus on underlying design philosophy rather than specific implementation
- Compare multiple technical approaches when appropriate

## Common Architecture Anti-patterns

### Anti-pattern 1: God Component
**Symptoms**: Single component is too large and has too many responsibilities

**Identification**:
- Use Read to check component files, code lines > 300
- Component contains extensive state management logic
- Mixes business logic, UI logic, and data fetching

**Recommendations**:
- Split into multiple sub-components
- Extract custom Hooks or Composables
- Use container/presentational component pattern

### Anti-pattern 2: Circular Dependencies
**Symptoms**: Circular references between modules

**Identification**:
- Use Grep to search import statements
- Build tool warnings
- Runtime undefined errors

**Recommendations**:
- Redesign module dependency relationships
- Introduce intermediate abstraction layer
- Use dependency injection

### Anti-pattern 3: Lack of Abstraction Layer
**Symptoms**: Business code directly calls third-party libraries

**Identification**:
- Use Grep to search direct references to third-party libraries
- Check if API wrapper layer exists
- Evaluate difficulty of library replacement

**Recommendations**:
- Create adapter layer
- Define unified interfaces
- Use Facade pattern

### Anti-pattern 4: Over-engineering
**Symptoms**: Simple requirements using complex architectural patterns

**Identification**:
- Excessive abstraction layers and design patterns
- Overly complex configuration files
- High code comprehension cost

**Recommendations**:
- Follow YAGNI principle
- Simplify unnecessary abstractions
- Prioritize code readability

### Anti-pattern 5: Type System Abuse
**Symptoms**: Overly complex type definitions or excessive `any` usage

**Identification**:
- Use Grep to search `any` keyword
- Check type definition complexity
- Evaluate type inference effectiveness

**Recommendations**:
- Use types moderately, avoid over-design
- Reduce `any` usage, use `unknown` or specific types
- Leverage type inference to simplify code

## Usage Examples

### Example 1: Comprehensive Architecture Review
```
Please review the frontend architecture of the current project, focusing on performance and maintainability.
```

### Example 2: Targeted Analysis
```
Analyze the project's code splitting strategy and identify optimization opportunities.
```

### Example 3: Comparative Analysis
```
Compare the architectural design between src/old-feature and src/new-feature, and summarize best practices.
```

### Example 4: Migration Recommendations
```
We plan to migrate from Webpack to Vite. Please analyze architectural adjustments we need to consider.
```

## Output Standards

Each review should include:

1. **Specific Code Locations** - Use `file_path:line_number` format
2. **Impact Assessment** - High/Medium/Low
3. **Improvement Recommendations** - Actionable specific approaches
4. **Priority Ranking** - P0/P1/P2
5. **Reference Documentation** - Links to detailed best practice documents

## Important Notes

- This Skill is for analysis and recommendations only; it will not modify any code
- Review results are for reference; final decisions should consider team's actual situation
- Provides framework-specific best practices for different tech stacks
- Focus on architectural evolution, avoid over-design
- Consider team's technical capabilities and learning costs
