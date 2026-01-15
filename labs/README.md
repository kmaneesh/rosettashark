# Labs - Execution Environments

**Version**: 2.0.0  
**Purpose**: Pre-configured containerized environments for analysis

---

## Overview

Labs provide isolated, reproducible execution environments for both HITL and Autonomous modes. Each lab container comes pre-loaded with analysis tools and dependencies.

## Available Labs

### 1. Dockerfile.openhands
**Purpose**: Autonomous mode execution with OpenHands + Local LLM

**Includes**:
- Python 3.12 + Node.js 22
- OpenHands framework
- Local LLM client (Ollama)
- Analysis tools (tshark, wireshark-common, tcpdump)
- Binary utilities (binutils, hexdump)

**Usage**:
```bash
# Build
docker build -f Dockerfile.openhands -t reshark-autonomous:latest .

# Run with docker-compose (recommended)
docker-compose up -d

# Access
docker exec -it reshark-autonomous bash
```

### 2. Dockerfile.pcap
**Purpose**: PCAP analysis tools for both modes

**Includes**:
- tshark, termshark, wireshark-common
- Python packages: pyshark, scapy, dpkt
- Data analysis: pandas, numpy, scikit-learn
- CLI utilities: rich, typer

**Usage**:
```bash
# Build
docker build -f Dockerfile.pcap -t reshark-pcap:latest .

# Run interactively
docker run -it --rm \
  -v $(pwd)/books:/workspace/books \
  -v $(pwd)/data/input:/workspace/data:ro \
  reshark-pcap:latest
```

---

## Docker Compose Setup

```bash
# Start all services (autonomous mode + dependencies)
docker-compose up -d

# Services:
# - reshark-autonomous: Main analysis environment
# - ollama: Local LLM service
# - qdrant: Vector database for Patternbook

# View logs
docker-compose logs -f reshark-autonomous

# Stop all services
docker-compose down
```

---

## Environment Configuration

### Network Isolation
Labs use internal Docker networks for security. No external network access by default.

### Volume Mounts
- `/app/books`: Knowledge storage (books directory)
- `/app/data/input`: Input data (read-only)
- `/app/data/output`: Analysis results

### Environment Variables
- `MODE`: `hitl` or `autonomous`
- `OLLAMA_HOST`: Local LLM endpoint
- `PYTHONUNBUFFERED=1`: Real-time logging

---

## Tool Verification

```bash
# Inside container
python --version      # Should show Python 3.12.x
node --version        # Should show v22.x.x
tshark --version      # Should show TShark/Wireshark version
```

---

## Development Workflows

### HITL Mode (Human-in-the-Loop)

Uses VS Code Dev Container with Cline + Local LLM:

```bash
# Open project in VS Code
code /Users/m/Sites/kmaneesh.github/reShark

# Dev Container will auto-start (configured in .devcontainer/)
# - Mounts workspace
# - Connects to Ollama service
# - Loads Cline extension
# - Configures Python environment

# Workflow:
# 1. Human writes initial analysis request in Cline
# 2. Cline + LLM generate agents/skills
# 3. Human reviews and refines
# 4. Results stored in books/notebook/
# 5. Patterns extracted to books/patternbook/
# 6. Rules formalized in books/rulebook/
```

### Autonomous Mode

Uses OpenHands container for unattended operation:

```bash
# Start autonomous environment
docker-compose up -d

# Submit task via API
curl -X POST http://localhost:3000/api/task \
  -H "Content-Type: application/json" \
  -d '{
    "type": "analyze_pcap",
    "input": "/workspace/data/sample.pcap",
    "grammar": "/workspace/books/rulebook/grammars/protocol_x.ksy"
  }'

# Monitor execution
docker logs -f reshark-autonomous

# Retrieve results
docker cp reshark-autonomous:/workspace/data/output/results.json .
```

---

## Lab Customization

### Adding Tools

Edit Dockerfile and rebuild:

```dockerfile
# Dockerfile.pcap
RUN apt-get update && apt-get install -y \
    your-new-tool \
    && apt-get clean
```

### Python Packages

Add to `requirements.txt` or install in container:

```bash
docker exec reshark-autonomous pip install your-package
```

### Environment Tuning

Adjust `docker-compose.yml` resources:

```yaml
services:
  reshark-autonomous:
    deploy:
      resources:
        limits:
          cpus: '4'
          memory: 8G
```

---

## Security Considerations

1. **Network Isolation**: Containers use internal networks only
2. **Read-Only Mounts**: Input data mounted as read-only
3. **User Permissions**: Run as non-root user inside container
4. **Resource Limits**: CPU and memory caps prevent abuse
5. **Audit Logging**: All operations logged to `/workspace/data/output/audit.log`

---

## Troubleshooting

### Container Won't Start

```bash
# Check logs
docker-compose logs reshark-autonomous

# Common issues:
# - Port conflicts (3000, 11434)
# - Volume mount permissions
# - Insufficient memory
```

### Tool Not Found

```bash
# Verify installation
docker exec reshark-autonomous which tshark

# Reinstall if missing
docker exec -u root reshark-autonomous apt-get install -y tshark
```

### Ollama Connection Issues

```bash
# Check Ollama service
docker-compose ps ollama

# Test connection from container
docker exec reshark-autonomous curl http://ollama:11434/api/version
```

---

## Performance Tips

1. **Pre-pull Models**: Download LLM models before starting:
   ```bash
   docker exec ollama ollama pull llama3.2:3b
   ```

2. **Volume Caching**: Use named volumes for better performance:
   ```yaml
   volumes:
     books-cache:
       driver: local
   ```

3. **Resource Allocation**: Increase limits for large PCAPs:
   ```yaml
   deploy:
     resources:
       limits:
         memory: 16G
   ```

---

## References

- [System Requirements Specification](../specs/system-requirements-specification.md)
- [System Implementation Technical Design](../specs/system-implementation-technical-design.md)
- [Docker Documentation](https://docs.docker.com/)
- [OpenHands Documentation](https://github.com/All-Hands-AI/OpenHands)
