# reShark Docker Environment

Docker environment with Python 3.12, Node.js 22, and tshark for protocol analysis.

## Quick Start

```bash
# Build and run
docker compose up -d

# Enter container
docker compose exec reshark bash

# Test tshark
docker compose exec reshark tshark --version
```

## Build Only

```bash
# Build image
docker build -t reshark:latest .

# Run interactively
docker run -it --rm -v $(pwd):/workspace reshark:latest
```

## Verify Installation

```bash
# Inside container
python --version      # Should show Python 3.12.x
node --version        # Should show v22.x.x
tshark --version      # Should show TShark/Wireshark version
```
