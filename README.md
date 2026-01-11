
# 🦈 reShark  
Autonomous Protocol Archaeology Lab

reShark is an open-source system for discovering how unknown network protocols work — directly from PCAP files.

It treats network traffic as a language and reconstructs its grammar through statistics, hypothesis testing, and validation.

## What It Does

PCAP → Bytes → Message boundaries → Fields → States → Protocol model

## Architecture

Three-layer design:

Storage (Volume)  
Execution (Docker Sandbox with tshark, zeek, scapy, numpy)  
Intelligence (Observer, Theorist, Validator agents)

## Memory

Notebook – short term hypotheses  
Rulebook – validated grammars  
Dream Brain – vector similarity memory

## Workflow

1. Normalize PCAPs  
2. Measure byte entropy  
3. Find control bytes  
4. Build grammar  
5. Validate  
6. Promote to Rulebook  

## Open Core

reShark is open-source for labs and tooling.  
Extracted real-world protocol grammars remain private.
