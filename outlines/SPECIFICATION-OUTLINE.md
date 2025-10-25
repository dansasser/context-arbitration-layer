# SPECIFICATION.md Outline (Implementation Blueprint)
**Purpose:** Complete technical specification for implementing CAL
**Audience:** Developers who will BUILD/IMPLEMENT CAL
**Tone:** Formal, precise, requirement-driven
**Length:** ~800-1000 lines

---

## Section 1: System Overview
**Content:**
- Formal system definition
- Design goals and constraints
- System boundaries
- Assumptions and dependencies
- Conformance requirements

**Requirements:**
- REQ-SYS-001: Controller must be claude-code
- REQ-SYS-002: Executors must accept stdin prompts
- REQ-SYS-003: Ledger must be JSON format

**Length:** ~80 lines

---

## Section 2: Architecture Specification
**Content:**
- Detailed component architecture
- Component interaction model
- Data flow diagrams
- State management requirements
- Component responsibilities matrix

**Diagrams:**
- Detailed architectural topology
- Component interaction sequence
- State transition diagrams

**Tables:**
- Component roles and interfaces
- Data flow paths
- State definitions

**Length:** ~120 lines

---

## Section 3: Data Structures and Schemas
**Content:**
- Complete .cal_memory.json schema
- Field specifications (type, constraints, validation)
- Registry.json schema
- Task metadata structure
- Checksum format specification

**Example Schemas:**
```json
{
  "task_id": "string (UUID format, required)",
  "executor": "string (registered executor ID, required)",
  "summary": "string (2-10 KB, required)",
  "tokens_used": "integer (≥0, required)",
  "checksum": "string (SHA256, required)",
  "linked_tasks": "array[string] (optional)",
  "timestamp": "string (ISO8601, required)",
  "metadata": "object (arbitrary key-value)"
}
```

**Validation Rules:**
- Schema version compatibility
- Field type enforcement
- Constraint checking
- Checksum verification algorithm

**Length:** ~150 lines

---

## Section 4: Arbitration and Scheduling Algorithms
**Content:**
- Complete arbitration decision tree
- Scheduling algorithm pseudocode
- Concurrency control mechanisms
- Dependency resolution algorithm
- Priority weighting formulas

**Algorithms:**
```python
def arbitrate(task):
    if task.requires_context():
        return ExecutionMode.SEQUENTIAL
    elif task.is_independent():
        return ExecutionMode.CONCURRENT
    elif task.io_load_high():
        return ExecutionMode.STAGGERED
    elif ledger.conflict(task):
        return ExecutionMode.QUEUED
    return ExecutionMode.CONCURRENT
```

**Requirements:**
- REQ-ARB-001: Arbitration must complete in <50ms
- REQ-ARB-002: Concurrency cap configurable (default: 5)
- REQ-ARB-003: Context-dependent tasks must serialize

**Length:** ~140 lines

---

## Section 5: Multi-Spawn Coordination Protocol
**Content:**
- Spawn lifecycle states (idle/running/await_merge/terminated)
- Process state transitions
- Mutex and lock specifications
- Lock hierarchy and timeout values
- Synchronization protocol

**Tables:**
- Process states and transitions
- Lock types and timeouts
- Mutex hierarchy

**Pseudocode:**
- Lock acquisition protocol
- Deadlock prevention
- Exponential backoff implementation

**Requirements:**
- REQ-MSP-001: Global lock timeout = 1500ms
- REQ-MSP-002: Task lock timeout = 500ms
- REQ-MSP-003: Max retry attempts = 5

**Length:** ~130 lines

---

## Section 6: Communication Protocol
**Content:**
- stdin/stdout protocol specification
- Message format (JSON schema)
- Request/response structure
- Handshake protocol
- Error message format

**Protocol Schema:**
```json
// Request (controller → executor)
{
  "prompt": "string (required)",
  "metadata": {
    "task_id": "string",
    "context": "object (optional)"
  }
}

// Response (executor → controller)
{
  "summary": "string (required)",
  "tokens_used": "integer (required)",
  "checksum": "string (SHA256, required)",
  "error": "string (optional)",
  "status": "success|error"
}
```

**Requirements:**
- REQ-COM-001: All messages must be valid JSON
- REQ-COM-002: Checksum must be SHA256
- REQ-COM-003: Timeout for executor response = 5s

**Length:** ~100 lines

---

## Section 7: Error Handling and Recovery
**Content:**
- Complete error taxonomy
- Error code definitions (E001-E008)
- Recovery procedures for each error type
- Retry policies (exponential backoff)
- Rollback procedures
- Failure detection mechanisms

**Error Code Table:**
| Code | Condition | Detection | Recovery | Backoff |
|------|-----------|-----------|----------|---------|
| E001 | Subprocess crash | Exit code ≠ 0 | Respawn | 500→1500ms |
| E002 | Timeout | No heartbeat >5s | Terminate→Retry | Incremental |
| E003 | Malformed JSON | Schema fail | Re-parse | Fixed 2s |
| E004 | Checksum mismatch | Hash validation | Rollback snapshot | Immediate |
| E005 | Disk I/O failure | IOError | Retry 3x → halt | 250ms→1s |
| E006 | Memory overflow | Heap alert | Purge cache | Adaptive |
| E007 | Ledger lock timeout | Mutex >2s | Requeue | Exponential |
| E008 | Audit write error | Permission | Temp buffer | N/A |

**Recovery Algorithm:**
```python
def recover(task, error):
    log_event(task.id, error)
    for attempt in range(MAX_RETRIES):
        try:
            result = rerun(task)
            if verify_checksum(result):
                commit(result)
                return SUCCESS
        except Exception as e:
            delay = exponential_backoff(attempt)
            sleep(delay)
    rollback(task)
    flag_audit(task.id)
    return FAILED
```

**Requirements:**
- REQ-ERR-001: All errors must log to audit trail
- REQ-ERR-002: Max retry attempts = 3 (except E007)
- REQ-ERR-003: Critical errors (E005) must halt system

**Length:** ~140 lines

---

## Section 8: Security and Access Control
**Content:**
- Permission model specification
- Access control matrix
- Checksum enforcement algorithm
- Sandbox requirements
- Environment sanitization

**Permission Matrix:**
| Component | Ledger Read | Ledger Write | Ledger Modify | Filesystem |
|-----------|-------------|--------------|---------------|------------|
| Controller | ✅ | ✅ | ✅ | Full access |
| Executor | ✅ | ❌ | ❌ | Read-only |
| Ledger | - | Restricted | Checksummed | - |

**Security Requirements:**
- REQ-SEC-001: Only controller may write ledger
- REQ-SEC-002: All writes must validate checksum
- REQ-SEC-003: Executors run in isolated shell
- REQ-SEC-004: Environment vars sanitized before spawn

**Length:** ~80 lines

---

## Section 9: Performance Requirements
**Content:**
- Latency requirements (per operation)
- Throughput targets
- Resource limits
- Scalability constraints
- Benchmark methodology

**Performance Table:**
| Operation | Requirement | Measured (Baseline) |
|-----------|-------------|---------------------|
| Arbitration | <50ms | 42ms |
| Ledger append | <120ms | 84ms |
| Checksum validation | <30ms | 18ms |
| Subprocess spawn | <500ms | 380ms |
| Context reintegration | <80ms | 64ms |

**Requirements:**
- REQ-PERF-001: Token efficiency ≥5x baseline
- REQ-PERF-002: Context retention ≥90%
- REQ-PERF-003: Concurrent spawns ≤12 (configurable)
- REQ-PERF-004: Ledger rotation at 250MB

**Length:** ~90 lines

---

## Section 10: Ledger Management
**Content:**
- Ledger lifecycle management
- Rotation policy specification
- Archive format
- Ledger validation procedures
- Backup and recovery
- Query interface

**Ledger Operations:**
- Append (atomic write)
- Read (query by task_id, executor, timestamp)
- Rotate (archive old entries)
- Validate (checksum verification)
- Compact (remove redundant entries)

**Requirements:**
- REQ-LED-001: Ledger writes must be atomic
- REQ-LED-002: Rotation threshold = 250MB
- REQ-LED-003: Archives stored in .cal_memory_history/
- REQ-LED-004: Checksum logged in .cal_hashlog

**Length:** ~100 lines

---

## Section 11: Executor Registration and Management
**Content:**
- Executor registration protocol
- Capability matrix specification
- Handshake validation
- Executor lifecycle management
- Registry schema

**Registration Handshake:**
1. Controller sends ping test
2. Executor returns valid JSON with summary field
3. Controller verifies structure and checksum
4. Executor added to active pool

**Registry Schema:**
```json
{
  "executors": [
    {
      "id": "string (required)",
      "type": "cli|http|mcp",
      "version": "string",
      "status": "active|inactive",
      "capabilities": {
        "prompt_input": true,
        "json_output": true,
        "filesystem_access": true,
        "checksum_support": false
      }
    }
  ]
}
```

**Requirements:**
- REQ-EXE-001: Executors must accept stdin/CLI -p
- REQ-EXE-002: Executors must return JSON
- REQ-EXE-003: Failed validation → inactive status
- REQ-EXE-004: Auto-discovery (future)

**Length:** ~90 lines

---

## Section 12: Testing and Validation
**Content:**
- Unit test requirements
- Integration test scenarios
- End-to-end test cases
- Performance test methodology
- Compliance validation

**Test Requirements:**
- Unit tests: Each algorithm must have ≥3 test cases
- Integration: Multi-spawn scenarios (sequential, concurrent, staggered)
- E2E: Complete workflow from spawn to ledger commit
- Performance: Token efficiency measurement
- Compliance: Checksum verification, ledger integrity

**Test Cases:**
1. Sequential spawn with context dependency
2. Concurrent spawn (3+ executors)
3. Staggered spawn with I/O delays
4. Error recovery (E001-E008)
5. Ledger rotation at threshold
6. Checksum mismatch rollback

**Length:** ~90 lines

---

## Section 13: Compliance and Governance
**Content:**
- Deterministic execution requirements
- Integrity validation procedures
- Audit trail specifications
- Traceability requirements
- Replay capability

**Compliance Checkpoints:**
- Ledger integrity (100% verified)
- Determinism (≤0.2% output variance)
- Schema conformance (no violations)
- Recovery fidelity (99.8% identical)
- Access control (no unauthorized writes)

**Audit Trail Format:**
```json
{
  "timestamp": "ISO8601",
  "task_id": "UUID",
  "executor": "string",
  "hash": "SHA256",
  "status": "validated|failed"
}
```

**Requirements:**
- REQ-GOV-001: All operations must be auditable
- REQ-GOV-002: Audit logs in /cal_audit/
- REQ-GOV-003: Replay capability via --replay-audit
- REQ-GOV-004: Deterministic output (same input → same ledger state)

**Length:** ~80 lines

---

## Section 14: Deployment and Configuration
**Content:**
- Installation requirements
- Configuration parameters
- Environment variables
- Directory structure
- Initialization procedures

**Configuration Parameters:**
```
CAL_CONCURRENCY_LIMIT=5
CAL_LEDGER_PATH=.cal_memory.json
CAL_AUDIT_PATH=/cal_audit/
CAL_ROTATION_THRESHOLD_MB=250
CAL_EXECUTOR_TIMEOUT_MS=5000
CAL_RETRY_MAX=3
```

**Requirements:**
- REQ-DEP-001: Python ≥3.9 (if implementing in Python)
- REQ-DEP-002: ollama-code installed and in PATH
- REQ-DEP-003: Write access to project directory
- REQ-DEP-004: 500MB free disk space minimum

**Length:** ~70 lines

---

## Section 15: License and Attribution
**Content:**
- Complete license text (dual: MIT OR AGPL-3.0)
- Copyright notices
- Redistribution requirements
- Attribution requirements
- IP and trademarks
- Legal safeguards (GDPR/CCPA compliance)

**Length:** ~120 lines

---

## Appendices

### Appendix A: Complete Error Code Reference
- All error codes E001-E008
- Exit code mapping
- Recovery flowcharts

### Appendix B: Revision History
- Version changelog
- Breaking changes
- Migration guides

### Appendix C: Glossary
- Technical terms
- Acronyms
- Domain-specific definitions

**Length:** ~100 lines (appendices)

---

**Total Estimated Length:** ~1,560 lines

**Note:** This is IMPLEMENTATION specification - all requirements, algorithms, schemas, and protocols needed to BUILD CAL. No usage examples or user-facing tutorials - those live in README.md.
