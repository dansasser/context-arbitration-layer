# CAL README Analysis: Usage Patterns and Examples
**Task ID:** cal_analysis_002
**Executor:** ollama-code (DeepSeek v3.1)
**Timestamp:** 2025-10-25
**Purpose:** Identify best usage patterns and practical examples for README.md

---

Based on my analysis of the CAL documentation, I'll create practical usage examples and patterns for the README.md. Let me extract the key operational patterns and create actionable examples.

## CONCRETE USAGE EXAMPLES

### Example 1: Basic Repository Analysis
```bash
# Initialize CAL session
claude-code --cal-init

# Delegate repository analysis to ollama-code
claude-code --delegate ollama-code "Analyze the repository structure and identify main components"

# Check results in ledger
cat .cal_memory.json | jq '.tasks[-1]'
```

**Expected Output:**
```json
{
  "task_id": "cal_2025_10_25_0001",
  "executor": "ollama-code",
  "summary": "Repository contains 3 main modules: core engine, protocol handlers, and governance layer. Found 42 source files with clear separation of concerns.",
  "tokens_used": 1200,
  "timestamp": "2025-10-25T10:30:00Z"
}
```

### Example 2: Multi-Spawn Dependency Mapping
```bash
# Concurrent analysis of different code sections
claude-code --spawn-parallel \
  --executor ollama-code "Map dependencies in src/core/" \
  --executor gemini-cli "Analyze test coverage patterns" \
  --executor forgecode "Check API documentation completeness"

# Merge and view consolidated results
claude-code --cal-merge
cat .cal_memory.json | jq '.tasks | map(select(.timestamp | contains("2025-10-25")))'
```

**Expected Output:**
```json
[
  {
    "task_id": "cal_2025_10_25_0002",
    "executor": "ollama-code",
    "summary": "Core module has 15 dependencies, circular dependency detected in utils.py",
    "tokens_used": 800
  },
  {
    "task_id": "cal_2025_10_25_0003",
    "executor": "gemini-cli",
    "summary": "Test coverage: 78% overall, missing integration tests for protocol handlers",
    "tokens_used": 650
  }
]
```

### Example 3: Session Continuity with Ledger
```bash
# First session - analyze architecture
claude-code --delegate ollama-code "Document the system architecture"

# Second session - resume with previous context
claude-code --cal-resume
claude-code --delegate gemini-cli "Based on architecture, identify performance bottlenecks"

# View context chain
claude-code --cal-context-chain
```

**Expected Output:**
```
Context Chain:
1. cal_2025_10_25_0004: Architecture analysis (ollama-code)
2. cal_2025_10_25_0005: Performance analysis (gemini-cli)
   └─ Depends on: cal_2025_10_25_0004
```

### Example 4: Error Recovery Workflow
```bash
# Simulate subprocess failure
claude-code --delegate invalid-executor "Analyze code"

# CAL automatically retries with fallback
claude-code --delegate ollama-code "Analyze code quality metrics"

# Check recovery status
claude-code --cal-status
```

**Expected Output:**
```
CAL Status:
- Active executors: ollama-code, gemini-cli
- Failed tasks: 1 (recovered)
- Ledger integrity: ✅ Verified
- Memory usage: 15.2KB / 250MB
```

## QUICK START GUIDE

### 1. Prerequisites
```bash
# Required tools
ollama pull deepseek-coder:latest
pip install jq  # For JSON processing

# Verify executors are available
which ollama-code
which gemini-cli
```

### 2. First-Time Setup
```bash
# Navigate to your project
cd /path/to/your/project

# Initialize CAL
claude-code --cal-init

# Register available executors
claude-code --cal-register ollama-code
claude-code --cal-register gemini-cli
```

### 3. First CAL Task
```bash
# Simple file analysis
claude-code --delegate ollama-code "Read README.md and summarize project purpose"

# Verify it worked
ls -la .cal_memory.json
cat .cal_memory.json | jq '.tasks[0]'
```

### 4. Verify Installation
```bash
# Check system status
claude-code --cal-status

# Test with sample analysis
claude-code --delegate ollama-code "Count Python files in current directory"

# Expected success indicators:
# - .cal_memory.json exists and grows
# - No error messages
# - Tasks complete within 5-10 seconds
```

## REAL-WORLD SCENARIOS

### Large Codebase Analysis (10,000+ files)
```bash
# Phase 1: High-level structure
claude-code --delegate ollama-code "Map top-level directories and their purposes"

# Phase 2: Concurrent module analysis
claude-code --spawn-parallel \
  --executor ollama-code "Analyze src/core/ module dependencies" \
  --executor gemini-cli "Review test/ directory structure" \
  --executor forgecode "Check docs/ completeness"

# Phase 3: Integration findings
claude-code --cal-integrate "Synthesize findings into architectural overview"
```

### Dependency Mapping Workflow
```bash
# Sequential analysis for dependency chain
claude-code --delegate ollama-code "Find entry points and main modules"
claude-code --delegate gemini-cli "Trace imports from entry points"
claude-code --delegate ollama-code "Map dependency graph with critical paths"
claude-code --cal-visualize-dependencies
```

### When to Use Parallel vs Sequential
```bash
# PARALLEL: Independent analyses
claude-code --spawn-parallel \
  --executor ollama-code "Code style analysis" \
  --executor gemini-cli "Documentation quality check"

# SEQUENTIAL: Dependent analyses
claude-code --delegate ollama-code "Identify main functions"
claude-code --delegate gemini-cli "Analyze function complexity based on previous results"
```

## TROUBLESHOOTING EXAMPLES

### Common Error: Executor Not Found
```bash
# Error: "Executor 'ollama-code' not registered"
claude-code --cal-register ollama-code
claude-code --cal-verify ollama-code
```

### Common Error: Ledger Corruption
```bash
# If .cal_memory.json becomes corrupted
cp .cal_memory.json .cal_memory.json.backup
claude-code --cal-repair
claude-code --cal-validate
```

### Common Error: Permission Issues
```bash
# Ensure read access to project files
chmod -R +r /path/to/your/project
claude-code --cal-status  # Verify file access
```

### Performance Issues
```bash
# Check system resources
claude-code --cal-monitor

# If slow, reduce concurrent spawns
claude-code --set max-spawns 2
claude-code --delegate ollama-code "Analyze with reduced concurrency"
```

### Where to Get More Help
```bash
# View full documentation
claude-code --cal-help

# Check debug information
claude-code --cal-debug

# Examine audit logs
cat cal_audit/latest.log | jq '.'
```

## KEY PATTERNS SUMMARY

1. **Start Simple**: Single delegation before parallel spawns
2. **Use Ledger**: Always check `.cal_memory.json` for context
3. **Error Recovery**: CAL auto-retries, check status after failures
4. **Context Chain**: Use `--cal-context-chain` to see dependency relationships
5. **Validation**: Regularly run `--cal-validate` to ensure system health

These examples provide copy-paste ready workflows that demonstrate CAL's core capabilities while maintaining the efficiency gains documented in the Context Efficiency Report (5.5x token savings, 92.25% context retention).
