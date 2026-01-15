# reShark Quickstart Guide

**Welcome to reShark v2.0.0** - A human-in-the-loop reverse engineering framework for unknown protocols using AI agents.

---

## Getting Started (For Private Use)

If you're analyzing proprietary or sensitive protocols, follow this approach to keep your knowledge private while using reShark:

### 1. Download the Framework (Without Git History)

**Option A: Download ZIP (Recommended for Private Work)**
```bash
# Download the latest release as ZIP from GitHub
# Go to: https://github.com/kmaneesh/reShark/archive/refs/heads/main.zip

# Extract to your workspace
unzip reShark-main.zip
mv reShark-main my-protocol-analysis
cd my-protocol-analysis

# Remove any git references
rm -rf .git
```

**Option B: Use GitHub's "Use this template" Feature**
```bash
# 1. Go to: https://github.com/kmaneesh/reShark
# 2. Click "Use this template" → "Create a new repository"
# 3. Choose "Private" for your repository visibility
# 4. Clone your new private repository
git clone https://github.com/YOUR_USERNAME/your-private-repo.git
cd your-private-repo
```

### 2. Initialize Your Private Repository

```bash
# Initialize a new git repository for your work
git init

# Create your first commit with the clean framework
git add .
git commit -m "Initial reShark framework setup"

# Add your private remote (GitHub/GitLab/Bitbucket)
git remote add origin https://github.com/YOUR_USERNAME/your-private-analysis.git
git push -u origin main
```

### 3. Add Your Sensitive Knowledge

Now you can safely add your proprietary analysis data:

```bash
# Create your private Books workspace
mkdir -p books/notebook/sessions
mkdir -p books/rulebook/grammars
mkdir -p books/cookbook/methods

# Add your proprietary PCAP files
mkdir -p artifacts/pcaps
# Copy your PCAP files here

# Add custom skills for your domain
mkdir -p books/skills/custom
# Create domain-specific methodology files
cat > books/skills/custom/my-protocol-analysis.md << 'EOF'
# My Protocol Analysis Methodology

## Domain Context
[Your proprietary protocol information]

## Analysis Approach
[Your custom methodology]
EOF

# Add custom tools or integrations
mkdir -p tools/custom
# Add any custom tool wrappers you need

# Commit your private work
git add .
git commit -m "Add proprietary analysis artifacts and custom skills"
git push
```

### 4. Keep Your Sensitive Data Isolated

The `.gitignore` file is pre-configured to exclude sensitive directories:

```bash
# Already gitignored by default:
books/notebook/        # Your session observations (ephemeral)
books/rulebook/        # Your validated grammars (sensitive)
artifacts/            # Your PCAP files and binaries (proprietary)
*.pcap                # All PCAP files
*.pcapng              # All PCAP-NG files
.env                  # Your environment secrets

# Add additional patterns to .gitignore if needed:
echo "my-custom-sensitive-dir/" >> .gitignore
```

**⚠️ Important**: Review `.gitignore` before every push to ensure sensitive data isn't accidentally committed.

### 5. Update Framework Without Losing Your Work

When reShark releases updates, you can safely merge them:

```bash
# Add the upstream reShark repository as a remote
git remote add upstream https://github.com/kmaneesh/reShark.git

# Fetch the latest changes
git fetch upstream

# Review what changed (optional)
git log upstream/main --oneline -n 10

# Merge framework updates into your private repo
git merge upstream/main --no-commit

# Review the merge, ensure no sensitive data conflicts
git status

# If conflicts occur, resolve them carefully
# Make sure your private books/, artifacts/ remain untouched

# Commit the merge
git commit -m "Merge upstream reShark framework updates"
git push
```

---

## Contributing Back (For Public Improvements)

If you've developed improvements to the core framework (not your private analysis), you can contribute back:

### 1. Create a Clean Development Environment

```bash
# Clone the official repository
git clone https://github.com/kmaneesh/reShark.git reShark-dev
cd reShark-dev

# Create a feature branch
git checkout -b feature/your-improvement-name
```

### 2. Develop Your Feature

Make your improvements to:
- **Core agents** (`reshark/agents/`)
- **Books system** (`reshark/books/`)
- **Tool wrappers** (`reshark/tools/`)
- **Skills library** (`books/skills/`) - Only generic, non-sensitive methodology files
- **Documentation** (`docs/`, `README.md`)
- **Tests** (`tests/`)

**⚠️ Never include**:
- Your proprietary PCAP files
- Your private grammars or Rulebook entries
- Your sensitive Notebook sessions
- Your custom domain-specific skills (unless they're generic enough)
- Any configuration with secrets or endpoints

### 3. Test Your Changes

```bash
# Run the test suite
poetry install
poetry run pytest tests/ -v

# Run constitutional compliance tests
poetry run pytest tests/integration/test_constitutional_compliance.py -v

# Verify code quality
poetry run black reshark/ tests/
poetry run pylint reshark/
```

### 4. Submit a Pull Request

```bash
# Commit your changes
git add .
git commit -m "feat: Add improved entropy calculation for PatternInterpreter"

# Push to your fork
git push origin feature/your-improvement-name

# Go to GitHub and create a Pull Request:
# 1. Go to: https://github.com/kmaneesh/reShark
# 2. Click "Pull requests" → "New pull request"
# 3. Select your branch
# 4. Fill in the PR template:
#    - Description of changes
#    - Why this improves the framework
#    - Testing performed
#    - Any breaking changes
```

### 5. PR Review Guidelines

Your PR should:
- ✅ Include tests for new functionality
- ✅ Update relevant documentation
- ✅ Follow the existing code style (Black, Pylint)
- ✅ Pass all CI checks
- ✅ Focus on framework improvements (not domain-specific use cases)
- ✅ Be free of any proprietary or sensitive information

Example PRs we'd love to see:
- New tool wrappers (e.g., Wireshark plugins, binary analysis tools)
- Improved agent coordination patterns
- Enhanced skills library with generic methodology files
- Better error handling in ToolsRegistry
- Performance optimizations for large PCAP processing
- Additional test coverage
- Documentation improvements
- Bug fixes

---

## Development Setup (Phase 1 HITL Mode)

### Prerequisites

- **Docker Desktop** (for Dev Container)
- **VS Code** with Remote-Containers extension
- **Git** (for version control)
- **8GB RAM minimum** (16GB recommended)
- **10GB disk space** (for Docker images + artifacts)

### Quick Setup

1. **Open in Dev Container**:
   ```bash
   # Open VS Code in the project directory
   code .
   
   # VS Code will prompt: "Reopen in Container"
   # Or use Command Palette: "Remote-Containers: Reopen in Container"
   ```

2. **Verify Ollama**:
   ```bash
   # Inside the container, check Ollama is running
   ollama list
   
   # Pull a recommended model (if not already available)
   ollama pull mistral:7b-instruct
   ```

3. **Install Dependencies**:
   ```bash
   poetry install
   
   # Verify installation
   poetry run pytest tests/agents/test_base.py -v
   ```

4. **Configure Cline Extension**:
   - Install Cline extension in VS Code
   - Configure to use Ollama endpoint: `http://localhost:11434`
   - Select model: `mistral:7b-instruct` or your preferred model

5. **First Analysis Session**:
   ```bash
   # Start an interactive session with Cline
   # Cline prompt: "Analyze sample.pcap using DataInterpreter"
   
   # Cline will:
   # 1. Read methodology from books/skills/protocol/data-extraction.md
   # 2. Invoke DataInterpreter agent
   # 3. Use ToolsRegistry to run tshark/hexdump
   # 4. Write observations to books/notebook/sessions/<session_id>/
   # 5. Present findings for your review
   ```

6. **View Results**:
   ```bash
   # Check session observations
   ls books/notebook/sessions/
   cat books/notebook/sessions/<session_id>/observations.json
   
   # Check if any grammars were validated
   ls books/rulebook/grammars/
   ```

---

## Security Best Practices

### Data Isolation

1. **Never commit sensitive data**:
   ```bash
   # Before every push, verify:
   git status
   git diff
   
   # Ensure no PCAP files, grammars, or sensitive configs
   ```

2. **Use environment variables for secrets**:
   ```bash
   # Create .env file (gitignored)
   cat > .env << 'EOF'
   OLLAMA_ENDPOINT=http://localhost:11434
   CUSTOM_API_KEY=your-secret-key
   EOF
   
   # Load in your scripts
   # Python: from dotenv import load_dotenv; load_dotenv()
   ```

3. **Separate repositories**:
   - **Private repo**: Your analysis work + sensitive data
   - **Public fork**: Only for contributing framework improvements

4. **Review merge conflicts carefully**:
   ```bash
   # When merging upstream updates
   git merge upstream/main
   
   # If conflicts in books/ or artifacts/
   # Always choose to keep YOUR version (the sensitive one)
   git checkout --ours books/
   git checkout --ours artifacts/
   ```

### Access Control

1. **Keep your analysis repo private** on GitHub/GitLab
2. **Use branch protection** for important branches
3. **Enable 2FA** on your Git hosting account
4. **Rotate credentials** if you suspect exposure

---

## Troubleshooting

### Ollama Not Responding

```bash
# Check if Ollama is running
curl http://localhost:11434/api/version

# Restart Ollama service (inside container)
docker restart ollama

# Check logs
docker logs ollama
```

### Agent Execution Errors

```bash
# Enable verbose logging
export RESHARK_LOG_LEVEL=DEBUG

# Run agent with detailed output
poetry run python -m reshark.cli analyze --verbose sample.pcap
```

### Dev Container Won't Start

```bash
# Rebuild container without cache
# Command Palette: "Remote-Containers: Rebuild Container Without Cache"

# Or manually:
docker-compose -f .devcontainer/docker-compose.yml down
docker-compose -f .devcontainer/docker-compose.yml build --no-cache
```

### Skills Not Loading

```bash
# Verify skills directory structure
ls -R books/skills/

# Check skills are readable
cat books/skills/protocol/data-extraction.md

# Validate markdown syntax
poetry run pytest tests/books/test_skills.py -v
```

---

## Next Steps

1. **Read the documentation**: [docs/architecture.md](docs/architecture.md)
2. **Explore skills library**: [books/skills/README.md](books/skills/README.md)
3. **Review example sessions**: [books/notebook/README.md](books/notebook/README.md)
4. **Join the community**: [GitHub Discussions](https://github.com/kmaneesh/reShark/discussions)
5. **Report issues**: [GitHub Issues](https://github.com/kmaneesh/reShark/issues)

---

## License

reShark is licensed under [LICENSE](LICENSE). Your private analysis work remains your property. Contributions back to the framework are under the same license.

## Support

- **Documentation**: [docs/](docs/)
- **Issues**: [GitHub Issues](https://github.com/kmaneesh/reShark/issues)
- **Discussions**: [GitHub Discussions](https://github.com/kmaneesh/reShark/discussions)
- **Email**: maintainer@reshark.dev

---

**Happy Protocol Reverse Engineering! 🦈**
