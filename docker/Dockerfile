FROM nikolaik/python-nodejs:python3.12-nodejs22

# Install required packages
RUN apt-get update && apt-get install -y \
    tshark \
    termshark \
    wireshark-common \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Note: editcap, mergecap, and capinfos are included in the tshark/wireshark-common package

# Install Python packages
RUN pip install --no-cache-dir \
    pyshark \
    scapy \
    dpkt \
    pandas \
    numpy \
    scikit-learn \
    pydantic \
    rich \
    typer

# Set working directory
WORKDIR /workspace

# Environment variables
ENV PYTHONUNBUFFERED=1
ENV DEBIAN_FRONTEND=noninteractive

# Create workspace structure
RUN mkdir -p /workspace/notebook \
    /workspace/rulebook/grammars \
    /workspace/rulebook/tests \
    /workspace/cookbook \
    /workspace/pcaps

# Default command
CMD ["/bin/bash"]
