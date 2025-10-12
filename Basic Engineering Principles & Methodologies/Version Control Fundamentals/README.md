# Version Control Fundamentals

This directory covers the essential concepts and practices of version control systems, with a focus on Git, the most widely used version control system.

## Overview

Version control is a crucial skill for software developers, enabling collaborative development, code history tracking, and project management. This guide focuses on fundamental concepts and practical usage.

## Key Concepts

1. **Basic Version Control Operations**
   - Initializing repositories
   - Tracking changes
   - Committing code
   - Branching and merging
   - Remote repositories

2. **Collaboration**
   - Push and pull
   - Code review
   - Conflict resolution
   - Branching strategies
   - Pull requests

## Example Commands and Workflows

### Basic Git Operations

```bash
# Initialize a new repository
git init

# Check repository status
git status

# Stage changes
git add file.txt
git add .  # Stage all changes

# Commit changes
git commit -m "Add new feature"

# View commit history
git log
git log --oneline
git log --graph --oneline --all

# View changes
git diff
git diff --staged
```

### Working with Branches

```bash
# Create and switch to a new branch
git checkout -b feature/new-feature

# List branches
git branch

# Switch branches
git checkout main
git switch main  # Git 2.23+

# Merge branches
git merge feature/new-feature

# Delete branch
git branch -d feature/new-feature
```

### Remote Repository Operations

```bash
# Add remote repository
git remote add origin https://github.com/username/repo.git

# Clone repository
git clone https://github.com/username/repo.git

# Push changes
git push origin main

# Pull changes
git pull origin main

# Fetch changes
git fetch origin
```

### Advanced Operations

```bash
# Interactive rebase
git rebase -i HEAD~3

# Cherry-pick commits
git cherry-pick commit-hash

# Stash changes
git stash
git stash pop
git stash list

# Reset changes
git reset HEAD~1
git reset --hard HEAD~1

# Amend last commit
git commit --amend
```

## Project Structure Example

```
project/
├── .git/                   # Git repository data
├── src/                    # Source code
│   ├── main.js
│   └── utils.js
├── tests/                  # Test files
│   └── main.test.js
├── .gitignore             # Git ignore file
├── README.md              # Project documentation
└── package.json           # Project configuration
```

### Example .gitignore File

```gitignore
# Dependencies
node_modules/
vendor/

# Build output
dist/
build/
*.min.js

# IDE files
.vscode/
.idea/
*.sublime-workspace

# Environment files
.env
.env.local

# Logs
*.log
npm-debug.log*

# Operating System
.DS_Store
Thumbs.db

# Coverage reports
coverage/
.nyc_output/

# Temporary files
*.tmp
*.swp
```

## Common Workflows

### Feature Branch Workflow

```bash
# Start a new feature
git checkout -b feature/user-authentication main

# Make changes and commit
git add .
git commit -m "Add user authentication"

# Push feature branch
git push origin feature/user-authentication

# Create pull request (on GitHub/GitLab)

# After review and approval, merge to main
git checkout main
git pull origin main
git merge feature/user-authentication
git push origin main
```

### Hotfix Workflow

```bash
# Create hotfix branch from main
git checkout -b hotfix/security-fix main

# Make fixes and commit
git add .
git commit -m "Fix security vulnerability"

# Push hotfix
git push origin hotfix/security-fix

# Merge to main
git checkout main
git merge hotfix/security-fix
git push origin main

# Merge to development
git checkout development
git merge hotfix/security-fix
git push origin development
```

### Release Workflow

```bash
# Create release branch
git checkout -b release/1.0.0 development

# Make release preparations
git add .
git commit -m "Prepare for release 1.0.0"

# Tag release
git tag -a v1.0.0 -m "Version 1.0.0"

# Merge to main
git checkout main
git merge release/1.0.0
git push origin main
git push --tags

# Merge back to development
git checkout development
git merge release/1.0.0
git push origin development
```

## Best Practices

1. **Commit Messages**
   ```
   # Good commit message format
   feat: add user authentication system

   - Implement JWT-based authentication
   - Add login and registration endpoints
   - Include password hashing
   ```

2. **Branching Strategy**
   ```
   main          # Production-ready code
   ├── develop   # Development branch
   ├── feature/* # Feature branches
   ├── bugfix/*  # Bug fix branches
   ├── hotfix/*  # Emergency fixes
   └── release/* # Release branches
   ```

3. **Code Review Checklist**
   - Code follows style guidelines
   - Tests are included
   - Documentation is updated
   - No sensitive information
   - Performance considerations
   - Security implications

4. **Conflict Resolution**
   ```bash
   # When conflicts occur
   git pull origin main
   # Fix conflicts in files
   git add .
   git commit -m "Resolve merge conflicts"
   git push origin feature-branch
   ```

## Tools and Extensions

1. **GUI Clients**
   - GitKraken
   - SourceTree
   - GitHub Desktop
   - Tower

2. **IDE Integration**
   - VS Code Git
   - IntelliJ IDEA Git
   - Eclipse EGit
   - Xcode Source Control

3. **Command Line Tools**
   - git-flow
   - hub
   - gh
   - lazygit

## Further Learning

- Advanced Git Commands
- Git Workflows
- Continuous Integration
- Git Hooks
- Repository Management
- Team Collaboration

---

*Part of the Basic Engineering Principles & Methodologies collection*
