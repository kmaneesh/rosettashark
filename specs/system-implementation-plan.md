# reShark Complete Implementation Plan

**Version**: 2.0.0  
**Date**: 2026-01-15  
**Status**: Draft  
**Scope**: Full Project (Phase 1 + Phase 2)  
**Governed By**: reShark Constitution v2.0.0

---

## 1. Executive Summary

This plan details the complete implementation roadmap for reShark v2.0.0 - a general-purpose reverse engineering framework with dual operating modes:

- **Phase 1** (Weeks 1-8): HITL Foundation - Dev Container + Cline + Local LLM
- **Phase 2** (Weeks 9-16): Autonomous Mode - OpenHands + Patternbook + Parallel Processing

**Total Duration Estimate**: 16 weeks (4 months, 1 developer)  
**Final Success Metric**: Framework can analyze diverse data formats (PCAP, binaries, firmware, file formats) in both HITL and Autonomous modes with shared knowledge base

---

## 2. Project Objectives

### 2.1 Phase 1 Goals (HITL Foundation)

1. **Establish HITL Environment**: Dev Container + Cline + Ollama Local LLM operational
2. **Implement Books System**: Notebook (sessions), Rulebook (grammars), Cookbook (methods), Skills (markdown procedures)
3. **Build Core Agents**: Interpreters, Orchestration, Validation, Scopes as Python modules
4. **Create Markdown Skills Library**: Documented analysis methodologies for LLM guidance
5. **Develop Tool Wrappers**: tshark, binutils, scapy, hexdump, entropy calculators
6. **Validate Knowledge Capture**: Human + Cline perform analysis, document findings in Books

### 2.2 Phase 2 Goals (Autonomous Mode)

1. **OpenHands Integration**: Autonomous execution environment with Local LLM
2. **Patternbook Implementation**: Qdrant vector database for cross-artifact pattern learning
3. **Skills-Driven Autonomy**: Agents read markdown skills to execute analysis procedures
4. **Multi-Artifact Analysis**: Support concurrent analysis of 5+ different artifacts
5. **Bidirectional Learning**: HITL discoveries → Skills → Autonomous application
6. **Performance Optimization**: Scale to production workloads

### 2.3 Overall Success Criteria

- [ ] Complete end-to-end analysis from unknown artifact to documented grammar/structure
- [ ] Constitutional compliance verified across all components
- [ ] At least 3 diverse artifacts analyzed: 1 PCAP protocol, 1 binary format, 1 file format (1 in Phase 1, 2+ in Phase 2)
- [ ] Skills library contains 12+ documented methodologies across categories (protocol, pattern, binary, grammar)
- [ ] Autonomous mode successfully applies skills learned in HITL mode
- [ ] System handles 5 concurrent artifact analyses (Phase 2)
- [ ] Documentation complete for both phases
- [ ] Test coverage >80% for all modules
- [ ] Open-source release ready (infrastructure only, per Constitution Section 7)

---

## 3. Phase Breakdown Overview

### Phase 1: HITL Foundation (Weeks 1-8)

| Week | Focus Area | Key Deliverable |
|------|-----------|-----------------|
| 1-2 | Infrastructure | Dev Container, Ollama, Cline setup |
| 3 | Books System | Notebook, Rulebook, Cookbook, Skills structure |
| 4 | Agents - Interpreters | Data parsing and extraction agents |
| 5 | Agents - Orchestration | Workflow and task management |
| 6 | Agents - Validation | Cross-sample validation agents |
| 7 | Tools & Skills | Tool wrappers + markdown skill documentation |
| 8 | HITL Integration | Complete HITL workflow, testing, documentation |

**Phase 1 Gate Criteria**: All Phase 1 success criteria met, one complete artifact analysis (HITL mode) demonstrated with documented skills.

### Phase 2: Autonomous Mode (Weeks 9-16)

| Week | Focus Area | Key Deliverable |
|------|-----------|-----------------|
| 9-10 | OpenHands Integration | Autonomous environment, agent adaptation |
| 11 | Patternbook | Qdrant vector DB, embeddings, pattern retrieval |
| 12 | Skills Autonomy | Agents read/execute markdown skills autonomously |
| 13 | Multi-Artifact | Parallel analysis support, resource management |
| 14 | Optimization | Performance tuning, caching, efficiency |
| 15 | Bidirectional Learning | HITL → Skills → Autonomous pipeline |
| 16 | Finalization | Testing, documentation, release preparation |

**Phase 2 Completion**: System analyzes 5 concurrent artifacts autonomously using skills learned in HITL mode, with >85% accuracy on validation tasks.

---

## 4. Detailed Implementation Schedule

---

## PHASE 1: HITL FOUNDATION (WEEKS 1-8)

---

### Week 1-2: HITL Infrastructure Setup

#### Objectives
- Dev Container environment operational
- Ollama Local LLM configured and running
- Cline extension integrated with VS Code
- Repository structure established
- Python environment configured

#### Tasks

**Day 1-2: Repository and Directory Structure**

1. **Create Repository Structure**
   ```bash
   mkdir reshark && cd reshark
   git init
   git branch -M main
   git remote add origin <repo-url>
   
   # Create directory structure per v2.0.0 architecture
   mkdir -p {books/{notebook/{sessions,archives},rulebook/{grammars,schemas},cookbook/{methods,examples},patternbook/{embeddings,queries}},agents/{interpreters,orchestration,validation,scopes},tools/{scripts},skills/{protocol,pattern,binary,grammar},labs,tests,docs,data/{input,output}}
   ```

2. **Initialize Git and Core Documentation**
   ```bash
   # Create .gitignore
   cat > .gitignore << 'EOF'
   __pycache__/
   *.py[cod]
   .pytest_cache/
   .coverage
   htmlcov/
   dist/
   *.egg-info/
   .env
   .venv/
   books/notebook/sessions/session-*
   data/input/*.pcap
   data/input/*.bin
   !data/input/samples/
   .DS_Store
   EOF
   
   # Copy constitutional documents
   cp /path/to/constitution-v2.md docs/
   cp /path/to/srs-v2.md docs/
   cp /path/to/sitd-v2.md docs/
   
   # Initial commit
   git add .
   git commit -m "chore: initialize reShark v2.0.0 repository"
   ```

**Day 3-4: Dev Container Configuration**

3. **Create Dev Container Setup**
   ```json
   // .devcontainer/devcontainer.json
   {
     "name": "reShark HITL Environment",
     "dockerComposeFile": "docker-compose.yml",
     "service": "reshark-hitl",
     "workspaceFolder": "/workspace",
     
     "customizations": {
       "vscode": {
         "extensions": [
           "saoudrizwan.claude-dev",  // Cline
           "ms-python.python",
           "ms-python.vscode-pylance",
           "charliermarsh.ruff",
           "ms-toolsai.jupyter",
           "tamasfe.even-better-toml"
         ],
         "settings": {
           "python.defaultInterpreterPath": "/usr/local/bin/python",
           "python.linting.enabled": true,
           "python.formatting.provider": "black",
           "terminal.integrated.defaultProfile.linux": "zsh"
         }
       }
     },
     
     "mounts": [
       "source=${localWorkspaceFolder}/books,target=/workspace/books,type=bind",
       "source=${localWorkspaceFolder}/data,target=/workspace/data,type=bind"
     ],
     
     "postCreateCommand": "poetry install && pre-commit install",
     "remoteUser": "vscode"
   }
   ```

   ```yaml
   # .devcontainer/docker-compose.yml
   version: '3.8'
   
   services:
     reshark-hitl:
       build:
         context: ..
         dockerfile: .devcontainer/Dockerfile
       volumes:
         - ..:/workspace:cached
       environment:
         - OLLAMA_HOST=http://ollama:11434
         - PYTHONUNBUFFERED=1
       networks:
         - reshark-network
       stdin_open: true
       tty: true
     
     ollama:
       image: ollama/ollama:latest
       volumes:
         - ollama-data:/root/.ollama
       ports:
         - "11434:11434"
       networks:
         - reshark-network
   
   networks:
     reshark-network:
       driver: bridge
   
   volumes:
     ollama-data:
   ```

   ```dockerfile
   # .devcontainer/Dockerfile
   FROM mcr.microsoft.com/devcontainers/python:3.12-bullseye
   
   # Install system dependencies
   RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
       && apt-get -y install --no-install-recommends \
       tshark \
       wireshark-common \
       tcpdump \
       binutils \
       hexdump \
       file \
       zsh \
       && apt-get clean \
       && rm -rf /var/lib/apt/lists/*
   
   # Install Poetry
   RUN pip install --no-cache-dir poetry==1.7.1
   
   # Configure zsh
   RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v1.1.5/zsh-in-docker.sh)"
   
   # Set working directory
   WORKDIR /workspace
   
   # Copy pyproject.toml for dependency caching
   COPY pyproject.toml poetry.lock* ./
   
   RUN poetry config virtualenvs.create false \
       && poetry install --no-interaction --no-ansi || true
   ```

4. **Python Environment Setup**
   ```bash
   # Initialize Poetry
   poetry init --name reshark --python "^3.12"
   
   # Add core dependencies
   poetry add \
     numpy \
     scipy \
     pandas \
     scapy \
     pyshark \
     pyyaml \
     pydantic \
     rich \
     typer \
     jinja2
   
   # Add dev dependencies
   poetry add --group dev \
     pytest \
     pytest-cov \
     pytest-asyncio \
     black \
     isort \
     mypy \
     ruff \
     pre-commit
   
   # Lock and install
   poetry lock
   poetry install
   ```

   ```toml
   # Add to pyproject.toml
   [tool.black]
   line-length = 88
   target-version = ['py312']
   
   [tool.isort]
   profile = "black"
   
   [tool.mypy]
   python_version = "3.12"
   warn_return_any = true
   warn_unused_configs = true
   disallow_untyped_defs = true
   
   [tool.pytest.ini_options]
   testpaths = ["tests"]
   python_files = ["test_*.py"]
   addopts = "-v --cov=reshark --cov-report=html"
   
   [tool.ruff]
   line-length = 88
   target-version = "py312"
   select = ["E", "F", "I", "N", "W", "UP"]
   ```

**Day 5-6: Ollama and Cline Configuration**

5. **Configure Ollama Local LLM**
   ```bash
   # From dev container, test Ollama connection
   curl http://ollama:11434/api/version
   
   # Pull recommended models
   docker exec ollama ollama pull llama3.2:3b    # Fast, good for tool use
   docker exec ollama ollama pull codellama:7b   # Code-focused
   docker exec ollama ollama pull llama3:8b      # Balanced capability
   ```

6. **Configure Cline Extension**
   ```json
   // .vscode/settings.json (created in workspace)
   {
     "claude-dev.apiProvider": "ollama",
     "claude-dev.ollamaBaseUrl": "http://ollama:11434",
     "claude-dev.ollamaModelId": "llama3.2:3b",
     "claude-dev.maxTokens": 8192,
     "claude-dev.enableContextFiles": true,
     "claude-dev.contextFiles": [
       "docs/constitution-v2.md",
       "docs/srs-v2.md",
       "skills/**/*.md"
     ]
   }
   ```

7. **Create Validation Script**
   ```python
   # scripts/validate_hitl_setup.py
   """Validate HITL environment setup."""
   
   import sys
   import subprocess
   from pathlib import Path
   import requests
   
   def check_python_version():
       """Check Python 3.12+"""
       if sys.version_info < (3, 12):
           print("❌ Python 3.12+ required")
           return False
       print(f"✓ Python {sys.version_info.major}.{sys.version_info.minor}")
       return True
   
   def check_tools():
       """Check required tools installed."""
       tools = ["tshark", "tcpdump", "hexdump", "objdump"]
       for tool in tools:
           result = subprocess.run(
               ["which", tool],
               capture_output=True
           )
           if result.returncode == 0:
               print(f"✓ {tool} installed")
           else:
               print(f"❌ {tool} not found")
               return False
       return True
   
   def check_ollama():
       """Check Ollama service accessible."""
       try:
           response = requests.get("http://ollama:11434/api/version", timeout=5)
           if response.status_code == 200:
               print(f"✓ Ollama service running: {response.json()}")
               return True
       except Exception as e:
           print(f"❌ Ollama not accessible: {e}")
           return False
       return False
   
   def check_directory_structure():
       """Check required directories exist."""
       dirs = [
           "books/notebook/sessions",
           "books/rulebook/grammars",
           "books/cookbook/methods",
           "skills/protocol",
           "skills/pattern",
           "agents/interpreters",
           "tools/scripts"
       ]
       for d in dirs:
           if Path(d).exists():
               print(f"✓ {d}/")
           else:
               print(f"❌ {d}/ missing")
               return False
       return True
   
   if __name__ == "__main__":
       print("Validating HITL Environment Setup\n")
       
       checks = [
           check_python_version(),
           check_tools(),
           check_ollama(),
           check_directory_structure()
       ]
       
       if all(checks):
           print("\n✓ All checks passed! HITL environment ready.")
           sys.exit(0)
       else:
           print("\n❌ Some checks failed. Review output above.")
           sys.exit(1)
   ```

**Day 7: CI/CD Pipeline**

8. **GitHub Actions Workflow**
   ```yaml
   # .github/workflows/tests.yml
   name: Tests
   
   on:
     push:
       branches: [ main, develop ]
     pull_request:
       branches: [ main, develop ]
   
   jobs:
     test:
       runs-on: ubuntu-latest
       
       steps:
       - uses: actions/checkout@v4
       
       - name: Set up Python
         uses: actions/setup-python@v5
         with:
           python-version: '3.12'
       
       - name: Install Poetry
         run: |
           pip install poetry==1.7.1
           poetry install
       
       - name: Run linters
         run: |
           poetry run black --check .
           poetry run isort --check .
           poetry run ruff check .
           poetry run mypy reshark/
       
       - name: Run tests
         run: |
           poetry run pytest --cov=reshark --cov-report=xml
       
       - name: Upload coverage
         uses: codecov/codecov-action@v3
         with:
           files: ./coverage.xml
   ```

#### Deliverables
- [ ] Dev Container configured and operational
- [ ] Ollama Local LLM accessible and tested
- [ ] Cline extension configured with local LLM
- [ ] Poetry environment with all dependencies
- [ ] Directory structure matches v2.0.0 architecture
- [ ] Validation script passes all checks
- [ ] CI/CD pipeline running

#### Validation Checkpoint
```bash
# From inside Dev Container
poetry run python scripts/validate_hitl_setup.py
# Expected: All checks pass

# Test Cline interaction
# 1. Open Cline panel in VS Code
# 2. Send test message: "Can you read the constitution?"
# 3. Verify Cline responds using local LLM
```

---

### Week 3: Books System (Knowledge Infrastructure)

#### Objectives
- Books directory structure established and documented
- Agent base class implemented with Books access
- Notebook session management operational
- Skills markdown template created
- Initial skill documentation written

#### Tasks

**Day 1-2: Agent Base Class and Books Access**

1. **Implement Agent Base Class**
   ```python
   # reshark/agents/base.py
   """
   Base class for all reShark Agents.
   
   Agents are autonomous components that perform specialized analysis tasks.
   They access knowledge through the Books system and invoke Tools for execution.
   """
   
   from abc import ABC, abstractmethod
   from typing import Dict, Any, Optional
   from pathlib import Path
   import json
   from datetime import datetime
   import logging
   
   logger = logging.getLogger(__name__)
   
   class AgentError(Exception):
       """Base exception for Agent errors."""
       pass
   
   class Agent(ABC):
       """
       Base class for all reShark Agents.
       
       Constitutional Requirements:
       - Agents MUST document their analysis steps
       - Agents MUST log evidence for all findings
       - Agents MUST reference skills when following methodologies
       - Agents operate in both HITL and Autonomous modes
       """
       
       def __init__(
           self,
           books_path: Path,
           tools_registry: Optional[Any] = None
       ):
           """
           Initialize agent with access to Books.
           
           Args:
               books_path: Root path to books/ directory
               tools_registry: Optional tools registry for invoking external tools
           """
           self.books_path = Path(books_path)
           self.tools_registry = tools_registry
           self.session_id: Optional[str] = None
           
           # Books paths
           self.notebook_path = self.books_path / "notebook"
           self.rulebook_path = self.books_path / "rulebook"
           self.cookbook_path = self.books_path / "cookbook"
           self.skills_path = Path("skills")  # Skills are in workspace root
           
           # Verify paths exist
           for path in [self.notebook_path, self.rulebook_path, 
                       self.cookbook_path]:
               path.mkdir(parents=True, exist_ok=True)
       
       @abstractmethod
       def execute(self, inputs: Dict[str, Any]) -> Dict[str, Any]:
           """
           Main execution method for Agent logic.
           
           Args:
               inputs: Dictionary of input parameters
               
           Returns:
               Dictionary of results
               
           Raises:
               AgentError: If execution fails
           """
           pass
       
       @property
       @abstractmethod
       def agent_type(self) -> str:
           """
           Return agent type/category.
           
           Returns one of: interpreters, orchestration, validation, scopes
           """
           pass
       
       # Notebook operations (read/write for session data)
       
       def write_notebook(self, key: str, data: Any) -> None:
           """
           Write data to Notebook for current session.
           
           Args:
               key: Data key (becomes filename)
               data: Data to write (must be JSON-serializable)
           """
           if not self.session_id:
               raise AgentError("session_id not set")
           
           session_dir = self.notebook_path / "sessions" / self.session_id
           session_dir.mkdir(parents=True, exist_ok=True)
           
           entry = {
               "timestamp": datetime.utcnow().isoformat(),
               "agent": self.__class__.__name__,
               "agent_type": self.agent_type,
               "data": data
           }
           
           notebook_file = session_dir / f"{key}.json"
           
           try:
               with open(notebook_file, 'w') as f:
                   json.dump(entry, f, indent=2)
               logger.info(f"Wrote to Notebook: {key}")
           except TypeError as e:
               raise AgentError(f"Data not JSON-serializable: {key}") from e
       
       def read_notebook(self, key: str) -> Optional[Any]:
           """
           Read data from Notebook for current session.
           
           Args:
               key: Data key
               
           Returns:
               Data if exists, None otherwise
           """
           if not self.session_id:
               raise AgentError("session_id not set")
           
           notebook_file = (
               self.notebook_path / "sessions" / self.session_id / f"{key}.json"
           )
           
           if not notebook_file.exists():
               logger.debug(f"Notebook key not found: {key}")
               return None
           
           try:
               with open(notebook_file) as f:
                   return json.load(f)
           except json.JSONDecodeError as e:
               raise AgentError(f"Invalid JSON in Notebook: {key}") from e
       
       # Rulebook operations (read grammars, write via validation)
       
       def read_grammar(self, grammar_name: str) -> Optional[str]:
           """
           Read grammar from Rulebook.
           
           Args:
               grammar_name: Name of grammar file (with or without .ksy extension)
               
           Returns:
               Grammar content if exists, None otherwise
           """
           # Support both .ksy (Kaitai Struct) and .py (Python) grammars
           for ext in ['.ksy', '.py', '']:
               grammar_name_clean = grammar_name.replace('.ksy', '').replace('.py', '')
               grammar_file = self.rulebook_path / "grammars" / f"{grammar_name_clean}{ext}"
               
               if grammar_file.exists():
                   return grammar_file.read_text()
           
           logger.debug(f"Grammar not found: {grammar_name}")
           return None
       
       # Skills operations (read markdown documentation)
       
       def read_skill(self, skill_path: str) -> Optional[str]:
           """
           Read skill methodology from markdown file.
           
           Args:
               skill_path: Path to skill (e.g., "protocol/header-parsing.md")
               
           Returns:
               Skill content if exists, None otherwise
               
           Skills guide agent behavior by documenting analysis methodologies
           that LLMs can read and execute.
           """
           skill_file = self.skills_path / skill_path
           
           if not skill_file.exists():
               logger.debug(f"Skill not found: {skill_path}")
               return None
           
           logger.info(f"Reading skill: {skill_path}")
           return skill_file.read_text()
       
       # Tool invocation
       
       def invoke_tool(self, tool_name: str, params: Dict[str, Any]) -> Dict[str, Any]:
           """
           Invoke external tool via tools registry.
           
           Args:
               tool_name: Name of tool (e.g., "tshark", "entropy")
               params: Tool parameters
               
           Returns:
               Tool output
               
           Raises:
               AgentError: If tool invocation fails
           """
           if not self.tools_registry:
               raise AgentError("No tools registry configured")
           
           try:
               return self.tools_registry.invoke(tool_name, params)
           except Exception as e:
               raise AgentError(f"Tool invocation failed: {e}") from e
       
       # Evidence logging
       
       def log_evidence(self, evidence: Dict[str, Any]) -> None:
           """
           Log evidence for analysis findings.
           
           Args:
               evidence: Dictionary describing evidence trail
           """
           timestamp = datetime.utcnow().timestamp()
           evidence_key = f"evidence_{timestamp}"
           
           self.write_notebook(evidence_key, {
               "agent": self.__class__.__name__,
               "agent_type": self.agent_type,
               "evidence": evidence
           })
           
           logger.info(f"Evidence logged: {evidence_key}")
   ```

2. **Unit Tests for Agent Base Class**
   ```python
   # tests/test_agent_base.py
   import pytest
   from pathlib import Path
   from reshark.agents.base import Agent, AgentError
   
   class MockAgent(Agent):
       """Mock agent for testing."""
       
       def execute(self, inputs):
           return {"result": "mock"}
       
       @property
       def agent_type(self):
           return "interpreters"
   
   def test_agent_notebook_operations(tmp_path):
       """Test Notebook read/write."""
       books_path = tmp_path / "books"
       agent = MockAgent(books_path)
       agent.session_id = "test-session-001"
       
       # Write data
       agent.write_notebook("test_data", {"analysis": "result"})
       
       # Read data back
       data = agent.read_notebook("test_data")
       assert data is not None
       assert data["data"]["analysis"] == "result"
       assert data["agent"] == "MockAgent"
       assert data["agent_type"] == "interpreters"
   
   def test_agent_read_skill(tmp_path):
       """Test reading skill markdown files."""
       books_path = tmp_path / "books"
       skills_path = tmp_path / "skills"
       skills_path.mkdir()
       
       # Create test skill
       skill_dir = skills_path / "protocol"
       skill_dir.mkdir()
       skill_file = skill_dir / "test-skill.md"
       skill_file.write_text("# Test Skill\n\nMethodology here")
       
       # Read skill
       agent = MockAgent(books_path)
       agent.skills_path = skills_path
       
       content = agent.read_skill("protocol/test-skill.md")
       assert content is not None
       assert "Test Skill" in content
   ```

**Day 3-4: Books Structure and Documentation**

3. **Create Books README files**
   ```bash
   # Notebook README
   cat > books/notebook/README.md << 'EOF'
   # Notebook - Session Working Memory
   
   Short-term working memory for analysis sessions. Organized by session ID.
   
   ## Structure
   
   ```
   notebook/
   ├── sessions/
   │   └── session-YYYYMMDD-HHMMSS/
   │       ├── observations.json
   │       ├── hypotheses.json
   │       ├── evidence_*.json
   │       └── results.json
   └── archives/
       └── session-YYYYMMDD-HHMMSS.tar.gz
   ```
   
   ## Session Lifecycle
   
   1. Session created with unique ID
   2. Agents write analysis data during session
   3. Session archived on completion
   4. Archives retained for audit trail
   
   ## Data Format
   
   All entries are JSON with metadata:
   ```json
   {
     "timestamp": "2026-01-15T10:30:00Z",
     "agent": "DataInterpreter",
     "agent_type": "interpreters",
     "data": { ... }
   }
   ```
   EOF
   
   # Rulebook README
   cat > books/rulebook/README.md << 'EOF'
   # Rulebook - Validated Grammars
   
   Contains validated artifact grammars and schemas.
   
   ## Structure
   
   ```
   rulebook/
   ├── grammars/
   │   ├── protocol_x_v1.ksy      # Kaitai Struct format
   │   ├── binary_format_v2.ksy
   │   └── file_format_v1.py      # Python parsers
   └── schemas/
       └── validation_results.json
   ```
   
   ## Grammar Formats
   
   - **Kaitai Struct (.ksy)**: Declarative binary format specification
   - **Python (.py)**: Custom parsers for complex formats
   
   ## Validation Requirements
   
   Before promotion to Rulebook:
   1. Tested against 3+ samples
   2. Boundary cases handled
   3. Documentation complete
   4. Human review approved
   EOF
   
   # Cookbook README
   cat > books/cookbook/README.md << 'EOF'
   # Cookbook - Analysis Methods
   
   Documented analysis procedures and workflows.
   
   ## Structure
   
   ```
   cookbook/
   ├── methods/
   │   ├── entropy-analysis.md
   │   ├── frequency-analysis.md
   │   └── boundary-detection.md
   └── examples/
       └── pcap-analysis-workflow.md
   ```
   
   ## Method Template
   
   ```markdown
   # Method Name
   
   ## Purpose
   Brief description
   
   ## Procedure
   Step-by-step instructions
   
   ## Tools Required
   List of tools
   
   ## Example
   Concrete example
   
   ## References
   Related skills, agents
   ```
   EOF
   ```

**Day 5: Skills Template and Initial Documentation**

4. **Create Skills Template**
   ```bash
   cat > skills/TEMPLATE.md << 'EOF'
   # [Skill Name]
   
   **Category**: [protocol|pattern|binary|grammar]  
   **Complexity**: [basic|intermediate|advanced]  
   **Prerequisites**: [Required tools, prior knowledge]
   
   ## Purpose
   
   One-sentence description of what this skill accomplishes.
   
   ## When to Use
   
   - Context/scenario 1
   - Context/scenario 2
   - Context/scenario 3
   
   ## Methodology
   
   ### Step 1: [Clear Action Verb]
   
   Specific instructions for this step...
   
   **Tools Required**: tool1, tool2  
   **Expected Output**: Description of what you should see
   
   ### Step 2: [Next Action]
   
   Continue with next step...
   
   ### Step 3: [Final Action]
   
   Concluding steps...
   
   ## Example
   
   ### Input
   Description or example of input data.
   
   ### Process
   Step-by-step walkthrough with concrete data.
   
   ### Output
   Expected results and interpretation.
   
   ## Interpretation Guide
   
   How to understand and act on the results:
   - Pattern A indicates X
   - Pattern B indicates Y
   - Pattern C requires further analysis
   
   ## Common Pitfalls
   
   - Pitfall 1 and how to avoid it
   - Pitfall 2 and how to avoid it
   
   ## Related Skills
   
   - [Related Skill A](../category/skill-a.md)
   - [Related Skill B](../category/skill-b.md)
   
   ## References
   
   - External documentation links
   - Research papers
   - Tool documentation
   EOF
   ```

5. **Create Initial Skill: Data Extraction**
   ```bash
   cat > skills/protocol/data-extraction.md << 'EOF'
   # Data Extraction from Network Captures
   
   **Category**: protocol  
   **Complexity**: basic  
   **Prerequisites**: tshark installed, basic PCAP understanding
   
   ## Purpose
   
   Extract raw packet data from PCAP files for analysis.
   
   ## When to Use
   
   - Beginning protocol reverse engineering analysis
   - Need to examine packet payloads
   - Preparing data for statistical analysis
   
   ## Methodology
   
   ### Step 1: Extract TCP/UDP Streams
   
   Use tshark to extract stream data:
   
   ```bash
   tshark -r input.pcap -q -z follow,tcp,ascii,0
   ```
   
   **Tools Required**: tshark  
   **Expected Output**: ASCII representation of TCP stream 0
   
   ### Step 2: Export Stream to File
   
   Extract raw bytes:
   
   ```bash
   tshark -r input.pcap -q -z follow,tcp,raw,0 > stream_0.raw
   ```
   
   ### Step 3: List All Streams
   
   Identify available streams:
   
   ```bash
   tshark -r input.pcap -q -z conv,tcp
   ```
   
   ## Example
   
   ### Input
   PCAP file: `sample_protocol.pcap`
   
   ### Process
   
   1. List TCP conversations:
      ```bash
      tshark -r sample_protocol.pcap -q -z conv,tcp
      ```
      Output shows 5 TCP streams.
   
   2. Extract stream 0:
      ```bash
      tshark -r sample_protocol.pcap -q -z follow,tcp,raw,0 > stream_0.raw
      ```
   
   3. Verify extraction:
      ```bash
      hexdump -C stream_0.raw | head -n 10
      ```
   
   ### Output
   
   Raw binary file ready for pattern analysis.
   
   ## Interpretation Guide
   
   - Multiple streams may represent different protocol phases
   - Stream 0 often contains handshake/negotiation
   - Look for repeated patterns across streams
   
   ## Common Pitfalls
   
   - Missing `-q` flag causes verbose output
   - Stream numbers are 0-indexed
   - UDP streams numbered separately from TCP
   
   ## Related Skills
   
   - [Entropy Analysis](../pattern/entropy-analysis.md)
   - [Frequency Analysis](../pattern/frequency-analysis.md)
   
   ## References
   
   - [tshark documentation](https://www.wireshark.org/docs/man-pages/tshark.html)
   - [Wireshark Follow Stream](https://wiki.wireshark.org/Follow-Stream)
   EOF
   ```

#### Deliverables
- [ ] Agent base class with Books access implemented
- [ ] Books directory structure documented
- [ ] Notebook session management operational
- [ ] Skills template created
- [ ] At least 1 initial skill documented (data extraction)
- [ ] Unit tests for Agent base class pass

#### Validation Checkpoint
```bash
# Test agent base functionality
poetry run pytest tests/test_agent_base.py -v
# Expected: All tests pass

# Verify Books structure
ls -R books/
# Expected: notebook/, rulebook/, cookbook/ with README files

# Verify skills structure
ls -R skills/
# Expected: protocol/, pattern/, binary/, grammar/ with TEMPLATE.md
---

### Week 4: Agents - Interpreters (Data Analysis Agents)

#### Objectives
- Interpreter agents implemented for data extraction and analysis
- Tool wrappers for tshark, hexdump, binutils created
- Skills documentation for data interpretation created
- HITL workflow for data analysis operational

#### Tasks

**Day 1-2: Data Interpreter Agent**

1. **Implement Data Interpreter**
   ```python
   # reshark/agents/interpreters/data_interpreter.py
   """
   Data Interpreter Agent - Extracts and interprets raw data from artifacts.
   
   Capabilities:
   - Extract streams from PCAPs
   - Parse binary structures
   - Identify data boundaries
   - Compute basic statistics
   """
   
   from typing import Dict, Any, List, Optional
   from pathlib import Path
   import logging
   
   from reshark.agents.base import Agent, AgentError
   
   logger = logging.getLogger(__name__)
   
   class DataInterpreter(Agent):
       """
       Agent specialized in data extraction and basic interpretation.
       
       Follows skills:
       - protocol/data-extraction.md
       - pattern/boundary-detection.md
       """
       
       @property
       def agent_type(self) -> str:
           return "interpreters"
       
       def execute(self, inputs: Dict[str, Any]) -> Dict[str, Any]:
           """
           Execute data interpretation on input artifact.
           
           Args:
               inputs: {
                   "artifact_path": Path to input file (PCAP, binary, etc.),
                   "artifact_type": "pcap" | "binary" | "file",
                   "skill": Optional skill to follow
               }
               
           Returns:
               {
                   "streams": List of extracted data streams,
                   "metadata": Artifact metadata,
                   "statistics": Basic statistics
               }
           """
           artifact_path = Path(inputs["artifact_path"])
           artifact_type = inputs.get("artifact_type", "pcap")
           
           if not artifact_path.exists():
               raise AgentError(f"Artifact not found: {artifact_path}")
           
           # Read relevant skill if specified
           skill_path = inputs.get("skill")
           if skill_path:
               skill_content = self.read_skill(skill_path)
               logger.info(f"Following skill: {skill_path}")
               # In HITL mode, Cline reads this and guides execution
               # In Autonomous mode, LLM reads this to determine steps
           
           # Execute based on artifact type
           if artifact_type == "pcap":
               return self._interpret_pcap(artifact_path)
           elif artifact_type == "binary":
               return self._interpret_binary(artifact_path)
           else:
               raise AgentError(f"Unsupported artifact type: {artifact_type}")
       
       def _interpret_pcap(self, pcap_path: Path) -> Dict[str, Any]:
           """Extract and interpret PCAP data."""
           logger.info(f"Interpreting PCAP: {pcap_path}")
           
           # Invoke tshark tool to list conversations
           conversations = self.invoke_tool("tshark", {
               "command": "conversations",
               "input": str(pcap_path),
               "protocol": "tcp"
           })
           
           # Extract streams
           streams = []
           if conversations.get("success"):
               tcp_convs = conversations.get("tcp_conversations", [])
               
               for idx, conv in enumerate(tcp_convs[:10]):  # Limit to first 10
                   stream_data = self.invoke_tool("tshark", {
                       "command": "follow_stream",
                       "input": str(pcap_path),
                       "stream_type": "tcp",
                       "stream_id": idx,
                       "format": "raw"
                   })
                   
                   if stream_data.get("success"):
                       streams.append({
                           "stream_id": idx,
                           "data": stream_data["data"],
                           "size": len(stream_data["data"]),
                           "conversation": conv
                       })
           
           # Log evidence
           self.log_evidence({
               "method": "pcap_interpretation",
               "tool": "tshark",
               "streams_found": len(streams),
               "artifact": str(pcap_path)
           })
           
           # Write to notebook
           self.write_notebook("extracted_streams", {
               "artifact": str(pcap_path),
               "streams": streams,
               "count": len(streams)
           })
           
           return {
               "streams": streams,
               "metadata": {
                   "artifact_type": "pcap",
                   "stream_count": len(streams)
               },
               "statistics": self._compute_stream_statistics(streams)
           }
       
       def _interpret_binary(self, binary_path: Path) -> Dict[str, Any]:
           """Extract and interpret binary file data."""
           logger.info(f"Interpreting binary: {binary_path}")
           
           # Invoke file tool to identify file type
           file_info = self.invoke_tool("file", {
               "input": str(binary_path)
           })
           
           # Extract strings
           strings_data = self.invoke_tool("strings", {
               "input": str(binary_path),
               "min_length": 4
           })
           
           # Get hexdump sample
           hexdump = self.invoke_tool("hexdump", {
               "input": str(binary_path),
               "length": 512,  # First 512 bytes
               "format": "canonical"
           })
           
           result = {
               "streams": [{
                   "stream_id": 0,
                   "data": binary_path.read_bytes(),
                   "size": binary_path.stat().st_size
               }],
               "metadata": {
                   "artifact_type": "binary",
                   "file_type": file_info.get("file_type"),
                   "size": binary_path.stat().st_size
               },
               "analysis": {
                   "strings": strings_data.get("strings", []),
                   "hexdump_preview": hexdump.get("output", "")
               }
           }
           
           self.log_evidence({
               "method": "binary_interpretation",
               "tools": ["file", "strings", "hexdump"],
               "artifact": str(binary_path)
           })
           
           self.write_notebook("extracted_binary", result)
           
           return result
       
       def _compute_stream_statistics(self, streams: List[Dict]) -> Dict:
           """Compute basic statistics for streams."""
           if not streams:
               return {}
           
           sizes = [s["size"] for s in streams]
           
           return {
               "total_streams": len(streams),
               "total_bytes": sum(sizes),
               "min_size": min(sizes),
               "max_size": max(sizes),
               "avg_size": sum(sizes) / len(sizes)
           }
   ```

2. **Unit Tests for Data Interpreter**
   ```python
   # tests/agents/interpreters/test_data_interpreter.py
   import pytest
   from pathlib import Path
   from reshark.agents.interpreters.data_interpreter import DataInterpreter
   
   @pytest.fixture
   def mock_tools_registry():
       """Mock tools registry for testing."""
       class MockRegistry:
           def invoke(self, tool_name, params):
               if tool_name == "tshark":
                   if params.get("command") == "conversations":
                       return {
                           "success": True,
                           "tcp_conversations": [
                               {"src": "192.168.1.1", "dst": "192.168.1.2"}
                           ]
                       }
                   elif params.get("command") == "follow_stream":
                       return {
                           "success": True,
                           "data": b"test_data"
                       }
               return {"success": False}
       
       return MockRegistry()
   
   def test_data_interpreter_initialization(tmp_path, mock_tools_registry):
       """Test DataInterpreter can be initialized."""
       books_path = tmp_path / "books"
       agent = DataInterpreter(books_path, mock_tools_registry)
       agent.session_id = "test-001"
       
       assert agent.agent_type == "interpreters"
   
   def test_read_skill(tmp_path, mock_tools_registry):
       """Test agent can read skill documentation."""
       books_path = tmp_path / "books"
       skills_path = tmp_path / "skills" / "protocol"
       skills_path.mkdir(parents=True)
       
       # Create test skill
       skill_file = skills_path / "data-extraction.md"
       skill_file.write_text("# Data Extraction\n\nMethodology...")
       
       agent = DataInterpreter(books_path, mock_tools_registry)
       agent.skills_path = tmp_path / "skills"
       
       content = agent.read_skill("protocol/data-extraction.md")
       assert "Data Extraction" in content
   ```

**Day 3: Pattern Interpreter Agent**

3. **Implement Pattern Interpreter**
   ```python
   # reshark/agents/interpreters/pattern_interpreter.py
   """
   Pattern Interpreter Agent - Identifies patterns in data.
   
   Capabilities:
   - Compute entropy maps
   - Analyze byte frequency distributions
   - Detect repeated sequences
   - Identify anomalies
   """
   
   from typing import Dict, Any, List
   import logging
   from collections import Counter
   import math
   
   from reshark.agents.base import Agent, AgentError
   
   logger = logging.getLogger(__name__)
   
   class PatternInterpreter(Agent):
       """
       Agent specialized in pattern recognition and statistical analysis.
       
       Follows skills:
       - pattern/entropy-analysis.md
       - pattern/frequency-analysis.md
       - pattern/sequence-detection.md
       """
       
       @property
       def agent_type(self) -> str:
           return "interpreters"
       
       def execute(self, inputs: Dict[str, Any]) -> Dict[str, Any]:
           """
           Execute pattern analysis on data streams.
           
           Args:
               inputs: {
                   "streams": List of data streams to analyze,
                   "analysis_type": "entropy" | "frequency" | "sequences"
               }
               
           Returns:
               {
                   "entropy_map": Per-position entropy values,
                   "frequency_dist": Byte frequency distribution,
                   "sequences": Detected repeated sequences
               }
           """
           streams = inputs.get("streams", [])
           analysis_type = inputs.get("analysis_type", "all")
           
           if not streams:
               raise AgentError("No streams provided for analysis")
           
           results = {}
           
           if analysis_type in ["entropy", "all"]:
               results["entropy"] = self._analyze_entropy(streams)
           
           if analysis_type in ["frequency", "all"]:
               results["frequency"] = self._analyze_frequency(streams)
           
           if analysis_type in ["sequences", "all"]:
               results["sequences"] = self._detect_sequences(streams)
           
           # Write results to notebook
           self.write_notebook("pattern_analysis", results)
           
           return results
       
       def _analyze_entropy(self, streams: List[Dict]) -> Dict:
           """
           Compute Shannon entropy for each byte position.
           Follows: skills/pattern/entropy-analysis.md
           """
           logger.info("Computing entropy map")
           
           # Extract byte arrays
           byte_streams = [s["data"] for s in streams if isinstance(s["data"], bytes)]
           
           if not byte_streams:
               return {}
           
           # Find max length
           max_len = max(len(s) for s in byte_streams)
           
           entropy_map = {}
           
           for pos in range(max_len):
               # Collect bytes at this position across all streams
               bytes_at_pos = []
               for stream in byte_streams:
                   if pos < len(stream):
                       bytes_at_pos.append(stream[pos])
               
               if bytes_at_pos:
                   # Compute entropy
                   entropy = self._shannon_entropy(bytes_at_pos)
                   entropy_map[pos] = {
                       "entropy": entropy,
                       "classification": self._classify_entropy(entropy)
                   }
           
           self.log_evidence({
               "method": "entropy_analysis",
               "positions_analyzed": len(entropy_map),
               "streams_count": len(byte_streams)
           })
           
           return entropy_map
       
       def _shannon_entropy(self, data: List[int]) -> float:
           """Calculate Shannon entropy."""
           if not data:
               return 0.0
           
           counter = Counter(data)
           total = len(data)
           
           entropy = 0.0
           for count in counter.values():
               p = count / total
               if p > 0:
                   entropy -= p * math.log2(p)
           
           return entropy
       
       def _classify_entropy(self, entropy: float) -> str:
           """Classify entropy level."""
           if entropy < 2.0:
               return "control"  # Low variation
           elif entropy > 6.0:
               return "data"  # High variation
           elif entropy == 0.0:
               return "invariant"  # Fixed
           else:
               return "mixed"
       
       def _analyze_frequency(self, streams: List[Dict]) -> Dict:
           """
           Analyze byte frequency distribution.
           Follows: skills/pattern/frequency-analysis.md
           """
           logger.info("Analyzing byte frequency")
           
           all_bytes = []
           for s in streams:
               if isinstance(s["data"], bytes):
                   all_bytes.extend(s["data"])
           
           if not all_bytes:
               return {}
           
           counter = Counter(all_bytes)
           total = len(all_bytes)
           
           frequency_dist = {
               byte_val: {
                   "count": count,
                   "frequency": count / total
               }
               for byte_val, count in counter.items()
           }
           
           self.log_evidence({
               "method": "frequency_analysis",
               "total_bytes": total,
               "unique_bytes": len(counter)
           })
           
           return {
               "distribution": frequency_dist,
               "total_bytes": total,
               "unique_bytes": len(counter)
           }
       
       def _detect_sequences(self, streams: List[Dict]) -> List[Dict]:
           """
           Detect repeated byte sequences.
           Follows: skills/pattern/sequence-detection.md
           """
           logger.info("Detecting repeated sequences")
           
           # Simple implementation: find 4-byte sequences that repeat
           sequences = []
           
           for stream in streams:
               if not isinstance(stream["data"], bytes):
                   continue
               
               data = stream["data"]
               min_len = 4
               
               for i in range(len(data) - min_len + 1):
                   seq = data[i:i+min_len]
                   
                   # Count occurrences
                   count = data.count(seq)
                   
                   if count > 1:
                       # Find all positions
                       positions = []
                       start = 0
                       while True:
                           pos = data.find(seq, start)
                           if pos == -1:
                               break
                           positions.append(pos)
                           start = pos + 1
                       
                       sequences.append({
                           "sequence": seq.hex(),
                           "length": len(seq),
                           "count": count,
                           "positions": positions
                       })
           
           # Remove duplicates
           unique_sequences = []
           seen = set()
           for seq in sequences:
               key = seq["sequence"]
               if key not in seen:
                   seen.add(key)
                   unique_sequences.append(seq)
           
           return unique_sequences
   ```

**Day 4-5: Tool Wrappers**

4. **Implement Tool Wrappers**
   ```python
   # reshark/tools/tshark_wrapper.py
   """
   Wrapper for tshark command-line tool.
   """
   
   import subprocess
   from typing import Dict, Any, List, Optional
   from pathlib import Path
   import logging
   
   logger = logging.getLogger(__name__)
   
   class TsharkWrapper:
       """Wrapper for tshark operations."""
       
       def __init__(self, tshark_path: str = "tshark"):
           self.tshark_path = tshark_path
           self._verify_installation()
       
       def _verify_installation(self):
           """Verify tshark is installed."""
           try:
               subprocess.run(
                   [self.tshark_path, "--version"],
                   capture_output=True,
                   check=True
               )
           except (subprocess.CalledProcessError, FileNotFoundError):
               raise RuntimeError(f"tshark not found at {self.tshark_path}")
       
       def list_conversations(
           self,
           pcap_path: Path,
           protocol: str = "tcp"
       ) -> Dict[str, Any]:
           """List conversations in PCAP."""
           cmd = [
               self.tshark_path,
               "-r", str(pcap_path),
               "-q",
               "-z", f"conv,{protocol}"
           ]
           
           try:
               result = subprocess.run(
                   cmd,
                   capture_output=True,
                   text=True,
                   timeout=60
               )
               
               if result.returncode == 0:
                   # Parse output
                   conversations = self._parse_conversations(result.stdout, protocol)
                   return {
                       "success": True,
                       f"{protocol}_conversations": conversations
                   }
               else:
                   return {
                       "success": False,
                       "error": result.stderr
                   }
           
           except subprocess.TimeoutExpired:
               return {"success": False, "error": "timeout"}
       
       def follow_stream(
           self,
           pcap_path: Path,
           stream_type: str,
           stream_id: int,
           format: str = "raw"
       ) -> Dict[str, Any]:
           """Extract stream data."""
           cmd = [
               self.tshark_path,
               "-r", str(pcap_path),
               "-q",
               "-z", f"follow,{stream_type},{format},{stream_id}"
           ]
           
           try:
               result = subprocess.run(
                   cmd,
                   capture_output=True,
                   timeout=60
               )
               
               if result.returncode == 0:
                   return {
                       "success": True,
                       "data": result.stdout
                   }
               else:
                   return {
                       "success": False,
                       "error": result.stderr.decode()
                   }
           
           except subprocess.TimeoutExpired:
               return {"success": False, "error": "timeout"}
       
       def _parse_conversations(self, output: str, protocol: str) -> List[Dict]:
           """Parse tshark conversation output."""
           # Simple parser - would need refinement
           conversations = []
           lines = output.strip().split('\n')
           
           for line in lines:
               if '<->' in line:
                   parts = line.split()
                   if len(parts) >= 2:
                       conversations.append({
                           "src": parts[0],
                           "dst": parts[2] if len(parts) > 2 else "unknown"
                       })
           
           return conversations
   ```

#### Deliverables
- [ ] DataInterpreter agent implemented and tested
- [ ] PatternInterpreter agent implemented and tested
- [ ] Tool wrappers (tshark, hexdump, file) operational
- [ ] Skills documentation updated with interpretation methods
- [ ] HITL workflow tested with sample PCAP

#### Validation Checkpoint
```bash
# Test interpreters
poetry run pytest tests/agents/interpreters/ -v
# Expected: All tests pass

# Test HITL workflow (with Cline)
# 1. Open Cline
# 2. Request: "Analyze sample.pcap using DataInterpreter"
# 3. Verify: Agent reads skill, invokes tools, writes notebook
```

---

### Week 8: Phase 1 Integration

#### Objectives
- Complete workflow orchestration script
- End-to-end testing
- Documentation finalization
- Phase 1 success criteria validated

#### Tasks

1. **Main Orchestration Script** (Day 1-2)
2. **Synthetic Protocol Generator** (Day 3)
3. **End-to-End Testing** (Day 4)
4. **Documentation** (Day 5)

#### Phase 1 Gate Review

**Criteria:**
- [ ] One complete protocol analysis successful
- [ ] All Scopes functional with >80% test coverage
- [ ] Constitutional compliance verified
- [ ] Memory boundaries enforced
- [ ] Documentation complete

**Go/No-Go Decision**: If all criteria met, proceed to Phase 2. Otherwise, extend Phase 1 by 1-2 weeks.

---

## PHASE 2: SCALE & AUTONOMY (WEEKS 9-16)

---

### Week 9-10: LangGraph Integration

#### Objectives
- Install and configure LangGraph
- Define workflow graph
- Wrap Scopes as nodes
- Test state management

#### Tasks

**Day 1-2: LangGraph Setup**

1. **Install Dependencies**
   ```bash
   # Add LangGraph dependencies
   poetry add langgraph langchain langchain-core
   poetry add langgraph-checkpoint-sqlite  # For state persistence
   ```

2. **Create Graph Definition**
   ```python
   # reshark/orchestration/graph.py
   """
   LangGraph workflow for autonomous protocol analysis.
   
   Phase 2 replaces manual orchestration scripts with
   autonomous graph-based execution per Constitution Section 10.
   """
   
   from typing import Dict, Any, Literal
   from langgraph.graph import StateGraph, END
   from langgraph.checkpoint.sqlite import SqliteSaver
   from pathlib import Path
   
   from reshark.scopes import (
       Observer,
       Theorist,
       Validator,
       Archivist
   )
   
   class AnalysisState(Dict):
       """
       State object passed between graph nodes.
       
       Contains:
       - session_id: Unique analysis session identifier
       - pcaps: List of PCAP file paths
       - observations: Observer results
       - hypotheses: Theorist results
       - validation: Validator results
       - status: Current workflow status
       """
       pass
   
   def create_analysis_graph(workspace: Path) -> StateGraph:
       """
       Create LangGraph workflow for protocol analysis.
       
       Graph structure:
       START → observe → theorize → validate → review → [archive | refine | END]
       """
       
       # Initialize Scopes
       notebook_path = workspace / "notebook"
       rulebook_path = workspace / "rulebook"
       cookbook_path = workspace / "cookbook"
       
       observer = Observer(notebook_path, rulebook_path, cookbook_path)
       theorist = Theorist(notebook_path, rulebook_path, cookbook_path)
       validator = Validator(notebook_path, rulebook_path, cookbook_path)
       archivist = Archivist(notebook_path, rulebook_path, cookbook_path)
       
       # Define graph
       workflow = StateGraph(AnalysisState)
       
       # Add nodes (Scope wrappers)
       workflow.add_node("observe", lambda state: observe_node(observer, state))
       workflow.add_node("theorize", lambda state: theorize_node(theorist, state))
       workflow.add_node("validate", lambda state: validate_node(validator, state))
       workflow.add_node("review", review_checkpoint_node)
       workflow.add_node("archive", lambda state: archive_node(archivist, state))
       
       # Define edges
       workflow.add_edge("observe", "theorize")
       workflow.add_edge("theorize", "validate")
       workflow.add_edge("validate", "review")
       
       # Conditional routing after review
       workflow.add_conditional_edges(
           "review",
           route_after_review,
           {
               "promote": "archive",
               "refine": "theorize",  # Loop back for refinement
               "reject": END
           }
       )
       
       workflow.add_edge("archive", END)
       
       # Set entry point
       workflow.set_entry_point("observe")
       
       # Add checkpointer for state persistence
       checkpointer = SqliteSaver.from_conn_string(
           str(workspace / "checkpoints.db")
       )
       
       return workflow.compile(checkpointer=checkpointer)
   
   def observe_node(observer: Observer, state: AnalysisState) -> AnalysisState:
       """Observation node wrapper."""
       observer.session_id = state["session_id"]
       
       results = observer.execute({
           "pcap_path": state["pcaps"][0],  # Golden PCAP
           "session_id": state["session_id"]
       })
       
       state["observations"] = results
       state["status"] = "observed"
       return state
   
   def theorize_node(theorist: Theorist, state: AnalysisState) -> AnalysisState:
       """Theorist node wrapper."""
       theorist.session_id = state["session_id"]
       
       results = theorist.execute({
           "session_id": state["session_id"]
       })
       
       state["hypotheses"] = results
       state["status"] = "theorized"
       return state
   
   def validate_node(validator: Validator, state: AnalysisState) -> AnalysisState:
       """Validator node wrapper."""
       validator.session_id = state["session_id"]
       
       results = validator.execute({
           "session_id": state["session_id"],
           "validation_pcaps": state["pcaps"][1:]  # All except golden
       })
       
       state["validation"] = results
       state["status"] = "validated"
       return state
   
   def review_checkpoint_node(state: AnalysisState) -> AnalysisState:
       """
       Human review checkpoint (Phase 2: can be automated).
       
       In Phase 2, this can become fully automated based on
       validation confidence scores.
       """
       validation = state["validation"]
       
       # Auto-promote if confidence > 0.9 and all validations pass
       if len(validation["data"]["passed"]) > 0:
           hypotheses = state["hypotheses"]["hypotheses"]
           passed_hyps = [hypotheses[i] for i in validation["data"]["passed"]]
           
           if all(h.get("confidence", 0) > 0.9 for h in passed_hyps):
               state["review_decision"] = "promote"
               state["status"] = "auto_approved"
               return state
       
       # Otherwise, require human review (checkpoint)
       state["review_decision"] = "review_required"
       state["status"] = "pending_review"
       return state
   
   def archive_node(archivist: Archivist, state: AnalysisState) -> AnalysisState:
       """Archive node wrapper."""
       archivist.session_id = state["session_id"]
       
       # Promote each validated hypothesis
       for hyp_id in state["validation"]["data"]["passed"]:
           archivist.execute({
               "operation": "promote",
               "session_id": state["session_id"],
               "hypothesis_id": hyp_id
           })
       
       # Archive session
       archivist.execute({
           "operation": "archive",
           "session_id": state["session_id"]
       })
       
       state["status"] = "completed"
       return state
   
   def route_after_review(state: AnalysisState) -> Literal["promote", "refine", "reject"]:
       """Route based on review decision."""
       decision = state.get("review_decision", "reject")
       
       if decision == "promote":
           return "promote"
       elif decision == "refine":
           return "refine"
       else:
           return "reject"
   ```

**Day 3-5: Integration and Testing**

3. **Test Graph Execution**
   ```python
   # tests/test_langgraph.py
   import pytest
   from pathlib import Path
   from reshark.orchestration.graph import create_analysis_graph
   
   def test_graph_construction(tmp_path):
       """Test graph can be constructed."""
       graph = create_analysis_graph(tmp_path)
       assert graph is not None
   
   def test_graph_execution(tmp_path, sample_pcaps):
       """Test graph executes end-to-end."""
       graph = create_analysis_graph(tmp_path)
       
       initial_state = {
           "session_id": "test-graph-001",
           "pcaps": sample_pcaps,
           "status": "initialized"
       }
       
       # Execute graph
       final_state = graph.invoke(initial_state)
       
       assert final_state["status"] in ["completed", "pending_review"]
   ```

#### Deliverables
- [ ] LangGraph integrated
- [ ] Workflow graph defined
- [ ] Scope node wrappers implemented
- [ ] State persistence functional

---

### Week 11: Patternbook Implementation

#### Objectives
- Install Qdrant vector database
- Implement embedding generation
- Create similarity search API
- Integrate with Theorist

#### Tasks

**Day 1-2: Qdrant Setup**

1. **Add Qdrant to Docker Compose**
   ```yaml
   # docker/docker-compose.yml (Phase 2 additions)
   services:
     # ... existing reshark service ...
     
     qdrant:
       image: qdrant/qdrant:latest
       ports:
         - "6333:6333"
         - "6334:6334"
       volumes:
         - qdrant_data:/qdrant/storage
       networks:
         - isolated
   
   volumes:
     qdrant_data:
   ```

2. **Install Qdrant Client**
   ```bash
   poetry add qdrant-client
   poetry add sentence-transformers  # For embeddings
   ```

**Day 3-5: Patternbook Implementation**

3. **Create Patternbook Class**
   ```python
   # reshark/memory/patternbook.py
   """
   Patternbook: Vector similarity memory for protocol patterns.
   
   Per Constitution Section 3.3: Patternbook is NON-AUTHORITATIVE.
   Suggestions must be validated before use.
   
   Phase 2 only.
   """
   
   from typing import List, Dict, Any, Optional
   import numpy as np
   from qdrant_client import QdrantClient
   from qdrant_client.models import (
       Distance,
       VectorParams,
       PointStruct,
       Filter,
       FieldCondition,
       MatchValue
   )
   from sentence_transformers import SentenceTransformer
   import logging
   
   logger = logging.getLogger(__name__)
   
   class Patternbook:
       """
       Vector similarity memory for protocol patterns.
       
       WARNING: All suggestions from Patternbook are ADVISORY ONLY.
       They MUST be validated before being treated as truth.
       """
       
       def __init__(
           self,
           qdrant_url: str = "http://localhost:6333",
           embedding_model: str = "all-MiniLM-L6-v2"
       ):
           self.client = QdrantClient(url=qdrant_url)
           self.collection_name = "protocol_patterns"
           self.embedding_model = SentenceTransformer(embedding_model)
           self.embedding_dim = 384  # all-MiniLM-L6-v2 dimension
           
           # Initialize collection
           self._init_collection()
       
       def _init_collection(self):
           """Initialize Qdrant collection if not exists."""
           try:
               self.client.get_collection(self.collection_name)
               logger.info(f"Collection {self.collection_name} already exists")
           except Exception:
               self.client.create_collection(
                   collection_name=self.collection_name,
                   vectors_config=VectorParams(
                       size=self.embedding_dim,
                       distance=Distance.COSINE
                   )
               )
               logger.info(f"Created collection {self.collection_name}")
       
       def store_pattern(
           self,
           pattern_id: str,
           observations: Dict[str, Any],
           metadata: Dict[str, Any]
       ) -> None:
           """
           Store protocol pattern embedding.
           
           Args:
               pattern_id: Unique pattern identifier
               observations: Observer output to embed
               metadata: Additional metadata (protocol name, session, etc.)
           """
           # Generate embedding from observations
           embedding = self._embed_observations(observations)
           
           # Store in Qdrant
           self.client.upsert(
               collection_name=self.collection_name,
               points=[
                   PointStruct(
                       id=pattern_id,
                       vector=embedding.tolist(),
                       payload={
                           **metadata,
                           "observation_summary": self._summarize_observations(observations)
                       }
                   )
               ]
           )
           
           logger.info(f"Stored pattern: {pattern_id}")
       
       def find_similar(
           self,
           observations: Dict[str, Any],
           top_k: int = 5,
           min_score: float = 0.7
       ) -> List[Dict]:
           """
           Find similar protocol patterns.
           
           Args:
               observations: Current observations to match
               top_k: Number of results to return
               min_score: Minimum similarity score (0-1)
               
           Returns:
               List of similar patterns with DISCLAIMER
           """
           # Embed query observations
           query_embedding = self._embed_observations(observations)
           
           # Search Qdrant
           results = self.client.search(
               collection_name=self.collection_name,
               query_vector=query_embedding.tolist(),
               limit=top_k,
               score_threshold=min_score
           )
           
           # Format results with CONSTITUTIONAL WARNING
           similar_patterns = []
           for r in results:
               similar_patterns.append({
                   "id": r.id,
                   "score": r.score,
                   "protocol": r.payload.get("protocol_name", "unknown"),
                   "summary": r.payload.get("observation_summary", ""),
                   "DISCLAIMER": "SUGGESTION ONLY - MUST BE VALIDATED",
                   "constitutional_authority": "Section 3.3 - Non-Authoritative"
               })
           
           logger.info(f"Found {len(similar_patterns)} similar patterns")
           return similar_patterns
       
       def _embed_observations(self, observations: Dict[str, Any]) -> np.ndarray:
           """
           Convert observations to embedding vector.
           
           Uses statistical features from observations:
           - Entropy distribution
           - Control byte positions
           - Message length statistics
           """
           # Create textual summary for embedding
           summary_parts = []
           
           # Entropy patterns
           entropy_map = observations.get("entropy_map", {})
           low_entropy = [pos for pos, e in entropy_map.items() if e < 2.0]
           high_entropy = [pos for pos, e in entropy_map.items() if e > 6.0]
           
           summary_parts.append(f"Low entropy positions: {len(low_entropy)}")
           summary_parts.append(f"High entropy positions: {len(high_entropy)}")
           
           # Control bytes
           control_bytes = observations.get("control_bytes", [])
           summary_parts.append(f"Control bytes at positions: {control_bytes[:5]}")
           
           # Message lengths
           lengths = observations.get("message_lengths", {})
           summary_parts.append(
               f"Message length range: {lengths.get('min')}-{lengths.get('max')}"
           )
           
           # Invariants
           invariants = observations.get("invariants", [])
           summary_parts.append(f"Fixed positions: {len(invariants)}")
           
           summary_text = ". ".join(summary_parts)
           
           # Generate embedding
           embedding = self.embedding_model.encode(summary_text)
           return embedding
       
       def _summarize_observations(self, observations: Dict[str, Any]) -> str:
           """Create human-readable summary of observations."""
           entropy_map = observations.get("entropy_map", {})
           control_bytes = observations.get("control_bytes", [])
           invariants = observations.get("invariants", [])
           
           return (
               f"{len(control_bytes)} control bytes, "
               f"{len(invariants)} invariants, "
               f"entropy range {min(entropy_map.values()):.2f}-{max(entropy_map.values()):.2f}"
           )
   ```

4. **Integrate with Theorist**
   ```python
   # Modify Theorist to use Patternbook suggestions
   # (additions to theorist.py)
   
   from reshark.memory.patternbook import Patternbook
   
   class Theorist(Scope):
       def __init__(self, notebook_path, rulebook_path, cookbook_path, 
                    use_patternbook: bool = True):
           super().__init__(notebook_path, rulebook_path, cookbook_path)
           self.patternbook = Patternbook() if use_patternbook else None
       
       def execute(self, inputs: Dict[str, Any]) -> Dict[str, Any]:
           self.session_id = inputs["session_id"]
           
           # Read observations
           observations = self.read_notebook("observations")
           
           # Generate base hypotheses
           hypotheses = []
           hypotheses.extend(self._hypothesize_boundaries(observations))
           hypotheses.extend(self._hypothesize_message_types(observations))
           hypotheses.extend(self._hypothesize_fields(observations))
           
           # PHASE 2: Use Patternbook for analogical hints
           if self.patternbook:
               similar = self.patternbook.find_similar(
                   observations.get("data", {}),
                   top_k=3
               )
               
               if similar:
                   # Log analogies (but DO NOT treat as truth)
                   self._log_evidence({
                       "method": "analogical_reasoning",
                       "source": "Patternbook",
                       "similar_protocols": [s["protocol"] for s in similar],
                       "DISCLAIMER": similar[0]["DISCLAIMER"],
                       "action": "Used as inspiration only, not as fact"
                   })
                   
                   # Generate additional hypotheses inspired by analogies
                   # (but still requires validation)
                   analogy_hyps = self._generate_from_analogies(
                       observations,
                       similar
                   )
                   hypotheses.extend(analogy_hyps)
           
           # Write to Notebook
           self.write_notebook("hypotheses", hypotheses)
           
           return {"hypotheses": hypotheses}
   ```

#### Deliverables
- [ ] Qdrant running and accessible
- [ ] Patternbook class implemented
- [ ] Embedding generation functional
- [ ] Theorist integration complete
- [ ] Constitutional disclaimers present

---

### Week 12: Scope Adaptation for Phase 2

#### Objectives
- Refactor Scopes for LangGraph compatibility
- Add state management
- Implement progress tracking
- Optimize for concurrent execution

#### Tasks

1. **State Management** (Day 1-2)
2. **Progress Tracking** (Day 3)
3. **Concurrent Execution** (Day 4-5)

---

### Week 13: Multi-Protocol Support

#### Objectives
- Parallel analysis of multiple protocols
- Resource management
- Session isolation
- Results aggregation

#### Tasks

1. **Parallel Execution Framework** (Day 1-3)
   ```python
   # reshark/orchestration/multi_protocol.py
   """
   Multi-protocol analysis orchestration.
   
   Phase 2: Scale to 5+ concurrent protocol analyses.
   """
   
   from typing import List, Dict, Any
   from pathlib import Path
   import asyncio
   from concurrent.futures import ProcessPoolExecutor
   
   from reshark.orchestration.graph import create_analysis_graph
   
   class MultiProtocolOrchestrator:
       """
       Orchestrates concurrent analysis of multiple protocols.
       
       Per Constitution Section 10.2: Phase 2 scales to multiple
       concurrent analyses while maintaining constitutional compliance.
       """
       
       def __init__(self, workspace: Path, max_concurrent: int = 5):
           self.workspace = workspace
           self.max_concurrent = max_concurrent
           self.sessions: Dict[str, Dict[str, Any]] = {}
       
       async def analyze_protocols(
           self,
           protocol_configs: List[Dict[str, Any]]
       ) -> Dict[str, Any]:
           """
           Analyze multiple protocols concurrently.
           
           Args:
               protocol_configs: List of protocol configurations,
                   each containing:
                   - session_id: Unique identifier
                   - pcaps: List of PCAP paths
                   - protocol_name: Optional name
           
           Returns:
               Dictionary of results per session_id
           """
           # Create analysis tasks
           tasks = []
           for config in protocol_configs[:self.max_concurrent]:
               task = self._analyze_single_protocol(config)
               tasks.append(task)
           
           # Execute concurrently
           results = await asyncio.gather(*tasks, return_exceptions=True)
           
           # Aggregate results
           return {
               config["session_id"]: result
               for config, result in zip(protocol_configs, results)
           }
       
       async def _analyze_single_protocol(
           self,
           config: Dict[str, Any]
       ) -> Dict[str, Any]:
           """Analyze single protocol in isolated session."""
           graph = create_analysis_graph(self.workspace)
           
           initial_state = {
               "session_id": config["session_id"],
               "pcaps": config["pcaps"],
               "protocol_name": config.get("protocol_name", "unknown"),
               "status": "initialized"
           }
           
           # Execute graph (LangGraph handles state management)
           final_state = await asyncio.to_thread(
               graph.invoke,
               initial_state
           )
           
           return final_state
   ```

2. **Resource Management** (Day 4-5)

---

### Week 14: Performance Optimization

#### Objectives
- Profile and optimize bottlenecks
- Implement caching
- Parallel PCAP processing
- Memory optimization

#### Tasks

1. **Profiling** (Day 1-2)
2. **Caching Strategy** (Day 3)
3. **Optimization** (Day 4-5)

---

### Week 15: Advanced Features

#### Objectives
- Autonomous checkpoint system
- Learning from validation feedback
- Protocol family clustering
- Advanced Patternbook queries

#### Tasks

1. **Autonomous Checkpoints** (Day 1-2)
   - Replace human review with confidence thresholds
   - Implement escalation for low-confidence cases
   - Log all automated decisions

2. **Learning System** (Day 3-4)
   - Store validation outcomes in Patternbook
   - Update similarity models based on success rates
   - Cross-protocol pattern recognition

3. **Clustering** (Day 5)
   - Group similar protocols
   - Identify protocol families
   - Generate family-specific hypotheses

---

### Week 16: Finalization and Release

#### Objectives
- Comprehensive testing
- Documentation completion
- Performance benchmarking
- Open-source release preparation

#### Tasks

**Day 1-2: Testing**

1. **Full System Testing**
   ```bash
   # Run comprehensive test suite
   poetry run pytest tests/ -v --cov=reshark --cov-report=html
   
   # Target: >80% coverage
   
   # Constitutional compliance
   poetry run python scripts/check_compliance.py
   
   # Performance benchmarks
   poetry run python scripts/benchmark.py
   ```

2. **Real-World Protocol Testing**
   - Analyze 3+ real protocols (non-proprietary)
   - Document findings
   - Measure success rates

**Day 3: Documentation**

3. **Complete Documentation**
   - API reference (auto-generated from docstrings)
   - User guide for both phases
   - Troubleshooting guide
   - Architecture diagrams
   - Performance tuning guide

**Day 4: Benchmarking**

4. **Performance Metrics**
   - PCAP processing speed
   - Hypothesis generation time
   - Validation throughput
   - Multi-protocol scaling
   - Memory usage profiles

**Day 5: Release Preparation**

5. **Open-Source Release**
   ```bash
   # Per Constitution Section 7: Open-Core model
   
   # Include in public release:
   # - All infrastructure code
   # - Scope implementations
   # - Orchestration framework
   # - Documentation
   # - Synthetic protocol examples
   
   # Exclude from public release:
   # - Real-world proprietary grammars
   # - Customer PCAP analyses
   # - Production configuration
   ```

6. **Final Review**
   - Constitutional compliance audit
   - Security review
   - License verification (Apache 2.0 for infrastructure)
   - Contributor guidelines
   - Code of conduct

---

## 5. Resource Requirements

### 5.1 Hardware

- **Development**: Modern workstation with sufficient memory and storage
- **Production**: Sufficient RAM recommended for Phase 2 multi-protocol
- **Storage**: 100GB for PCAPs, grammars, Patternbook

### 5.2 Software Stack

#### Phase 1
```toml
[tool.poetry.dependencies]
python = "^3.11"
numpy = "^1.26"
scipy = "^1.12"
pandas = "^2.2"
scapy = "^2.5"
pyyaml = "^6.0"
```

#### Phase 2 Additions
```toml
langgraph = "^0.0.50"
langchain = "^0.1.0"
langchain-core = "^0.1.0"
qdrant-client = "^1.7"
sentence-transformers = "^2.3"
```

### 5.3 External Services

- **OpenHands**: Latest stable release
- **Docker**: 24+
- **Qdrant**: 1.7+ (Phase 2)
- **GitHub**: For CI/CD and version control

---

## 6. Risk Management

### 6.1 Technical Risks

| Risk | Phase | Impact | Mitigation |
|------|-------|--------|-----------|
| OpenHands integration complexity | 1 | High | Dedicated Week 2, fallback to direct Docker |
| LangGraph learning curve | 2 | Medium | Start with simple graphs, use examples |
| Qdrant performance | 2 | Medium | Benchmark early, optimize embeddings |
| Multi-protocol resource contention | 2 | Medium | Implement resource limits, monitoring |
| Hypothesis quality | 1-2 | Medium | Start with synthetic protocols, iterate |

### 6.2 Schedule Risks

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Phase 1 overruns | High | 2-week buffer, strict gate criteria |
| Dependencies unavailable | Medium | Lock versions, vendor critical deps |
| Testing takes longer | Low | Continuous testing from Week 1 |
| Documentation lag | Low | Document as you code |

### 6.3 Compliance Risks

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Constitutional violations | Critical | Mandatory code reviews, automated checks |
| Memory boundary leaks | High | Unit tests, runtime assertions |
| Missing evidence trails | Medium | Enforced logging, audit tools |

---

## 7. Quality Assurance

### 7.1 Code Review Checklist

For every pull request:

- [ ] Constitutional compliance verified (run `check_compliance.py`)
- [ ] Memory boundaries respected (Section 3)
- [ ] Evidence logging present (Section 2)
- [ ] Tests included (>80% coverage)
- [ ] Type hints added
- [ ] Docstrings complete
- [ ] Black formatting applied
- [ ] isort imports sorted
- [ ] mypy type checking passed
- [ ] ruff linting passed

### 7.2 Testing Standards

| Test Type | Coverage Target | Automation |
|-----------|----------------|------------|
| Unit tests | >80% | pytest in CI |
| Integration tests | Key workflows | pytest in CI |
| Constitutional compliance | 100% | Custom script in CI |
| Performance benchmarks | Baselines established | Weekly manual |
| End-to-end | 1 per phase | Manual + automated |

### 7.3 Documentation Standards

- Every module has docstring
- Every public function has docstring with type hints and examples
- Cookbook procedures have runnable examples
- README includes quickstart for both phases
- Architecture diagrams current

---

## 8. Milestone Schedule

| Week | Phase | Milestone | Gate Criteria |
|------|-------|-----------|---------------|
| 2 | 1 | Infrastructure Complete | Docker running, MCP functional, CI/CD operational |
| 3 | 1 | Memory System Complete | Base Scope tested, schemas validated |
| 4 | 1 | Observer Complete | Statistical analysis functional, >80% coverage |
| 5 | 1 | Theorist Complete | Hypothesis generation functional, >80% coverage |
| 6 | 1 | Validator Complete | Cross-PCAP validation functional, >80% coverage |
| 7 | 1 | Archivist Complete | Promotion workflow functional, >80% coverage |
| 8 | 1 | **PHASE 1 GATE** | One complete protocol analysis successful |
| 10 | 2 | LangGraph Integration | Autonomous workflow operational |
| 11 | 2 | Patternbook Complete | Vector similarity functional, Theorist integrated |
| 12 | 2 | Scope Adaptation | All Scopes work as LangGraph nodes |
| 13 | 2 | Multi-Protocol | 5 concurrent analyses functional |
| 14 | 2 | Optimization | Performance targets met |
| 15 | 2 | Advanced Features | Autonomous checkpoints, learning system |
| 16 | 2 | **PHASE 2 COMPLETE** | Full system operational, ready for release |

---

## 9. Success Criteria

### 9.1 Phase 1 (Week 8 Gate)

- [ ] OpenHands executes workflows reproducibly
- [ ] Observer generates accurate statistical measurements
- [ ] Theorist proposes falsifiable hypotheses
- [ ] Validator enforces 3-PCAP minimum
- [ ] Archivist promotes validated grammars
- [ ] One synthetic protocol successfully analyzed end-to-end
- [ ] Memory boundaries enforced (no violations)
- [ ] Constitutional compliance verified
- [ ] Test coverage >80%
- [ ] Documentation complete for Phase 1

### 9.2 Phase 2 (Week 16 Completion)

- [ ] LangGraph orchestrates autonomous workflow
- [ ] Patternbook provides useful analogies (measured)
- [ ] System handles 5 concurrent protocol analyses
- [ ] Performance targets met:
  - 1GB PCAP processed in <10 minutes
  - Hypothesis generation <2 minutes
  - Validation <5 minutes for 3 PCAPs
- [ ] At least 2 real-world protocols analyzed
- [ ] Autonomous checkpoints functional
- [ ] Learning from validation feedback demonstrated
- [ ] Constitutional principles maintained
- [ ] Test coverage >80% (including Phase 2 code)
- [ ] Complete documentation for both phases
- [ ] Open-source release prepared

---

## 10. Post-Phase 2 Activities

### 10.1 Production Readiness

1. **Security Audit**
   - Third-party security review
   - Penetration testing of sandboxing
   - Secrets management review

2. **Performance Tuning**
   - Optimize for larger PCAPs (>10GB)
   - Tune Patternbook embeddings
   - Optimize LangGraph state management

3. **Operational Tooling**
   - Monitoring dashboards
   - Alerting for failures
   - Automated health checks
   - Log aggregation

### 10.2 Future Enhancements (Not in 16-week plan)

- **Phase 3** (Potential): 
  - State machine visualization
  - Interactive hypothesis refinement UI
  - Distributed processing for 100+ protocols
  - Real-time traffic analysis (with caution)

- **Community**:
  - Contribution guidelines
  - Example protocol parsers
  - Tutorial videos
  - Conference presentations

- **Research**:
  - Academic paper on epistemic framework
  - Benchmark dataset publication
  - Cross-tool comparisons (Wireshark, Zeek)

---

## 11. Contingency Plans

### 11.1 Phase 1 Extension

If Phase 1 gate criteria not met at Week 8:

**Option A**: 1-week extension
- Focus on failing criteria only
- Re-gate at Week 9

**Option B**: 2-week extension
- Re-implement problematic Scope
- Re-gate at Week 10

**Option C**: Pivot
- Reassess feasibility
- Adjust Phase 2 scope

### 11.2 Phase 2 Scope Reduction

If Phase 2 timeline at risk:

**Priority 1 (Must Have)**:
- LangGraph orchestration
- Basic Patternbook

**Priority 2 (Should Have)**:
- Multi-protocol (reduce to 3 concurrent)
- Advanced Patternbook queries

**Priority 3 (Nice to Have)**:
- Autonomous checkpoints
- Learning system
- Clustering

---

## 12. Communication and Reporting

### 12.1 Weekly Status Reports

Every Friday:
- Completed tasks
- Blockers
- Risks
- Next week plan
- Gate criteria progress

### 12.2 Gate Reviews

Formal reviews at:
- Week 8 (Phase 1 gate)
- Week 16 (Phase 2 completion)

Review includes:
- Demo of functionality
- Test results
- Constitutional compliance audit
- Go/no-go decision

---

## 13. Appendices

### A. Quick Reference Commands

```bash
# Setup
poetry install
docker-compose -f docker/docker-compose.yml up -d

# Phase 1: Run analysis
poetry run python -m reshark.orchestration.analyze_protocol \
  --golden pcaps/golden/test.pcap \
  --validation pcaps/validation/*.pcap

# Phase 2: Run autonomous analysis
poetry run python -m reshark.orchestration.langgraph_analyze \
  --sessions configs/multi_protocol.yaml

# Testing
poetry run pytest tests/ -v --cov=reshark

# Compliance check
poetry run python scripts/check_compliance.py

# Code quality
poetry run black reshark/ tests/
poetry run isort reshark/ tests/
poetry run mypy reshark/
poetry run ruff check reshark/
```

### B. Constitutional Compliance Verification

Before each phase gate, verify:

```bash
# 1. Memory boundaries
poetry run pytest tests/test_constitution.py::test_memory_boundaries

# 2. Validation requirements
poetry run pytest tests/test_constitution.py::test_validation_requirements

# 3. Evidence logging
poetry run pytest tests/test_constitution.py::test_evidence_logging

# 4. Scope authority
poetry run pytest tests/test_constitution.py::test_scope_authority

# 5. Tool governance
poetry run pytest tests/test_constitution.py::test_tool_governance
```

### C. Performance Benchmarks

Target metrics:

| Metric | Phase 1 Target | Phase 2 Target |
|--------|---------------|---------------|
| PCAP processing (1GB) | <10 min | <10 min |
| Entropy calculation | <2 min | <1 min |
| Hypothesis generation | <2 min | <1 min |
| Validation (3 PCAPs) | <5 min | <3 min |
| Concurrent protocols | 1 | 5+ |
| Memory usage (per protocol) | <8GB | <4GB |

---

**Plan Status**: Draft  
**Version**: 2.0.0  
**Last Updated**: 2026-01-11  
**Approval Required**: Project Stakeholder + Constitutional Review  
**Next Review**: Week 4 (Phase 1 progress check)  
**Governed By**: reShark Constitution v2.0.0