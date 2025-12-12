# Contributing to Banana Slides

Thank you for your interest in contributing to Banana Slides! This guide will help you get started.

## 🤝 Ways to Contribute

- **Report bugs**: Found a bug? [Open an issue](https://github.com/Anionex/banana-slides/issues)
- **Suggest features**: Have an idea? Share it in discussions
- **Improve documentation**: Fix typos, clarify instructions, add examples
- **Write code**: Fix bugs, add features, improve performance
- **Share usage examples**: Show how you use Banana Slides

## 🐛 Reporting Bugs

### Before Submitting

1. **Search existing issues** to avoid duplicates
2. **Try the latest version** to see if it's already fixed
3. **Gather information**:
   - Steps to reproduce
   - Expected vs actual behavior
   - Error messages and logs
   - Environment details (OS, versions, etc.)

### Bug Report Template

```markdown
## Description
Brief description of the bug

## Steps to Reproduce
1. Go to...
2. Click on...
3. See error...

## Expected Behavior
What should happen

## Actual Behavior
What actually happens

## Environment
- OS: [e.g., Windows 10, macOS 13, Ubuntu 22.04]
- Installation: [Docker / Manual]
- Python Version: [e.g., 3.11]
- Node Version: [e.g., 18.x]
- Browser: [e.g., Chrome 120]

## Logs
```
Paste relevant logs here
```

## Screenshots
[If applicable]
```

## 💡 Suggesting Features

### Feature Request Template

```markdown
## Feature Description
Clear description of the proposed feature

## Use Case
Why is this feature needed? Who will benefit?

## Proposed Solution
How should it work?

## Alternatives Considered
Other approaches you've thought about

## Additional Context
Any other relevant information
```

## 🔧 Development Setup

### Prerequisites

- Python 3.10+
- Node.js 16+
- Git
- Code editor (VSCode recommended)

### Setup Steps

1. **Fork the repository**:
   - Click "Fork" on GitHub
   - Clone your fork:
     ```bash
     git clone https://github.com/YOUR_USERNAME/banana-slides.git
     cd banana-slides
     ```

2. **Add upstream remote**:
   ```bash
   git remote add upstream https://github.com/Anionex/banana-slides.git
   ```

3. **Install dependencies**:
   ```bash
   # Backend
   uv sync
   
   # Frontend
   cd frontend
   npm install
   ```

4. **Configure environment**:
   ```bash
   cp .env.example .env
   # Edit .env with your API keys
   ```

5. **Start development servers**:
   ```bash
   # Terminal 1: Backend
   cd backend
   uv run python app.py
   
   # Terminal 2: Frontend
   cd frontend
   npm run dev
   ```

## 📝 Code Style

### Python (Backend)

Follow PEP 8 style guide:

```python
# Good
def generate_outline(project_context: ProjectContext) -> dict:
    """Generate outline from project context.
    
    Args:
        project_context: Project information
        
    Returns:
        Outline dictionary
    """
    prompt = get_outline_generation_prompt(project_context)
    response = text_provider.generate_text(prompt)
    return json.loads(response)

# Bad
def generate_outline(project_context):
    prompt = get_outline_generation_prompt(project_context)
    response=text_provider.generate_text(prompt)
    return json.loads(response)
```

**Key points**:
- Type hints for function parameters and returns
- Docstrings for functions and classes
- 4 spaces for indentation
- Meaningful variable names
- Line length ≤ 100 characters

### TypeScript (Frontend)

Follow TypeScript best practices:

```typescript
// Good
interface ProjectData {
  projectId: string;
  title: string;
  pages: Page[];
}

const createProject = async (data: CreateProjectRequest): Promise<Project> => {
  const response = await apiClient.post('/api/projects', data);
  return response.data;
};

// Bad
const createProject = async (data) => {
  const response = await apiClient.post('/api/projects', data);
  return response.data;
}
```

**Key points**:
- Explicit type definitions
- Interface over type for object types
- Async/await over promises
- Arrow functions for consistency
- 2 spaces for indentation

### Formatting

Use automated formatters:

```bash
# Python: Black (future)
pip install black
black backend/

# TypeScript: Prettier (future)
npm install -D prettier
npx prettier --write frontend/src/
```

## 🌿 Branch Strategy

### Branch Naming

- `feature/description` - New features
- `fix/description` - Bug fixes
- `docs/description` - Documentation
- `refactor/description` - Code refactoring
- `test/description` - Tests

Examples:
- `feature/add-slide-animation`
- `fix/image-generation-timeout`
- `docs/update-api-reference`

### Workflow

1. **Create a branch**:
   ```bash
   git checkout -b feature/my-new-feature
   ```

2. **Make changes**:
   - Write code
   - Test thoroughly
   - Add/update documentation

3. **Commit changes**:
   ```bash
   git add .
   git commit -m "Add feature: description"
   ```

4. **Keep branch updated**:
   ```bash
   git fetch upstream
   git rebase upstream/main
   ```

5. **Push to your fork**:
   ```bash
   git push origin feature/my-new-feature
   ```

6. **Create Pull Request**:
   - Go to GitHub
   - Click "New Pull Request"
   - Fill in description

## 📤 Pull Request Process

### PR Title Format

```
<type>: <description>

Types:
- feat: New feature
- fix: Bug fix
- docs: Documentation
- refactor: Code refactoring
- test: Tests
- chore: Maintenance

Examples:
- feat: Add slide animation support
- fix: Resolve image generation timeout
- docs: Update API reference
```

### PR Description Template

```markdown
## Description
What does this PR do?

## Related Issue
Closes #123

## Changes Made
- Change 1
- Change 2
- Change 3

## Testing
How was this tested?

## Screenshots
[If applicable]

## Checklist
- [ ] Code follows style guidelines
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No breaking changes (or documented)
```

### Review Process

1. **Automated checks** run (linting, tests)
2. **Code review** by maintainers
3. **Address feedback** if requested
4. **Approval** and merge

## 🧪 Testing

### Backend Tests

```bash
cd backend
uv run pytest tests/
```

### Frontend Tests

```bash
cd frontend
npm test
```

### Manual Testing

1. Test the feature/fix in a real environment
2. Check edge cases
3. Verify no regressions
4. Test on different browsers (frontend)

## 📚 Documentation

When adding features or making changes:

1. **Update relevant docs** in `/docs`
2. **Add code comments** for complex logic
3. **Update API reference** if adding endpoints
4. **Add examples** where helpful

## 🎯 Best Practices

### General

- **Small PRs**: Easier to review and merge
- **One feature per PR**: Keep changes focused
- **Write tests**: Ensure reliability
- **Update docs**: Keep documentation current

### Code Quality

- **DRY**: Don't Repeat Yourself
- **KISS**: Keep It Simple, Stupid
- **YAGNI**: You Aren't Gonna Need It
- **Clean code**: Self-documenting when possible

### Commit Messages

```
# Good
feat: Add natural language slide editing
fix: Resolve CORS issue in Docker setup
docs: Update installation guide with troubleshooting

# Bad
update stuff
fix bug
changes
```

## 🚫 What Not to Do

- **Don't commit secrets**: API keys, passwords, etc.
- **Don't commit dependencies**: node_modules/, venv/, etc.
- **Don't commit generated files**: Build artifacts, .pyc, etc.
- **Don't make unrelated changes**: Stick to the PR scope
- **Don't ignore feedback**: Address review comments

## 📦 Adding Dependencies

### Backend (Python)

```bash
# Add to pyproject.toml
[project]
dependencies = [
    "new-package>=1.0.0",
]

# Install
uv sync
```

### Frontend (Node)

```bash
cd frontend
npm install new-package
```

**Note**: Justify new dependencies in PR description

## 🏗️ Architecture Decisions

For significant changes:

1. **Open an issue** to discuss
2. **Explain the problem** you're solving
3. **Propose solution** with pros/cons
4. **Get feedback** before implementing

## 🎓 Learning Resources

- [Python Best Practices](https://docs.python-guide.org/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [React Documentation](https://react.dev/)
- [Flask Documentation](https://flask.palletsprojects.com/)

## 💬 Community

- **GitHub Discussions**: Ask questions, share ideas
- **Issues**: Bug reports and feature requests
- **Pull Requests**: Code contributions

## 🙏 Recognition

Contributors are recognized in:
- GitHub contributors page
- Release notes
- Special thanks in README (for significant contributions)

## 📄 License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

## Quick Start Checklist

- [ ] Fork and clone repository
- [ ] Set up development environment
- [ ] Create feature branch
- [ ] Make changes and test
- [ ] Update documentation
- [ ] Commit with clear messages
- [ ] Push to your fork
- [ ] Create Pull Request
- [ ] Address review feedback

---

**Thank you for contributing to Banana Slides!** 🎉

Every contribution, no matter how small, helps make this project better.

Questions? Open an issue or start a discussion!
