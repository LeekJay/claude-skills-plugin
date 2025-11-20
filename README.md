# Development Best Practices Plugin for Claude Code

[中文文档](./README_CN.md)

A comprehensive collection of development best practices as agent skills for Claude Code, including Git conventions, code quality standards, documentation guidelines, and frontend architecture design.

## 📦 What's Included

This plugin provides 7 essential skills that help Claude understand and enforce development best practices:

### 1. **Git Commit Conventions** (`git-commit-conventions`)
Standardize Git commit message format using Conventional Commits with Chinese subjects.

**Use when:**
- Creating Git commits
- Checking commit message format
- Preparing pull requests

**Key Features:**
- Conventional Commits format with Chinese subjects
- Strictly prohibits AI identifiers in commit messages
- Supports automated tooling integration

### 2. **Pull Request Guidelines** (`pull-request-guidelines`)
Standardize Pull Request workflow and content requirements.

**Use when:**
- Creating PRs
- Preparing for code review
- Validating PR content

**Key Features:**
- Chinese summary requirements
- Related issue links
- Verification steps and impact assessment

### 3. **Code Quality Standards** (`code-quality-standards`)
Code quality check standards and workflow.

**Use when:**
- Completing feature implementation
- Fixing bugs
- Before creating commits/PRs

**Key Features:**
- Execution order for formatting, linting, type checking, and testing
- Strict TypeScript type error fixing rules
- Exception scenarios handling

### 4. **Language Preferences** (`language-preferences`)
Language and interaction standards for communication.

**Use when:**
- Communicating with users
- Explaining code
- Reporting errors
- Providing documentation

**Key Features:**
- Chinese/English usage scenarios
- Technical terms in original language
- Requirement clarification process

### 5. **Documentation Guidelines** (`documentation-guidelines`)
Documentation creation and update standards.

**Use when:**
- Considering creating documentation
- Code changes involve documentation

**Key Features:**
- Don't proactively create docs unless critical
- Update relevant docs only after code changes pass checks

### 6. **Git Operations Safety** (`git-operations-safety`)
Git operation safety standards.

**Use when:**
- Before executing any Git operation
- Especially before destructive operations (reset, force push, clean)

**Key Features:**
- Must run git status before operations
- Destructive operations require explicit user confirmation
- Special restrictions on force push to main/master branches

### 7. **Frontend Architecture Design** (`frontend-architecture-design`)
Review and redesign frontend project architecture.

**Use when:**
- Architecture review needed
- Code structure analysis
- Architecture redesign
- Performance optimization
- Architectural improvements

**Key Features:**
- Analyzes 5 dimensions: Performance, Maintainability, Scalability, Engineering, Type Safety
- Supports React, Vue, and other tech stacks
- Provides detailed review reports with actionable recommendations

## 🚀 Installation

### Prerequisites
- Claude Code installed and running
- Git installed on your system

### Quick Install

1. **Add this marketplace to Claude Code:**
```shell
/plugin marketplace add LeekJay/claude-skills-plugin
```

2. **Install the plugin:**
```shell
/plugin install development-best-practices@LeekJay
```

3. **Restart Claude Code** to activate the skills

4. **Verify installation:**
The skills will automatically be available. Check with:
```shell
claude --help
```

### Alternative: Local Installation

1. **Clone this repository:**
```bash
git clone https://github.com/LeekJay/claude-skills-plugin.git
cd claude-skills-plugin
```

2. **Add as local marketplace:**
```shell
/plugin marketplace add ./claude-skills-plugin
```

3. **Install the plugin:**
```shell
/plugin install development-best-practices@local
```

## 📖 Usage

Once installed, Claude will automatically use these skills when appropriate contexts are detected. You don't need to manually invoke them - Claude will recognize when to apply each skill based on your requests.

### Example Scenarios

**When you commit code:**
```bash
# Claude will automatically apply git-commit-conventions
git add .
# Ask Claude to create a commit
```

**When you create a PR:**
```bash
# Claude will apply pull-request-guidelines
# Ask Claude to create a pull request
```

**When you ask for architecture review:**
```
"Please review the frontend architecture of my project"
# Claude will use frontend-architecture-design skill
```

**Before any git operations:**
```bash
# Claude will apply git-operations-safety automatically
# before executing destructive git commands
```

## 🏗️ Plugin Structure

```
claude-skills-plugin/
├── .claude-plugin/
│   ├── plugin.json              # Plugin metadata
│   └── marketplace.json         # Marketplace configuration
├── skills/                       # All agent skills
│   ├── git-commit-conventions/
│   │   └── SKILL.md
│   ├── pull-request-guidelines/
│   │   └── SKILL.md
│   ├── code-quality-standards/
│   │   └── SKILL.md
│   ├── language-preferences/
│   │   └── SKILL.md
│   ├── documentation-guidelines/
│   │   └── SKILL.md
│   ├── git-operations-safety/
│   │   └── SKILL.md
│   └── frontend-architecture-design/
│       ├── SKILL.md
│       ├── PERFORMANCE.md
│       ├── MAINTAINABILITY.md
│       ├── SCALABILITY.md
│       ├── ENGINEERING.md
│       └── TYPESCRIPT.md
├── README.md                     # This file
├── README_CN.md                  # Chinese documentation
└── LICENSE                       # MIT License
```

## 🔧 Customization

You can customize these skills for your team:

1. Fork this repository
2. Modify the skill files in the `skills/` directory to match your team's standards
3. Update the `plugin.json` with your information
4. Share with your team via your own marketplace

## 🤝 Contributing

Contributions are welcome! Here's how you can help:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-skill`)
3. Commit your changes (following our git-commit-conventions!)
4. Push to the branch (`git push origin feature/amazing-skill`)
5. Open a Pull Request (following our pull-request-guidelines!)

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Built for [Claude Code](https://claude.com/claude-code)
- Inspired by industry best practices and team collaboration needs
- Thanks to all contributors and users

## 📞 Support

- **Issues**: [GitHub Issues](https://github.com/LeekJay/claude-skills-plugin/issues)
- **Discussions**: [GitHub Discussions](https://github.com/LeekJay/claude-skills-plugin/discussions)
- **Documentation**: [Claude Code Plugins Guide](https://docs.anthropic.com/claude/docs/plugins)

## 🔄 Version History

### v1.0.0 (Initial Release)
- Git Commit Conventions skill
- Pull Request Guidelines skill
- Code Quality Standards skill
- Language Preferences skill
- Documentation Guidelines skill
- Git Operations Safety skill
- Frontend Architecture Design skill

---

**Made with ❤️ for better development practices**
