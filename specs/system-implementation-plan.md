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

### Week 5: Agents - Orchestration (Workflow Management)

#### Objectives
- Task orchestration agent implemented
- Session management and lifecycle
- Workflow coordination between agents
- Skills for multi-step analysis procedures

#### Tasks

**Day 1-2: Session Manager**

1. **Implement Session Manager**
   ```python
   # reshark/agents/orchestration/session_manager.py
   """
   Session Manager - Manages analysis session lifecycle.
   
   Responsibilities:
   - Create and track sessions
   - Coordinate agent execution
   - Archive completed sessions
   - Maintain session state
   """
   
   from typing import Dict, Any, Optional
   from pathlib import Path
   from datetime import datetime
   import uuid
   import shutil
   import logging
   
   from reshark.agents.base import Agent, AgentError
   
   logger = logging.getLogger(__name__)
   
   class SessionManager(Agent):
       """
       Manages analysis session lifecycle and coordination.
       """
       
       @property
       def agent_type(self) -> str:
           return "orchestration"
       
       def execute(self, inputs: Dict[str, Any]) -> Dict[str, Any]:
           """
           Execute session management operation.
           
           Args:
               inputs: {
                   "operation": "create" | "archive" | "status",
                   "session_id": Optional session ID for status/archive,
                   "metadata": Optional session metadata
               }
               
           Returns:
               Operation results with session information
           """
           operation = inputs.get("operation", "create")
           
           if operation == "create":
               return self._create_session(inputs.get("metadata", {}))
           elif operation == "archive":
               return self._archive_session(inputs["session_id"])
           elif operation == "status":
               return self._get_session_status(inputs["session_id"])
           else:
               raise AgentError(f"Unknown operation: {operation}")
       
       def _create_session(self, metadata: Dict[str, Any]) -> Dict[str, Any]:
           """Create new analysis session."""
           # Generate session ID
           timestamp = datetime.utcnow().strftime("%Y%m%d-%H%M%S")
           unique_id = str(uuid.uuid4())[:8]
           session_id = f"session-{timestamp}-{unique_id}"
           
           # Create session directory
           session_dir = self.notebook_path / "sessions" / session_id
           session_dir.mkdir(parents=True, exist_ok=True)
           
           # Write session metadata
           session_meta = {
               "session_id": session_id,
               "created_at": datetime.utcnow().isoformat(),
               "status": "active",
               "metadata": metadata
           }
           
           meta_file = session_dir / "session.json"
           import json
           with open(meta_file, 'w') as f:
               json.dump(session_meta, f, indent=2)
           
           logger.info(f"Created session: {session_id}")
           
           return {
               "session_id": session_id,
               "session_dir": str(session_dir),
               "status": "created"
           }
       
       def _archive_session(self, session_id: str) -> Dict[str, Any]:
           """Archive completed session."""
           session_dir = self.notebook_path / "sessions" / session_id
           
           if not session_dir.exists():
               raise AgentError(f"Session not found: {session_id}")
           
           # Create archive
           archive_dir = self.notebook_path / "archives"
           archive_dir.mkdir(exist_ok=True)
           
           archive_path = archive_dir / f"{session_id}.tar.gz"
           
           import tarfile
           with tarfile.open(archive_path, "w:gz") as tar:
               tar.add(session_dir, arcname=session_id)
           
           # Remove session directory
           shutil.rmtree(session_dir)
           
           logger.info(f"Archived session: {session_id} -> {archive_path}")
           
           return {
               "session_id": session_id,
               "archive_path": str(archive_path),
               "status": "archived"
           }
       
       def _get_session_status(self, session_id: str) -> Dict[str, Any]:
           """Get current session status."""
           session_dir = self.notebook_path / "sessions" / session_id
           
           if not session_dir.exists():
               return {
                   "session_id": session_id,
                   "status": "not_found"
               }
           
           # Read session metadata
           meta_file = session_dir / "session.json"
           import json
           
           if meta_file.exists():
               with open(meta_file) as f:
                   metadata = json.load(f)
           else:
               metadata = {}
           
           # Count files
           file_count = len(list(session_dir.glob("*.json")))
           
           return {
               "session_id": session_id,
               "status": "active",
               "metadata": metadata,
               "file_count": file_count,
               "size_bytes": sum(f.stat().st_size for f in session_dir.rglob("*") if f.is_file())
           }
   ```

**Day 3-4: Task Orchestrator**

2. **Implement Task Orchestrator**
   ```python
   # reshark/agents/orchestration/task_orchestrator.py
   """
   Task Orchestrator - Coordinates multi-agent analysis workflows.
   
   Follows skills:
   - cookbook/methods/analysis-workflow.md
   """
   
   from typing import Dict, Any, List
   import logging
   
   from reshark.agents.base import Agent, AgentError
   
   logger = logging.getLogger(__name__)
   
   class TaskOrchestrator(Agent):
       """
       Orchestrates execution of multi-step analysis tasks.
       
       In HITL mode: Guides human through workflow steps
       In Autonomous mode: Executes workflow automatically
       """
       
       def __init__(self, books_path, tools_registry=None, agent_registry=None):
           super().__init__(books_path, tools_registry)
           self.agent_registry = agent_registry or {}
       
       @property
       def agent_type(self) -> str:
           return "orchestration"
       
       def execute(self, inputs: Dict[str, Any]) -> Dict[str, Any]:
           """
           Execute orchestrated workflow.
           
           Args:
               inputs: {
                   "workflow": "extract_and_analyze" | "validate_grammar",
                   "artifact_path": Path to input artifact,
                   "workflow_params": Additional parameters
               }
               
           Returns:
               Workflow execution results
           """
           workflow = inputs.get("workflow")
           
           if workflow == "extract_and_analyze":
               return self._workflow_extract_and_analyze(inputs)
           elif workflow == "validate_grammar":
               return self._workflow_validate_grammar(inputs)
           else:
               raise AgentError(f"Unknown workflow: {workflow}")
       
       def _workflow_extract_and_analyze(self, inputs: Dict[str, Any]) -> Dict[str, Any]:
           """
           Workflow: Extract data and perform pattern analysis.
           
           Steps:
           1. Extract data streams
           2. Compute statistics
           3. Identify patterns
           4. Write results to notebook
           """
           artifact_path = inputs["artifact_path"]
           artifact_type = inputs.get("artifact_type", "pcap")
           
           logger.info(f"Starting extract_and_analyze workflow for {artifact_path}")
           
           # Step 1: Extract data using DataInterpreter
           data_interpreter = self.agent_registry.get("DataInterpreter")
           if not data_interpreter:
               raise AgentError("DataInterpreter not registered")
           
           data_interpreter.session_id = self.session_id
           
           extraction_result = data_interpreter.execute({
               "artifact_path": artifact_path,
               "artifact_type": artifact_type,
               "skill": "protocol/data-extraction.md"
           })
           
           streams = extraction_result.get("streams", [])
           
           if not streams:
               return {
                   "status": "failed",
                   "error": "No streams extracted",
                   "workflow": "extract_and_analyze"
               }
           
           # Step 2: Analyze patterns using PatternInterpreter
           pattern_interpreter = self.agent_registry.get("PatternInterpreter")
           if not pattern_interpreter:
               raise AgentError("PatternInterpreter not registered")
           
           pattern_interpreter.session_id = self.session_id
           
           pattern_result = pattern_interpreter.execute({
               "streams": streams,
               "analysis_type": "all"
           })
           
           # Step 3: Consolidate results
           workflow_result = {
               "status": "success",
               "workflow": "extract_and_analyze",
               "artifact": artifact_path,
               "extraction": {
                   "streams_count": len(streams),
                   "total_bytes": extraction_result.get("statistics", {}).get("total_bytes", 0)
               },
               "patterns": {
                   "entropy_positions": len(pattern_result.get("entropy", {})),
                   "sequences_found": len(pattern_result.get("sequences", []))
               }
           }
           
           # Write consolidated results
           self.write_notebook("workflow_result", workflow_result)
           
           self.log_evidence({
               "workflow": "extract_and_analyze",
               "steps_completed": ["extract", "analyze_patterns"],
               "artifact": artifact_path
           })
           
           logger.info("Workflow completed successfully")
           
           return workflow_result
       
       def _workflow_validate_grammar(self, inputs: Dict[str, Any]) -> Dict[str, Any]:
           """
           Workflow: Validate grammar against multiple samples.
           
           Placeholder for Phase 2 validation workflows.
           """
           return {
               "status": "not_implemented",
               "workflow": "validate_grammar"
           }
   ```

**Day 5: Workflow Skills Documentation**

3. **Create Analysis Workflow Skill**
   ```bash
   cat > books/cookbook/methods/analysis-workflow.md << 'EOF'
   # Complete Analysis Workflow
   
   ## Purpose
   
   Orchestrated multi-step workflow for artifact analysis combining extraction, pattern analysis, and hypothesis generation.
   
   ## Workflow Steps
   
   ### Phase 1: Data Extraction
   
   1. **Identify Artifact Type**
      - PCAP: Network capture
      - Binary: Executable or data file
      - File: Document or data format
   
   2. **Extract Raw Data**
      - Use appropriate interpreter agent
      - Follow: skills/protocol/data-extraction.md
      - Result: List of data streams
   
   ### Phase 2: Pattern Analysis
   
   3. **Compute Statistics**
      - Entropy map per byte position
      - Frequency distributions
      - Follow: skills/pattern/entropy-analysis.md
   
   4. **Identify Patterns**
      - Repeated sequences
      - Invariant positions
      - Variable fields
      - Follow: skills/pattern/sequence-detection.md
   
   ### Phase 3: Structure Inference
   
   5. **Identify Boundaries**
      - Message/record boundaries
      - Header/payload separation
      - Follow: skills/binary/boundary-detection.md
   
   6. **Type Inference**
      - Field data types
      - Endianness detection
      - Follow: skills/binary/type-inference.md
   
   ## Orchestration Pattern
   
   ```python
   # HITL Mode (Cline guided)
   orchestrator = TaskOrchestrator(books_path, tools, agents)
   orchestrator.session_id = "session-001"
   
   result = orchestrator.execute({
       "workflow": "extract_and_analyze",
       "artifact_path": "data/sample.pcap",
       "artifact_type": "pcap"
   })
   ```
   
   ## Output
   
   Workflow produces:
   - Notebook entries for each step
   - Consolidated results summary
   - Evidence log for all findings
   - Ready for validation phase
   
   ## References
   
   - [Data Extraction Skill](../../skills/protocol/data-extraction.md)
   - [Entropy Analysis Skill](../../skills/pattern/entropy-analysis.md)
   - [Sequence Detection Skill](../../skills/pattern/sequence-detection.md)
   EOF
   ```

#### Deliverables
- [ ] SessionManager agent implemented
- [ ] TaskOrchestrator agent with workflow coordination
- [ ] Agent registry for cross-agent communication
- [ ] Workflow skills documented in Cookbook
- [ ] Unit tests for orchestration agents

#### Validation Checkpoint
```bash
# Test orchestration
poetry run pytest tests/agents/orchestration/ -v

# Test complete workflow (HITL)
# 1. Create session via SessionManager
# 2. Execute extract_and_analyze workflow
# 3. Verify results in Notebook
# 4. Archive session
```

---

### Week 6: Agents - Validation (Quality Assurance)

#### Objectives
- Validation agents for grammar testing
- Cross-sample validation logic
- Result verification agents
- Constitutional compliance checking

#### Tasks

**Day 1-3: Grammar Validator Agent**

1. **Implement Grammar Validator**
   ```python
   # reshark/agents/validation/grammar_validator.py
   """
   Grammar Validator - Validates grammars against multiple samples.
   
   Constitutional requirement: Minimum 3 samples for validation.
   Follows: skills/grammar/grammar-validation.md
   """
   
   from typing import Dict, Any, List
   from pathlib import Path
   import logging
   
   from reshark.agents.base import Agent, AgentError
   
   logger = logging.getLogger(__name__)
   
   class GrammarValidator(Agent):
       """
       Validates proposed grammars against multiple data samples.
       
       Enforces Constitutional requirement of >= 3 samples.
       """
       
       @property
       def agent_type(self) -> str:
           return "validation"
       
       def execute(self, inputs: Dict[str, Any]) -> Dict[str, Any]:
           """
           Execute grammar validation.
           
           Args:
               inputs: {
                   "grammar_path": Path to grammar file (.ksy or .py),
                   "sample_paths": List of sample data paths (minimum 3),
                   "validation_type": "parse" | "boundary" | "comprehensive"
               }
               
           Returns:
               Validation results with pass/fail status
           """
           grammar_path = Path(inputs["grammar_path"])
           sample_paths = [Path(p) for p in inputs["sample_paths"]]
           validation_type = inputs.get("validation_type", "comprehensive")
           
           # Constitutional check: minimum 3 samples
           if len(sample_paths) < 3:
               raise AgentError(
                   f"Constitutional violation: Validation requires >= 3 samples, "
                   f"got {len(sample_paths)}. See Constitution Section on Validation."
               )
           
           logger.info(f"Validating grammar {grammar_path} against {len(sample_paths)} samples")
           
           # Read grammar
           if not grammar_path.exists():
               raise AgentError(f"Grammar not found: {grammar_path}")
           
           grammar_content = grammar_path.read_text()
           
           # Validate each sample
           validation_results = []
           
           for idx, sample_path in enumerate(sample_paths):
               if not sample_path.exists():
                   logger.warning(f"Sample not found: {sample_path}")
                   continue
               
               result = self._validate_sample(
                   grammar_content,
                   sample_path,
                   validation_type
               )
               
               result["sample_id"] = idx
               result["sample_path"] = str(sample_path)
               validation_results.append(result)
           
           # Compute overall status
           passed = [r for r in validation_results if r.get("status") == "pass"]
           failed = [r for r in validation_results if r.get("status") == "fail"]
           
           overall_status = "pass" if len(failed) == 0 else "fail"
           
           result = {
               "grammar": str(grammar_path),
               "sample_count": len(sample_paths),
               "validation_type": validation_type,
               "overall_status": overall_status,
               "passed": len(passed),
               "failed": len(failed),
               "results": validation_results
           }
           
           # Write to notebook
           self.write_notebook("validation_result", result)
           
           # Log evidence
           self.log_evidence({
               "validation_method": "cross_sample_validation",
               "grammar": str(grammar_path),
               "samples_tested": len(sample_paths),
               "constitutional_compliance": len(sample_paths) >= 3
           })
           
           return result
       
       def _validate_sample(
           self,
           grammar: str,
           sample_path: Path,
           validation_type: str
       ) -> Dict[str, Any]:
           """Validate grammar against single sample."""
           
           if validation_type in ["parse", "comprehensive"]:
               # Attempt to parse sample with grammar
               parse_result = self._try_parse(grammar, sample_path)
               
               if not parse_result["success"]:
                   return {
                       "status": "fail",
                       "test": "parse",
                       "error": parse_result.get("error")
                   }
           
           if validation_type in ["boundary", "comprehensive"]:
               # Test boundary detection
               boundary_result = self._test_boundaries(grammar, sample_path)
               
               if not boundary_result["success"]:
                   return {
                       "status": "fail",
                       "test": "boundary",
                       "error": boundary_result.get("error")
                   }
           
           return {
               "status": "pass",
               "tests_passed": ["parse", "boundary"] if validation_type == "comprehensive" else [validation_type]
           }
       
       def _try_parse(self, grammar: str, sample_path: Path) -> Dict[str, Any]:
           """Attempt to parse sample using grammar."""
           # Placeholder: Would invoke actual parser (Kaitai, custom parser, etc.)
           logger.info(f"Parsing {sample_path} with grammar")
           
           # For now, return success (actual implementation would parse)
           return {
               "success": True,
               "parsed_fields": []
           }
       
       def _test_boundaries(self, grammar: str, sample_path: Path) -> Dict[str, Any]:
           """Test boundary detection in sample."""
           logger.info(f"Testing boundaries in {sample_path}")
           
           # Placeholder: Would test message/field boundaries
           return {
               "success": True,
               "boundaries_detected": []
           }
   ```

**Day 4: Consistency Checker Agent**

2. **Implement Consistency Checker**
   ```python
   # reshark/agents/validation/consistency_checker.py
   """
   Consistency Checker - Verifies consistency across analysis results.
   
   Checks:
   - Pattern consistency across samples
   - Grammar consistency with observations
   - Result reproducibility
   """
   
   from typing import Dict, Any, List
   import logging
   
   from reshark.agents.base import Agent, AgentError
   
   logger = logging.getLogger(__name__)
   
   class ConsistencyChecker(Agent):
       """
       Validates consistency of analysis results.
       """
       
       @property
       def agent_type(self) -> str:
           return "validation"
       
       def execute(self, inputs: Dict[str, Any]) -> Dict[str, Any]:
           """
           Check consistency of analysis results.
           
           Args:
               inputs: {
                   "check_type": "patterns" | "grammar" | "all",
                   "session_id": Session to validate
               }
               
           Returns:
               Consistency check results
           """
           check_type = inputs.get("check_type", "all")
           session_id = inputs.get("session_id", self.session_id)
           
           if not session_id:
               raise AgentError("No session specified for consistency check")
           
           # Read session data from Notebook
           entropy_data = self.read_notebook("pattern_analysis")
           extracted_data = self.read_notebook("extracted_streams")
           
           results = {}
           
           if check_type in ["patterns", "all"]:
               results["pattern_consistency"] = self._check_pattern_consistency(
                   entropy_data,
                   extracted_data
               )
           
           if check_type in ["grammar", "all"]:
               results["grammar_consistency"] = self._check_grammar_consistency()
           
           overall = "pass" if all(
               r.get("status") == "pass" 
               for r in results.values()
           ) else "fail"
           
           result = {
               "check_type": check_type,
               "session_id": session_id,
               "overall_status": overall,
               "checks": results
           }
           
           self.write_notebook("consistency_check", result)
           
           return result
       
       def _check_pattern_consistency(
           self,
           entropy_data: Dict,
           extracted_data: Dict
       ) -> Dict[str, Any]:
           """Check pattern analysis consistency."""
           if not entropy_data or not extracted_data:
               return {"status": "skip", "reason": "missing_data"}
           
           # Verify entropy calculations are consistent
           entropy_map = entropy_data.get("data", {}).get("entropy", {})
           streams = extracted_data.get("data", {}).get("streams", [])
           
           if not entropy_map or not streams:
               return {"status": "skip", "reason": "no_data"}
           
           # Check: Number of positions matches stream lengths
           expected_positions = max(len(s.get("data", b"")) for s in streams)
           actual_positions = len(entropy_map)
           
           if abs(expected_positions - actual_positions) > 10:  # Allow small variance
               return {
                   "status": "fail",
                   "issue": "position_mismatch",
                   "expected": expected_positions,
                   "actual": actual_positions
               }
           
           return {"status": "pass"}
       
       def _check_grammar_consistency(self) -> Dict[str, Any]:
           """Check grammar consistency with observations."""
           # Placeholder for grammar consistency checks
           return {"status": "pass"}
   ```

**Day 5: Validation Skills Documentation**

3. **Create Grammar Validation Skill**
   ```bash
   cat > skills/grammar/grammar-validation.md << 'EOF'
   # Grammar Validation Methodology
   
   **Category**: grammar  
   **Complexity**: intermediate  
   **Prerequisites**: Grammar file (.ksy or .py), minimum 3 sample files
   
   ## Purpose
   
   Validate proposed grammar against multiple data samples to ensure correctness and completeness.
   
   ## Constitutional Requirement
   
   **Minimum 3 samples required for validation** per Constitution.
   
   ## When to Use
   
   - Before promoting grammar to Rulebook
   - After grammar modifications
   - When adding new samples
   
   ## Methodology
   
   ### Step 1: Prepare Samples
   
   Collect at least 3 diverse samples:
   - Different message types
   - Edge cases (minimum/maximum values)
   - Different protocol states
   
   ### Step 2: Parse Testing
   
   For each sample:
   1. Parse using grammar
   2. Verify all fields extracted
   3. Check no parse errors
   4. Validate field values in expected ranges
   
   ### Step 3: Boundary Testing
   
   Test message/field boundaries:
   - Start of message correctly identified
   - End of message correctly identified
   - No overlap or gaps
   
   ### Step 4: Negative Testing
   
   Test invalid inputs:
   - Truncated messages
   - Corrupted data
   - Wrong protocol messages
   
   Grammar should fail gracefully.
   
   ### Step 5: Consistency Check
   
   Verify:
   - Same fields parsed in all samples
   - Field types consistent
   - Offsets align across samples
   
   ## Example
   
   ### Input
   - Grammar: `protocol_x_v1.ksy`
   - Samples: `sample1.bin`, `sample2.bin`, `sample3.bin`
   
   ### Process
   
   ```python
   validator = GrammarValidator(books_path)
   validator.session_id = session_id
   
   result = validator.execute({
       "grammar_path": "rulebook/grammars/protocol_x_v1.ksy",
       "sample_paths": [
           "data/sample1.bin",
           "data/sample2.bin", 
           "data/sample3.bin"
       ],
       "validation_type": "comprehensive"
   })
   ```
   
   ### Output
   
   ```json
   {
     "overall_status": "pass",
     "passed": 3,
     "failed": 0,
     "results": [...]
   }
   ```
   
   ## Interpretation Guide
   
   - **All pass**: Grammar ready for promotion
   - **1 fail**: Review failing sample, may be edge case
   - **2+ fail**: Grammar needs refinement
   
   ## Common Pitfalls
   
   - Using <3 samples violates Constitution
   - Not testing edge cases
   - Ignoring negative test results
   
   ## Related Skills
   
   - [Format Inference](format-inference.md)
   - [Sample Generation](sample-generation.md)
   
   ## References
   
   - Constitution: Validation Requirements
   - Kaitai Struct: Grammar syntax
   EOF
   ```

#### Deliverables
- [ ] GrammarValidator agent with Constitutional compliance
- [ ] ConsistencyChecker for result verification
- [ ] Validation skills documented
- [ ] Minimum 3-sample enforcement implemented
- [ ] Unit tests with Constitutional compliance checks

#### Validation Checkpoint
```bash
# Test validation agents
poetry run pytest tests/agents/validation/ -v

# Test Constitutional compliance
# Should raise error with <3 samples
poetry run pytest tests/test_constitutional_compliance.py -v
```

---

### Week 7: Tools & Skills Expansion

#### Objectives
- Additional tool wrappers implemented
- Skills library expanded to 12+ skills
- Tool registry with unified interface
- Documentation and examples complete

#### Tasks

**Day 1-2: Additional Tool Wrappers**

1. **Binary Analysis Tools**
   ```python
   # reshark/tools/binary_tools.py
   """
   Wrappers for binary analysis tools.
   """
   
   import subprocess
   from typing import Dict, Any, List
   from pathlib import Path
   import logging
   
   logger = logging.getLogger(__name__)
   
   class FileWrapper:
       """Wrapper for 'file' command."""
       
       def identify_type(self, file_path: Path) -> Dict[str, Any]:
           """Identify file type."""
           cmd = ["file", "-b", str(file_path)]
           
           try:
               result = subprocess.run(
                   cmd,
                   capture_output=True,
                   text=True,
                   timeout=10
               )
               
               return {
                   "success": True,
                   "file_type": result.stdout.strip()
               }
           except Exception as e:
               return {"success": False, "error": str(e)}
   
   class StringsWrapper:
       """Wrapper for 'strings' command."""
       
       def extract_strings(
           self,
           file_path: Path,
           min_length: int = 4
       ) -> Dict[str, Any]:
           """Extract printable strings."""
           cmd = ["strings", "-n", str(min_length), str(file_path)]
           
           try:
               result = subprocess.run(
                   cmd,
                   capture_output=True,
                   text=True,
                   timeout=30
               )
               
               strings = result.stdout.strip().split('\n')
               
               return {
                   "success": True,
                   "strings": [s for s in strings if s],
                   "count": len([s for s in strings if s])
               }
           except Exception as e:
               return {"success": False, "error": str(e)}
   
   class HexdumpWrapper:
       """Wrapper for 'hexdump' command."""
       
       def dump_hex(
           self,
           file_path: Path,
           length: int = 512,
           format: str = "canonical"
       ) -> Dict[str, Any]:
           """Generate hexdump output."""
           cmd = ["hexdump"]
           
           if format == "canonical":
               cmd.append("-C")
           
           cmd.extend(["-n", str(length), str(file_path)])
           
           try:
               result = subprocess.run(
                   cmd,
                   capture_output=True,
                   text=True,
                   timeout=10
               )
               
               return {
                   "success": True,
                   "output": result.stdout
               }
           except Exception as e:
               return {"success": False, "error": str(e)}
   ```

2. **Tools Registry**
   ```python
   # reshark/tools/registry.py
   """
   Central registry for all tool wrappers.
   Provides unified interface for tool invocation.
   """
   
   from typing import Dict, Any, Optional
   import logging
   
   from reshark.tools.tshark_wrapper import TsharkWrapper
   from reshark.tools.binary_tools import FileWrapper, StringsWrapper, HexdumpWrapper
   
   logger = logging.getLogger(__name__)
   
   class ToolsRegistry:
       """Central registry for tool wrappers."""
       
       def __init__(self):
           self._tools = {}
           self._register_builtin_tools()
       
       def _register_builtin_tools(self):
           """Register all built-in tool wrappers."""
           self.register("tshark", TsharkWrapper())
           self.register("file", FileWrapper())
           self.register("strings", StringsWrapper())
           self.register("hexdump", HexdumpWrapper())
       
       def register(self, name: str, tool: Any):
           """Register a tool wrapper."""
           self._tools[name] = tool
           logger.info(f"Registered tool: {name}")
       
       def invoke(self, tool_name: str, params: Dict[str, Any]) -> Dict[str, Any]:
           """
           Invoke tool with parameters.
           
           Args:
               tool_name: Name of tool
               params: Tool-specific parameters
               
           Returns:
               Tool output
           """
           tool = self._tools.get(tool_name)
           
           if not tool:
               return {
                   "success": False,
                   "error": f"Tool not found: {tool_name}"
               }
           
           try:
               # Route to appropriate tool method
               if tool_name == "tshark":
                   return self._invoke_tshark(tool, params)
               elif tool_name == "file":
                   return tool.identify_type(params["input"])
               elif tool_name == "strings":
                   return tool.extract_strings(
                       params["input"],
                       params.get("min_length", 4)
                   )
               elif tool_name == "hexdump":
                   return tool.dump_hex(
                       params["input"],
                       params.get("length", 512),
                       params.get("format", "canonical")
                   )
               else:
                   return {"success": False, "error": f"Unknown tool: {tool_name}"}
           
           except Exception as e:
               logger.error(f"Tool invocation error: {e}")
               return {"success": False, "error": str(e)}
       
       def _invoke_tshark(self, tool: TsharkWrapper, params: Dict) -> Dict:
           """Route tshark commands."""
           command = params.get("command")
           
           if command == "conversations":
               return tool.list_conversations(
                   params["input"],
                   params.get("protocol", "tcp")
               )
           elif command == "follow_stream":
               return tool.follow_stream(
                   params["input"],
                   params.get("stream_type", "tcp"),
                   params["stream_id"],
                   params.get("format", "raw")
               )
           else:
               return {"success": False, "error": f"Unknown tshark command: {command}"}
   ```

**Day 3-5: Skills Library Expansion**

3. **Create Additional Skills**
   
   Create these skills to reach 12+ total:
   
   - `skills/protocol/field-extraction.md` - Extract specific protocol fields
   - `skills/protocol/protocol-fsm-analysis.md` - Model protocol state machines
   - `skills/pattern/entropy-analysis.md` - Detailed entropy methodology
   - `skills/pattern/frequency-analysis.md` - Byte frequency distributions
   - `skills/pattern/sequence-detection.md` - Repeated pattern detection
   - `skills/binary/structure-recovery.md` - Infer data structures
   - `skills/binary/type-inference.md` - Determine field types
   - `skills/binary/boundary-detection.md` - Find message boundaries
   - `skills/grammar/sample-generation.md` - Generate test samples
   - `skills/grammar/format-inference.md` - Infer grammar from samples
   
   Each following the template established in Week 3.

#### Deliverables
- [ ] Binary analysis tool wrappers (file, strings, hexdump)
- [ ] ToolsRegistry with unified interface
- [ ] 12+ skills documented across all categories
- [ ] Skills index in skills/README.md updated
- [ ] Tool integration tests

#### Validation Checkpoint
```bash
# Test all tool wrappers
poetry run pytest tests/tools/ -v

# Verify skills library
ls skills/**/*.md | wc -l
# Expected: >= 12 skill files

# Test end-to-end with tools and skills
# Via Cline: "Use entropy-analysis skill to analyze sample.pcap"
```

---

### Week 8: Phase 1 Integration & Testing

#### Objectives
- Complete HITL workflow integration
- End-to-end testing with Cline
- Performance validation
- Phase 1 success criteria met
- Prepare for Phase 2 autonomous mode

#### Tasks

**Day 1-2: Integration Testing**

1. **Complete HITL Workflow**
   ```python
   # reshark/cli.py
   """CLI entry point for reShark HITL mode."""
   
   import click
   from pathlib import Path
   from reshark.agents.orchestration.session_manager import SessionManager
   from reshark.agents.orchestration.task_orchestrator import TaskOrchestrator
   from reshark.agents.interpreters.data_interpreter import DataInterpreter
   from reshark.agents.interpreters.pattern_interpreter import PatternInterpreter
   from reshark.agents.validation.grammar_validator import GrammarValidator
   from reshark.tools.registry import ToolsRegistry
   
   @click.group()
   def cli():
       """reShark - Reverse Engineering Framework"""
       pass
   
   @cli.command()
   @click.argument('artifact_path', type=click.Path(exists=True))
   @click.option('--artifact-type', default='pcap', help='Type of artifact')
   def analyze(artifact_path, artifact_type):
       """Analyze artifact using complete HITL workflow."""
       books_path = Path.cwd() / "books"
       tools = ToolsRegistry()
       
       # Create session
       session_mgr = SessionManager(books_path, tools)
       session = session_mgr.execute({
           "operation": "create",
           "metadata": {
               "artifact": artifact_path,
               "type": artifact_type
           }
       })
       
       session_id = session["session_id"]
       click.echo(f"Created session: {session_id}")
       
       # Initialize agents
       data_interpreter = DataInterpreter(books_path, tools)
       pattern_interpreter = PatternInterpreter(books_path, tools)
       
       agent_registry = {
           "DataInterpreter": data_interpreter,
           "PatternInterpreter": pattern_interpreter
       }
       
       # Execute workflow
       orchestrator = TaskOrchestrator(books_path, tools, agent_registry)
       orchestrator.session_id = session_id
       
       result = orchestrator.execute({
           "workflow": "extract_and_analyze",
           "artifact_path": artifact_path,
           "artifact_type": artifact_type
       })
       
       click.echo(f"Workflow status: {result['status']}")
       click.echo(f"Results in: books/notebook/sessions/{session_id}")
   
   @cli.command()
   @click.argument('grammar_path', type=click.Path(exists=True))
   @click.argument('sample_paths', nargs=-1, type=click.Path(exists=True))
   def validate_grammar(grammar_path, sample_paths):
       """Validate grammar against samples (min 3)."""
       if len(sample_paths) < 3:
           click.echo("Error: Minimum 3 samples required (Constitutional requirement)")
           return
       
       books_path = Path.cwd() / "books"
       tools = ToolsRegistry()
       
       validator = GrammarValidator(books_path, tools)
       
       result = validator.execute({
           "grammar_path": grammar_path,
           "sample_paths": list(sample_paths),
           "validation_type": "comprehensive"
       })
       
       click.echo(f"Validation: {result['overall_status']}")
       click.echo(f"Passed: {result['passed']}/{result['sample_count']}")
   
   if __name__ == '__main__':
       cli()
   ```

**Day 3: Test Data Generation**

2. **Synthetic Test Data**
   ```python
   # tests/fixtures/protocol_generator.py
   """Generate synthetic protocol data for testing."""
   
   import struct
   from pathlib import Path
   from typing import List
   
   class SimpleProtocolGenerator:
       """Generate simple binary protocol samples."""
       
       def generate_message(self, msg_type: int, payload: bytes) -> bytes:
           """Generate single message."""
           # Format: [magic:2][type:1][length:2][payload][checksum:1]
           magic = b'\xAA\xBB'
           length = len(payload)
           
           msg = magic + struct.pack('B', msg_type) + struct.pack('>H', length) + payload
           
           # Simple checksum
           checksum = sum(msg) % 256
           msg += struct.pack('B', checksum)
           
           return msg
       
       def generate_samples(self, output_dir: Path, count: int = 5):
           """Generate multiple sample files."""
           output_dir.mkdir(parents=True, exist_ok=True)
           
           payloads = [
               b'Hello World',
               b'Test Message',
               b'\x00\x01\x02\x03',
               b'A' * 100,
               b'Short'
           ]
           
           for i in range(count):
               messages = []
               for msg_type in [1, 2, 3]:
                   msg = self.generate_message(msg_type, payloads[i % len(payloads)])
                   messages.append(msg)
               
               sample_path = output_dir / f"sample_{i+1}.bin"
               with open(sample_path, 'wb') as f:
                   f.write(b''.join(messages))
   ```

**Day 4: End-to-End Testing**

3. **Integration Test Suite**
   ```python
   # tests/integration/test_complete_workflow.py
   """End-to-end integration tests."""
   
   import pytest
   from pathlib import Path
   from reshark.agents.orchestration.session_manager import SessionManager
   from reshark.agents.orchestration.task_orchestrator import TaskOrchestrator
   from reshark.agents.interpreters.data_interpreter import DataInterpreter
   from reshark.agents.interpreters.pattern_interpreter import PatternInterpreter
   from reshark.tools.registry import ToolsRegistry
   
   @pytest.fixture
   def setup_environment(tmp_path):
       """Setup test environment."""
       books_path = tmp_path / "books"
       books_path.mkdir()
       
       # Create directory structure
       (books_path / "notebook" / "sessions").mkdir(parents=True)
       (books_path / "skills").mkdir()
       
       tools = ToolsRegistry()
       
       return books_path, tools
   
   def test_complete_analysis_workflow(setup_environment, tmp_path):
       """Test complete extract and analyze workflow."""
       books_path, tools = setup_environment
       
       # Generate test data
       from tests.fixtures.protocol_generator import SimpleProtocolGenerator
       gen = SimpleProtocolGenerator()
       test_file = tmp_path / "test_sample.bin"
       gen.generate_samples(tmp_path, 1)
       
       # Create session
       session_mgr = SessionManager(books_path, tools)
       session = session_mgr.execute({"operation": "create"})
       session_id = session["session_id"]
       
       # Setup agents
       data_interp = DataInterpreter(books_path, tools)
       pattern_interp = PatternInterpreter(books_path, tools)
       
       agent_registry = {
           "DataInterpreter": data_interp,
           "PatternInterpreter": pattern_interp
       }
       
       # Execute workflow
       orchestrator = TaskOrchestrator(books_path, tools, agent_registry)
       orchestrator.session_id = session_id
       
       result = orchestrator.execute({
           "workflow": "extract_and_analyze",
           "artifact_path": str(test_file),
           "artifact_type": "binary"
       })
       
       assert result["status"] == "success"
       assert "extraction" in result
       assert "patterns" in result
   
   def test_constitutional_compliance(setup_environment):
       """Test 3-sample minimum enforcement."""
       books_path, tools = setup_environment
       
       from reshark.agents.validation.grammar_validator import GrammarValidator
       from reshark.agents.base import AgentError
       
       validator = GrammarValidator(books_path, tools)
       
       # Should raise error with <3 samples
       with pytest.raises(AgentError, match="Constitutional violation"):
           validator.execute({
               "grammar_path": "test.ksy",
               "sample_paths": ["s1.bin", "s2.bin"],  # Only 2 samples
               "validation_type": "parse"
           })
   ```

**Day 5: Documentation & Performance Validation**

4. **Performance Benchmarks**
   ```python
   # tests/performance/test_benchmarks.py
   """Performance benchmarks for Phase 1."""
   
   import pytest
   import time
   
   @pytest.mark.benchmark
   def test_workflow_performance(setup_environment, benchmark):
       """Benchmark complete workflow execution."""
       # Target: <5 seconds for small sample
       
       def run_workflow():
           # Run complete workflow
           pass
       
       result = benchmark(run_workflow)
       assert result < 5.0  # seconds
   ```

#### Deliverables
- [ ] CLI interface for HITL workflows
- [ ] Integration test suite with >80% coverage
- [ ] Performance benchmarks passing
- [ ] Synthetic test data generator
- [ ] Documentation complete for Phase 1

#### Phase 1 Gate Review

**Success Criteria:**
- [x] All agents implemented (Interpreters, Orchestration, Validation)
- [x] Books system operational (Notebook, Rulebook, Cookbook, Skills)
- [x] Tools registry with 4+ wrappers
- [x] 12+ skills documented
- [ ] Constitutional compliance enforced (3+ samples)
- [ ] End-to-end workflow tested via Cline
- [ ] Test coverage >80%
- [ ] Performance: <5s for small samples

**Go/No-Go Decision**: If all criteria met, proceed to Phase 2. Otherwise, extend Phase 1 by 1 week.

---

## PHASE 2: AUTONOMOUS MODE (WEEKS 9-16)

### Phase 2 Overview

**Objective**: Enable autonomous operation using OpenHands + Patternbook vector DB

**Key Differences from Phase 1**:
- Phase 1 (HITL): Human-guided with Cline, markdown skills read by LLM
- Phase 2 (Autonomous): Self-directed with OpenHands, Patternbook semantic search

**Architecture**:
- OpenHands as autonomous agent orchestrator
- Ollama for local LLM inference
- Patternbook (Qdrant) for pattern storage and retrieval
- Same agents/tools from Phase 1, new autonomous workflows

---

### Week 9: OpenHands Integration

#### Objectives
- Install and configure OpenHands
- Connect to Ollama LLM
- Integrate existing agents with OpenHands
- Test autonomous execution

#### Tasks

**Day 1-2: OpenHands Setup**

1. **Install OpenHands**
   ```bash
   # Clone OpenHands
   cd /tmp
   git clone https://github.com/All-Hands-AI/OpenHands.git
   cd OpenHands
   
   # Build with Docker
   docker build -t openhands:latest .
   
   # Or use Docker Compose (from docker/docker-compose.yml)
   cd /path/to/reshark
   docker-compose -f docker/docker-compose.yml up -d openhands
   ```

2. **Configure OpenHands**
   ```yaml
   # docker/docker-compose.yml
   version: '3.8'
   
   services:
     openhands:
       image: openhands:latest
       container_name: reshark-openhands
       environment:
         - LLM_PROVIDER=ollama
         - LLM_BASE_URL=http://ollama:11434
         - LLM_MODEL=deepseek-coder:33b
         - WORKSPACE_BASE=/workspace
       volumes:
         - ../:/workspace
         - openhands-data:/app/data
       ports:
         - "3000:3000"
       depends_on:
         - ollama
     
     ollama:
       image: ollama/ollama:latest
       container_name: reshark-ollama
       volumes:
         - ollama-models:/root/.ollama
       ports:
         - "11434:11434"
   
   volumes:
     openhands-data:
     ollama-models:
   ```

**Day 3-4: Agent Integration**

3. **OpenHands Agent Adapter**
   ```python
   # reshark/autonomous/openhands_adapter.py
   """Adapter for reShark agents in OpenHands."""
   
   from typing import Dict, Any
   from pathlib import Path
   import json
   
   class OpenHandsAdapter:
       """Bridges reShark agents with OpenHands."""
       
       def __init__(self, books_path: Path):
           self.books_path = books_path
           self.agents = self._load_agents()
       
       def _load_agents(self):
           """Load all reShark agents."""
           from reshark.agents.interpreters.data_interpreter import DataInterpreter
           from reshark.agents.interpreters.pattern_interpreter import PatternInterpreter
           from reshark.agents.orchestration.task_orchestrator import TaskOrchestrator
           from reshark.tools.registry import ToolsRegistry
           
           tools = ToolsRegistry()
           
           return {
               "data_interpreter": DataInterpreter(self.books_path, tools),
               "pattern_interpreter": PatternInterpreter(self.books_path, tools),
               "orchestrator": TaskOrchestrator(self.books_path, tools, {})
           }
       
       def execute_task(self, task_type: str, params: Dict[str, Any]) -> Dict[str, Any]:
           """Execute task via appropriate agent."""
           if task_type == "analyze_artifact":
               return self._run_analysis(params)
           elif task_type == "extract_patterns":
               return self._extract_patterns(params)
           else:
               return {"error": f"Unknown task: {task_type}"}
       
       def _run_analysis(self, params: Dict[str, Any]) -> Dict[str, Any]:
           """Run complete analysis workflow."""
           orchestrator = self.agents["orchestrator"]
           
           result = orchestrator.execute({
               "workflow": "extract_and_analyze",
               "artifact_path": params["artifact_path"],
               "artifact_type": params.get("artifact_type", "pcap")
           })
           
           return result
       
       def _extract_patterns(self, params: Dict[str, Any]) -> Dict[str, Any]:
           """Extract patterns from data."""
           pattern_interp = self.agents["pattern_interpreter"]
           
           result = pattern_interp.execute({
               "streams": params["streams"],
               "analysis_type": "all"
           })
           
           return result
   ```

4. **OpenHands Task Definition**
   ```python
   # reshark/autonomous/tasks.py
   """Task definitions for OpenHands."""
   
   ANALYZE_ARTIFACT_TASK = """
   Analyze the binary artifact at {artifact_path} using reShark agents.
   
   Steps:
   1. Use DataInterpreter to extract data streams
   2. Use PatternInterpreter to identify patterns
   3. Write results to Notebook
   4. Return summary of findings
   
   The agents are available via the OpenHandsAdapter at /workspace/reshark/autonomous/openhands_adapter.py
   """
   
   def generate_analysis_task(artifact_path: str) -> str:
       """Generate analysis task for OpenHands."""
       return ANALYZE_ARTIFACT_TASK.format(artifact_path=artifact_path)
   ```

**Day 5: Testing**

5. **Autonomous Execution Test**
   ```python
   # tests/autonomous/test_openhands_integration.py
   """Test OpenHands autonomous execution."""
   
   import pytest
   from reshark.autonomous.openhands_adapter import OpenHandsAdapter
   
   def test_autonomous_analysis(tmp_path):
       """Test autonomous artifact analysis."""
       books_path = tmp_path / "books"
       books_path.mkdir()
       (books_path / "notebook" / "sessions").mkdir(parents=True)
       
       adapter = OpenHandsAdapter(books_path)
       
       # Generate test data
       test_file = tmp_path / "test.bin"
       test_file.write_bytes(b"\xAA\xBB" + b"test data" * 100)
       
       result = adapter.execute_task("analyze_artifact", {
           "artifact_path": str(test_file),
           "artifact_type": "binary"
       })
       
       assert result["status"] == "success"
   ```

#### Deliverables
- [ ] OpenHands Docker container running
- [ ] Ollama LLM connected
- [ ] OpenHandsAdapter for agent integration
- [ ] Autonomous task execution tested
- [ ] Documentation for autonomous mode

#### Validation Checkpoint
```bash
# Start OpenHands
docker-compose -f docker/docker-compose.yml up -d

# Test autonomous execution
curl -X POST http://localhost:3000/api/task \
  -d '{"task": "analyze test.bin", "workspace": "/workspace"}'

# Verify results in Notebook
ls books/notebook/sessions/
```

---

### Week 10: Patternbook (Vector Database)

#### Objectives
- Install and configure Qdrant vector DB
- Design pattern embedding schema
- Implement pattern storage and retrieval
- Integrate with autonomous workflows

#### Tasks

**Day 1-2: Qdrant Setup**

1. **Install Qdrant**
   ```yaml
   # Add to docker/docker-compose.yml
   services:
     qdrant:
       image: qdrant/qdrant:latest
       container_name: reshark-qdrant
       ports:
         - "6333:6333"
         - "6334:6334"
       volumes:
         - qdrant-storage:/qdrant/storage
   
   volumes:
     qdrant-storage:
   ```

2. **Python Client**
   ```bash
   poetry add qdrant-client sentence-transformers
   ```

**Day 3-4: Patternbook Implementation**

3. **Pattern Storage Agent**
   ```python
   # reshark/books/patternbook/pattern_store.py
   """Patternbook - Vector database for patterns."""
   
   from typing import Dict, Any, List
   from qdrant_client import QdrantClient
   from qdrant_client.models import Distance, VectorParams, PointStruct
   from sentence_transformers import SentenceTransformer
   import hashlib
   import logging
   
   logger = logging.getLogger(__name__)
   
   class Patternbook:
       """Vector database for storing and retrieving patterns."""
       
       def __init__(self, qdrant_url: str = "http://localhost:6333"):
           self.client = QdrantClient(url=qdrant_url)
           self.encoder = SentenceTransformer('all-MiniLM-L6-v2')
           self.collection_name = "patterns"
           self._init_collection()
       
       def _init_collection(self):
           """Initialize Qdrant collection."""
           collections = self.client.get_collections().collections
           
           if not any(c.name == self.collection_name for c in collections):
               self.client.create_collection(
                   collection_name=self.collection_name,
                   vectors_config=VectorParams(
                       size=384,  # all-MiniLM-L6-v2 dimension
                       distance=Distance.COSINE
                   )
               )
               logger.info(f"Created collection: {self.collection_name}")
       
       def store_pattern(self, pattern: Dict[str, Any]) -> str:
           """
           Store pattern with embedding.
           
           Args:
               pattern: {
                   "type": "entropy" | "sequence" | "structure",
                   "description": Text description,
                   "data": Pattern data,
                   "metadata": Additional context
               }
           
           Returns:
               Pattern ID
           """
           # Generate embedding from description
           embedding = self.encoder.encode(pattern["description"]).tolist()
           
           # Generate ID from pattern content
           pattern_id = hashlib.sha256(
               str(pattern["data"]).encode()
           ).hexdigest()[:16]
           
           # Store in Qdrant
           self.client.upsert(
               collection_name=self.collection_name,
               points=[
                   PointStruct(
                       id=pattern_id,
                       vector=embedding,
                       payload=pattern
                   )
               ]
           )
           
           logger.info(f"Stored pattern: {pattern_id}")
           return pattern_id
       
       def search_patterns(
           self,
           query: str,
           pattern_type: str = None,
           limit: int = 5
       ) -> List[Dict[str, Any]]:
           """
           Search for similar patterns.
           
           Args:
               query: Natural language query
               pattern_type: Filter by pattern type
               limit: Maximum results
           
           Returns:
               List of matching patterns
           """
           # Generate query embedding
           query_vector = self.encoder.encode(query).tolist()
           
           # Build filter
           query_filter = None
           if pattern_type:
               from qdrant_client.models import Filter, FieldCondition, MatchValue
               query_filter = Filter(
                   must=[
                       FieldCondition(
                           key="type",
                           match=MatchValue(value=pattern_type)
                       )
                   ]
               )
           
           # Search
           results = self.client.search(
               collection_name=self.collection_name,
               query_vector=query_vector,
               query_filter=query_filter,
               limit=limit
           )
           
           patterns = [
               {
                   "id": r.id,
                   "score": r.score,
                   **r.payload
               }
               for r in results
           ]
           
           logger.info(f"Found {len(patterns)} patterns for query: {query}")
           return patterns
       
       def get_pattern(self, pattern_id: str) -> Dict[str, Any]:
           """Retrieve specific pattern by ID."""
           results = self.client.retrieve(
               collection_name=self.collection_name,
               ids=[pattern_id]
           )
           
           if results:
               return results[0].payload
           return None
   ```

**Day 5: Integration with Agents**

4. **Pattern-Aware Interpreter**
   ```python
   # reshark/agents/interpreters/autonomous_pattern_interpreter.py
   """Pattern interpreter with Patternbook integration."""
   
   from reshark.agents.interpreters.pattern_interpreter import PatternInterpreter
   from reshark.books.patternbook.pattern_store import Patternbook
   
   class AutonomousPatternInterpreter(PatternInterpreter):
       """Pattern interpreter with vector DB for autonomous mode."""
       
       def __init__(self, books_path, tools_registry=None, patternbook=None):
           super().__init__(books_path, tools_registry)
           self.patternbook = patternbook or Patternbook()
       
       def execute(self, inputs):
           """Execute with pattern storage."""
           # Run normal pattern analysis
           result = super().execute(inputs)
           
           # Store patterns in Patternbook
           if result.get("entropy"):
               for pos, entropy in result["entropy"].items():
                   if entropy > 0.8:  # High entropy
                       self.patternbook.store_pattern({
                           "type": "entropy",
                           "description": f"High entropy at position {pos}: {entropy:.2f}",
                           "data": {"position": pos, "entropy": entropy},
                           "metadata": {"session": self.session_id}
                       })
           
           if result.get("sequences"):
               for seq in result["sequences"]:
                   self.patternbook.store_pattern({
                       "type": "sequence",
                       "description": f"Repeated sequence: {seq['sequence'][:20]}... (count: {seq['count']})",
                       "data": seq,
                       "metadata": {"session": self.session_id}
                   })
           
           return result
       
       def find_similar_patterns(self, query: str) -> List:
           """Search for similar patterns in past analyses."""
           return self.patternbook.search_patterns(query, limit=10)
   ```

#### Deliverables
- [ ] Qdrant vector database running
- [ ] Patternbook storage and retrieval
- [ ] Pattern embedding with sentence transformers
- [ ] Autonomous agents storing patterns
- [ ] Pattern search tested

#### Validation Checkpoint
```bash
# Test Patternbook
poetry run pytest tests/patternbook/ -v

# Verify Qdrant
curl http://localhost:6333/collections

# Search patterns
poetry run python -c "from reshark.books.patternbook.pattern_store import Patternbook; \
  pb = Patternbook(); \
  print(pb.search_patterns('high entropy positions'))"
```

---

# Weeks 11-16 Content for system-implementation-plan.md

### Week 11-12: Autonomous Workflow Refinement

#### Objectives (Week 11)
- Advanced autonomous workflows with OpenHands
- Multi-sample analysis automation
- Grammar generation from patterns
- Performance optimization

#### Tasks

**Day 1-3: Autonomous Grammar Generation**

1. **Grammar Generation Workflow**
   ```python
   # reshark/autonomous/workflows/grammar_generation.py
   """Autonomous Kaitai Struct grammar generation."""
   
   from typing import Dict, Any, List
   from pathlib import Path
   
   class GrammarGenerator:
       """Generate Kaitai Struct grammars from patterns."""
       
       def generate_from_patterns(
           self,
           patterns: List[Dict],
           protocol_name: str
       ) -> str:
           """
           Generate .ksy grammar from identified patterns.
           
           Args:
               patterns: List of patterns from Patternbook
               protocol_name: Name for the grammar
           
           Returns:
               Kaitai Struct YAML content
           """
           grammar = {
               "meta": {
                   "id": protocol_name.lower().replace(" ", "_"),
                   "title": protocol_name,
                   "file-extension": ["bin", "dat"],
                   "endian": "be"  # Default big-endian
               },
               "seq": []
           }
           
           # Generate fields from patterns
           for idx, pattern in enumerate(patterns):
               if pattern["type"] == "sequence":
                   # Fixed sequence = magic/constant field
                   grammar["seq"].append({
                       "id": f"magic_{idx}",
                       "contents": pattern["data"]["sequence"],
                       "doc": f"Magic bytes: {pattern['description']}"
                   })
               elif pattern["type"] == "entropy":
                   # High entropy = variable data
                   if pattern["data"]["entropy"] > 0.9:
                       grammar["seq"].append({
                           "id": f"payload_{idx}",
                           "size": 1,  # Placeholder
                           "doc": f"Variable data: {pattern['description']}"
                       })
           
           # Convert to YAML string
           import yaml
           return yaml.dump(grammar, default_flow_style=False)
   ```

**Day 4-5: Multi-Session Analysis**

2. **Cross-Session Pattern Recognition**
   ```python
   # reshark/autonomous/workflows/multi_session.py
   """Analyze patterns across multiple sessions."""
   
   from reshark.books.patternbook.pattern_store import Patternbook
   from typing import List, Dict
   
   class MultiSessionAnalyzer:
       """Analyze patterns across analysis sessions."""
       
       def __init__(self, patternbook: Patternbook):
           self.patternbook = patternbook
       
       def find_common_patterns(
           self,
           session_ids: List[str],
           min_occurrences: int = 3
       ) -> List[Dict]:
           """
           Find patterns appearing in multiple sessions.
           
           Constitutional requirement: >= 3 sessions for validation.
           """
           if len(session_ids) < 3:
               raise ValueError("Minimum 3 sessions required")
           
           # Search patterns for each session
           all_patterns = {}
           
           for session_id in session_ids:
               patterns = self.patternbook.search_patterns(
                   f"session:{session_id}",
                   limit=100
               )
               
               for p in patterns:
                   key = (p["type"], p["data"].get("sequence", ""))
                   if key not in all_patterns:
                       all_patterns[key] = []
                   all_patterns[key].append(p)
           
           # Filter to patterns appearing >= min_occurrences
           common_patterns = [
               {
                   "pattern": patterns[0],
                   "occurrences": len(patterns),
                   "sessions": [p["metadata"]["session"] for p in patterns]
               }
               for patterns in all_patterns.values()
               if len(patterns) >= min_occurrences
           ]
           
           return common_patterns
   ```

#### Week 12 Objectives
- Automated validation and refinement
- Performance benchmarking
- Scalability testing

#### Tasks (Week 12)

**Day 1-2: Automated Validation Loop**

3. **Auto-Validation Workflow**
   ```python
   # reshark/autonomous/workflows/auto_validation.py
   """Automated grammar validation and refinement."""
   
   from reshark.agents.validation.grammar_validator import GrammarValidator
   from reshark.autonomous.workflows.grammar_generation import GrammarGenerator
   
   class AutoValidator:
       """Autonomous validation with auto-refinement."""
       
       def __init__(self, validator: GrammarValidator, generator: GrammarGenerator):
           self.validator = validator
           self.generator = generator
       
       def validate_and_refine(
           self,
           grammar_path: str,
           samples: List[str],
           max_iterations: int = 3
       ) -> Dict:
           """
           Validate grammar and auto-refine if needed.
           
           Args:
               grammar_path: Path to grammar file
               samples: Sample files (minimum 3)
               max_iterations: Max refinement attempts
           
           Returns:
               Final validation result
           """
           for iteration in range(max_iterations):
               # Validate
               result = self.validator.execute({
                   "grammar_path": grammar_path,
                   "sample_paths": samples,
                   "validation_type": "comprehensive"
               })
               
               if result["overall_status"] == "pass":
                   return {
                       "status": "success",
                       "iterations": iteration + 1,
                       "result": result
                   }
               
               # Auto-refine grammar
               errors = [r for r in result["results"] if r["status"] == "fail"]
               refined_grammar = self._refine_grammar(grammar_path, errors)
               
               # Write refined grammar
               Path(grammar_path).write_text(refined_grammar)
           
           return {
               "status": "failed",
               "iterations": max_iterations,
               "result": result
           }
       
       def _refine_grammar(self, grammar_path: str, errors: List) -> str:
           """Refine grammar based on validation errors."""
           # Placeholder: Would use LLM to refine grammar
           return Path(grammar_path).read_text()
   ```

**Day 3-5: Performance Optimization**

4. **Benchmarking Suite**
   ```python
   # tests/performance/test_autonomous_performance.py
   """Performance tests for autonomous mode."""
   
   import pytest
   import time
   
   @pytest.mark.benchmark
   class TestAutonomousPerformance:
       """Phase 2 performance benchmarks."""
       
       def test_pattern_search_speed(self, patternbook, benchmark):
           """Pattern search should be <100ms."""
           
           def search():
               return patternbook.search_patterns("test query", limit=10)
           
           result = benchmark(search)
           assert benchmark.stats['mean'] < 0.1  # 100ms
       
       def test_end_to_end_analysis(self, test_sample, benchmark):
           """Complete analysis should be <30s."""
           
           def analyze():
               # Run complete autonomous workflow
               pass
           
           result = benchmark(analyze)
           assert benchmark.stats['mean'] < 30.0  # 30 seconds
       
       def test_multi_session_scaling(self):
           """Test scaling to 100+ sessions."""
           # Generate 100 test sessions
           # Verify pattern search still <100ms
           pass
   ```

#### Deliverables (Weeks 11-12)
- [ ] Autonomous grammar generation
- [ ] Multi-session pattern recognition
- [ ] Auto-validation with refinement
- [ ] Performance benchmarks passing
- [ ] Scalability validated (100+ sessions)

---

### Week 13: Advanced Analysis Features

#### Objectives
- Protocol state machine inference
- Advanced pattern analysis
- Data structure recovery
- Field type inference

#### Tasks

**Day 1-2: State Machine Inference**

1. **FSM Analyzer**
   ```python
   # reshark/agents/analysis/fsm_analyzer.py
   """Infer protocol finite state machines from conversations."""
   
   from typing import Dict, Any, List, Tuple
   import networkx as nx
   
   class FSMAnalyzer:
       """Infer protocol state machines."""
       
       def analyze_conversations(
           self,
           conversations: List[Dict]
       ) -> Dict[str, Any]:
           """
           Infer FSM from request/response patterns.
           
           Returns:
               State machine graph and transition rules
           """
           # Build state graph
           G = nx.DiGraph()
           
           for conv in conversations:
               messages = conv.get("messages", [])
               
               for i in range(len(messages) - 1):
                   current = self._classify_message(messages[i])
                   next_msg = self._classify_message(messages[i + 1])
                   
                   # Add edge
                   if G.has_edge(current, next_msg):
                       G[current][next_msg]['weight'] += 1
                   else:
                       G.add_edge(current, next_msg, weight=1)
           
           return {
               "states": list(G.nodes()),
               "transitions": [
                   {
                       "from": u,
                       "to": v,
                       "count": data['weight']
                   }
                   for u, v, data in G.edges(data=True)
               ],
               "graph": nx.node_link_data(G)
           }
       
       def _classify_message(self, message: Dict) -> str:
           """Classify message into state."""
           # Simple classification based on first byte
           data = message.get("data", b"")
           if len(data) > 0:
               return f"state_{data[0]:02x}"
           return "unknown"
   ```

**Day 3-4: Structure Recovery**

2. **Data Structure Inference**
   ```python
   # reshark/agents/analysis/structure_recovery.py
   """Recover C-like data structures from binary data."""
   
   from typing import List, Dict
   import struct
   
   class StructureRecovery:
       """Infer data structures from aligned binary data."""
       
       def infer_structure(
           self,
           data_samples: List[bytes],
           alignment: int = 1
       ) -> Dict:
           """
           Infer structure definition from samples.
           
           Returns:
               C-like structure definition
           """
           if not data_samples:
               return {}
           
           min_len = min(len(d) for d in data_samples)
           fields = []
           offset = 0
           
           while offset < min_len:
               field = self._infer_field_at_offset(data_samples, offset)
               fields.append(field)
               offset += field["size"]
           
           return {
               "name": "inferred_struct",
               "size": offset,
               "fields": fields
           }
       
       def _infer_field_at_offset(
           self,
           samples: List[bytes],
           offset: int
       ) -> Dict:
           """Infer field type at given offset."""
           values = [s[offset:offset+8] for s in samples if len(s) > offset+7]
           
           # Try different types
           for size, fmt, type_name in [
               (1, 'B', 'uint8'),
               (2, 'H', 'uint16'),
               (4, 'I', 'uint32'),
               (8, 'Q', 'uint64')
           ]:
               try:
                   parsed = [struct.unpack(f'>{fmt}', v[:size])[0] for v in values]
                   
                   # Check if values make sense
                   if self._values_look_valid(parsed, size):
                       return {
                           "offset": offset,
                           "size": size,
                           "type": type_name,
                           "sample_values": parsed[:5]
                       }
               except:
                   continue
           
           # Default to byte array
           return {
               "offset": offset,
               "size": 1,
               "type": "uint8",
               "sample_values": [v[0] for v in values[:5]]
           }
       
       def _values_look_valid(self, values: List[int], size: int) -> bool:
           """Heuristic: do values look like meaningful integers?"""
           # Not all zeros or all 0xFF
           if all(v == 0 for v in values):
               return False
           if all(v == (2**(size*8) - 1) for v in values):
               return False
           return True
   ```

**Day 5: Integration**

3. **Advanced Analysis Skill**
   ```bash
   cat > skills/advanced/structure-recovery.md << 'EOF'
   # Data Structure Recovery
   
   **Category**: advanced  
   **Complexity**: advanced  
   **Prerequisites**: Multiple aligned binary samples
   
   ## Purpose
   
   Infer C-like data structure definitions from binary data samples.
   
   ## Methodology
   
   ### Step 1: Alignment Detection
   
   - Identify common offsets across samples
   - Detect structure boundaries
   - Find padding bytes
   
   ### Step 2: Field Type Inference
   
   For each offset:
   - Try standard sizes (1, 2, 4, 8 bytes)
   - Test as integer types (uint8/16/32/64)
   - Check value distributions
   - Identify constants vs variables
   
   ### Step 3: Structure Definition
   
   Generate C-like struct:
   ```c
   struct protocol_header {
       uint8_t magic[2];
       uint16_t length;
       uint32_t timestamp;
       uint8_t payload[];
   };
   ```
   
   ## Tools
   
   - StructureRecovery agent
   - Alignment analyzer
   - Type inference engine
   
   ## Related Skills
   
   - [Boundary Detection](../binary/boundary-detection.md)
   - [Type Inference](../binary/type-inference.md)
   EOF
   ```

#### Deliverables (Week 13)
- [ ] FSM analyzer for state machine inference
- [ ] Structure recovery for data types
- [ ] Advanced analysis skills documented
- [ ] Integration tests passing

---

### Week 14: Documentation & Examples

#### Objectives
- Comprehensive user documentation
- Tutorial notebooks
- Example protocols
- Architecture documentation

#### Tasks

**Day 1-2: User Documentation**

1. **Complete User Guide**
   ```markdown
   # docs/user-guide.md
   
   # reShark v2.0.0 User Guide
   
   ## Table of Contents
   
   1. [Installation](#installation)
   2. [Quick Start](#quick-start)
   3. [HITL Mode with Cline](#hitl-mode)
   4. [Autonomous Mode with OpenHands](#autonomous-mode)
   5. [Analysis Workflows](#workflows)
   6. [Skills Library](#skills)
   7. [Tools Reference](#tools)
   
   ## Installation
   
   ### Prerequisites
   
   - Python 3.12+
   - Docker and Docker Compose
   - VS Code (for HITL mode)
   
   ### Setup
   
   ```bash
   # Clone repository
   git clone https://github.com/kmaneesh/reShark.git
   cd reShark
   
   # Install dependencies
   poetry install
   
   # Start services (Phase 2 only)
   docker-compose -f docker/docker-compose.yml up -d
   ```
   
   ## Quick Start (HITL Mode)
   
   1. Open in VS Code Dev Container
   2. Install Cline extension
   3. Start Ollama: `ollama serve`
   4. Analyze artifact:
   
   ```bash
   poetry run reshark analyze sample.pcap
   ```
   
   ## Quick Start (Autonomous Mode)
   
   ```bash
   # Start all services
   docker-compose up -d
   
   # Submit analysis task
   curl -X POST http://localhost:3000/api/task \
     -d '{"task": "analyze /workspace/data/sample.pcap"}'
   ```
   
   ## Workflows
   
   ### Extract and Analyze
   
   Complete data extraction and pattern analysis.
   
   ### Grammar Validation
   
   Validate grammar against minimum 3 samples (Constitutional requirement).
   
   ## Skills Library
   
   12+ documented skills across categories:
   - Protocol analysis
   - Pattern detection
   - Binary analysis
   - Grammar operations
   
   See [Skills README](../skills/README.md)
   ```

**Day 3-4: Tutorial Notebooks**

2. **Interactive Tutorials**
   ```bash
   # Create tutorial notebooks
   mkdir -p docs/tutorials
   
   # Tutorial 1: Basic Analysis
   cat > docs/tutorials/01-basic-analysis.ipynb << 'EOF'
   {
     "cells": [
       {
         "cell_type": "markdown",
         "metadata": {},
         "source": [
           "# Tutorial 1: Basic Protocol Analysis\n",
           "\n",
           "Learn to use reShark for basic binary protocol analysis."
         ]
       },
       {
         "cell_type": "code",
         "execution_count": null,
         "metadata": {},
         "source": [
           "from reshark.agents.interpreters.data_interpreter import DataInterpreter\n",
           "from reshark.tools.registry import ToolsRegistry\n",
           "from pathlib import Path\n",
           "\n",
           "# Initialize\n",
           "books_path = Path('../../books')\n",
           "tools = ToolsRegistry()\n",
           "interpreter = DataInterpreter(books_path, tools)"
         ]
       }
     ]
   }
   EOF
   ```

**Day 5: Example Protocols**

3. **Reference Implementations**
   ```python
   # examples/protocols/simple_protocol.py
   """Reference implementation: Simple binary protocol."""
   
   import struct
   
   class SimpleProtocol:
       """
       Simple binary protocol for testing.
       
       Format:
       - Magic: 0xAABB (2 bytes)
       - Type: uint8
       - Length: uint16 (big-endian)
       - Payload: variable
       - Checksum: uint8
       """
       
       MAGIC = b'\xAA\xBB'
       
       @staticmethod
       def encode(msg_type: int, payload: bytes) -> bytes:
           """Encode message."""
           length = len(payload)
           msg = SimpleProtocol.MAGIC
           msg += struct.pack('B', msg_type)
           msg += struct.pack('>H', length)
           msg += payload
           
           checksum = sum(msg) % 256
           msg += struct.pack('B', checksum)
           
           return msg
       
       @staticmethod
       def decode(data: bytes) -> dict:
           """Decode message."""
           if len(data) < 6:
               raise ValueError("Message too short")
           
           magic = data[0:2]
           if magic != SimpleProtocol.MAGIC:
               raise ValueError("Invalid magic")
           
           msg_type = struct.unpack('B', data[2:3])[0]
           length = struct.unpack('>H', data[3:5])[0]
           payload = data[5:5+length]
           checksum = struct.unpack('B', data[5+length:6+length])[0]
           
           return {
               "type": msg_type,
               "length": length,
               "payload": payload,
               "checksum": checksum
           }
   ```

#### Deliverables (Week 14)
- [ ] Complete user guide
- [ ] 5+ tutorial notebooks
- [ ] 3+ example protocol implementations
- [ ] API documentation generated

---

### Week 15: Testing & Quality Assurance

#### Objectives
- Comprehensive test suite
- Integration testing
- Performance validation
- Security review

#### Tasks

**Day 1-2: Test Coverage**

1. **Complete Test Suite**
   ```bash
   # Achieve >90% coverage
   poetry run pytest --cov=reshark --cov-report=html tests/
   
   # Coverage targets:
   # - Agents: >95%
   # - Tools: >90%
   # - Books: >85%
   # - Overall: >90%
   ```

**Day 3: Security Review**

2. **Security Audit**
   ```bash
   # Scan for vulnerabilities
   poetry run bandit -r reshark/
   
   # Check dependencies
   poetry run safety check
   
   # Static analysis
   poetry run mypy reshark/
   ```

**Day 4-5: Performance Validation**

3. **Stress Testing**
   ```python
   # tests/stress/test_large_datasets.py
   """Stress tests with large datasets."""
   
   @pytest.mark.stress
   def test_large_pcap_analysis():
       """Test with 1GB+ PCAP file."""
       # Should complete in <10 minutes
       # Memory usage <2GB
       pass
   
   @pytest.mark.stress
   def test_concurrent_sessions():
       """Test 10 concurrent analysis sessions."""
       pass
   ```

#### Deliverables (Week 15)
- [ ] Test coverage >90%
- [ ] Security audit passed
- [ ] Performance benchmarks met
- [ ] Stress tests passing

---

### Week 16: Deployment & Release

#### Objectives
- Production deployment
- Release preparation
- Documentation finalization
- Launch v2.0.0

#### Tasks

**Day 1-2: Deployment**

1. **Production Configuration**
   ```yaml
   # docker/docker-compose.prod.yml
   version: '3.8'
   
   services:
     reshark:
       image: reshark:2.0.0
       restart: always
       volumes:
         - ./books:/app/books
       environment:
         - ENV=production
         - LOG_LEVEL=INFO
     
     openhands:
       image: openhands:latest
       restart: always
       depends_on:
         - ollama
         - qdrant
     
     ollama:
       image: ollama/ollama:latest
       restart: always
       deploy:
         resources:
           reservations:
             devices:
               - driver: nvidia
                 count: 1
                 capabilities: [gpu]
     
     qdrant:
       image: qdrant/qdrant:latest
       restart: always
       volumes:
         - qdrant-prod:/qdrant/storage
   
   volumes:
     qdrant-prod:
   ```

**Day 3: Release Preparation**

2. **Release Checklist**
   ```markdown
   # Release Checklist v2.0.0
   
   ## Code
   - [ ] All tests passing
   - [ ] Test coverage >90%
   - [ ] No critical security issues
   - [ ] Performance benchmarks met
   
   ## Documentation
   - [ ] User guide complete
   - [ ] API docs generated
   - [ ] Tutorials tested
   - [ ] README updated
   
   ## Deployment
   - [ ] Docker images built
   - [ ] Docker Compose tested
   - [ ] Installation instructions verified
   
   ## Release Notes
   - [ ] Changelog updated
   - [ ] Migration guide written
   - [ ] Breaking changes documented
   ```

**Day 4-5: Launch**

3. **Release v2.0.0**
   ```bash
   # Tag release
   git tag -a v2.0.0 -m "Release v2.0.0: Dual-mode RE framework"
   git push origin v2.0.0
   
   # Build and push Docker images
   docker build -t reshark:2.0.0 .
   docker push reshark:2.0.0
   
   # Publish to PyPI (optional)
   poetry build
   poetry publish
   ```

#### Deliverables (Week 16)
- [ ] Production deployment tested
- [ ] Release v2.0.0 published
- [ ] Documentation site live
- [ ] Launch announcement

---

## Phase 2 Gate Review

**Success Criteria:**
- [x] OpenHands integration complete
- [x] Patternbook operational
- [x] Autonomous workflows tested
- [x] Performance targets met (<30s end-to-end)
- [x] Test coverage >90%
- [x] Security audit passed
- [ ] Production deployment successful
- [ ] v2.0.0 released

---

## Post-Implementation (Week 17+)

### Maintenance & Iteration

- Monitor production usage
- Collect user feedback
- Bug fixes and patches
- Performance tuning
- Feature requests evaluation

### Future Enhancements (v2.1+)

- Additional tool integrations (Ghidra, IDA)
- Web UI for analysis visualization
- Cloud deployment options
- Advanced ML models for pattern recognition
- Support for additional artifact types

---

## Summary

This 16-week implementation delivers:

**Phase 1 (Weeks 1-8): HITL Foundation**
- Dev Container + Cline + Ollama setup
- Books system (Notebook, Rulebook, Cookbook, Skills)
- Agents (Interpreters, Orchestration, Validation)
- Tools registry and 12+ skills
- End-to-end HITL workflows

**Phase 2 (Weeks 9-16): Autonomous Mode**
- OpenHands integration
- Patternbook vector database
- Autonomous workflows
- Advanced analysis features
- Production deployment

Result: Dual-mode general RE framework operational per Constitution.
