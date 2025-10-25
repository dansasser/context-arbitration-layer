# README.md Outline (User Documentation)
**Purpose:** Help users understand what CAL is and how to use it
**Audience:** Developers who want to USE CAL in their workflow
**Tone:** Friendly, explanatory, example-driven
**Length:** ~300-400 lines

---

## Section 1: Introduction
**Content:**
- One-sentence description: "CAL extends claude-code into a multi-process reasoning system"
- What CAL does (context arbitration, subprocess orchestration)
- Who it's for (developers using claude-code for large codebases)
- Key benefit: Proven 5.5x token efficiency, 92.25% context retention

**Length:** ~40 lines

---

## Section 2: The Problem CAL Solves
**Content:**
- Context window limitations in long sessions
- Token waste from re-analyzing same files
- Summarization drift and lost context
- Visual: Simple before/after diagram

**Length:** ~50 lines

---

## Section 3: How CAL Works (High-Level)
**Content:**
- Simple architecture diagram (controller → executors → ledger)
- 3 core components explained:
  - claude-code as controller
  - ollama-code as executor(s)
  - .cal_memory.json as persistent ledger
- Simple workflow: delegate → execute → validate → persist

**Length:** ~60 lines

---

## Section 4: Key Features
**Content:**
- Multi-spawn concurrent execution
- Persistent context across sessions
- Automatic task arbitration
- Token efficiency (empirical results)
- Model-agnostic design
- Fault tolerance and recovery

**Length:** ~50 lines

---

## Section 5: Proven Results
**Content:**
- Table: Token savings (69,500 tokens saved)
- Chart: Context retention (92.25% vs 57.5%)
- Efficiency multiplier (5.5x)
- Real-world example metrics
- Source: Context Efficiency Report (2025-10-25)

**Length:** ~60 lines

---

## Section 6: Quick Start
**Content:**
- Prerequisites (claude-code, ollama-code installed)
- Basic setup steps
- First CAL-orchestrated task example
- Expected output format

**Example:**
```bash
# CAL automatically spawns ollama-code when needed
claude-code --with-cal "analyze this codebase"

# CAL spawns multiple ollama-code instances in parallel
# Results stored in .cal_memory.json
# Summary reintegrated into claude-code context
```

**Length:** ~70 lines

---

## Section 7: Usage Examples
**Content:**
- Example 1: Parallel codebase analysis
- Example 2: Dependency mapping across large repo
- Example 3: Sequential analysis with context reuse
- Example 4: Using ledger for session continuity

**Each example shows:**
- Task description
- How CAL orchestrates it
- Expected output
- Token savings

**Length:** ~90 lines

---

## Section 8: The .cal_memory.json Ledger
**Content:**
- What it is (persistent context store)
- What it contains (task summaries, checksums, dependencies)
- How it enables session continuity
- Simple example entry
- When/how it's updated
- Note: "See SPECIFICATION.md for complete schema"

**Length:** ~50 lines

---

## Section 9: Comparison to Alternatives
**Content:**
- Simple comparison table:
  - claude-code alone
  - claude-code with sub-agents
  - claude-code with CAL

**Compare:**
- Token efficiency
- Context persistence
- Multi-model support
- Concurrent execution
- Fault tolerance

**Length:** ~40 lines

---

## Section 10: When to Use CAL
**Content:**
- Large codebase analysis (100+ files)
- Long debugging sessions
- Iterative development with context preservation
- Multi-file refactoring tasks
- Documentation generation
- When NOT to use CAL (simple single-file tasks)

**Length:** ~40 lines

---

## Section 11: Configuration & Customization
**Content:**
- Concurrency limits (default: 3-5 spawns)
- Ledger location (.cal_memory.json)
- Executor selection (ollama-code by default)
- Arbitration modes (sequential/concurrent/staggered)
- Note: "See SPECIFICATION.md for advanced configuration"

**Length:** ~40 lines

---

## Section 12: Troubleshooting
**Content:**
- Common issues and solutions:
  - Subprocess crashes
  - Ledger corruption
  - Memory overflow
  - Timeout errors
- Where to find logs
- Recovery procedures
- Note: "See SPECIFICATION.md Section 7 for error codes"

**Length:** ~50 lines

---

## Section 13: Links & Resources
**Content:**
- Link to SPECIFICATION.md (for implementation details)
- Link to Context Efficiency Report
- Repository/project links
- Contributing guidelines
- Issue reporting

**Length:** ~30 lines

---

## Section 14: License
**Content:**
- Dual-license notice (MIT OR AGPL-3.0)
- Copyright notice
- Simple explanation of license choice
- Link to full license text in SPECIFICATION.md

**Length:** ~30 lines

---

**Total Estimated Length:** ~700 lines (adjusted from initial 300-400 estimate due to examples)

**Note:** This is USER documentation - no implementation details, no pseudocode, no deep technical specifications. All technical depth lives in SPECIFICATION.md.
