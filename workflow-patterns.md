# Workflow Patterns & Execution Engine

## 1. The Core Agent Execution Loop
Every interaction must rigidly adhere to the following execution lifecycle. Moving to a subsequent phase without satisfying the prerequisites of the current phase is an operational failure.

[Phase 1: Discover] ──> [Phase 2: Plan & Validate] ──> [Phase 3: Execute]


### Phase 1: Discover & Map
- **Context Gathering:** Before examining source files, the agent must check root configurations (`pyproject.toml`, `requirements.txt`, `.env.example`) to map project boundaries, dependencies, and language versions.
- **Dependency Audit:** If a task requires modifying an existing module, the agent must search for all internal imports referencing that module to map the blast radius of changes.

### Phase 2: Plan & Validate
- **The 3-Bullet Blueprint:** The agent must present a 3-bullet execution plan detailing:
  1. Files to be modified/created.
  2. Potential breaking changes or side effects.
  3. The verification strategy (test command or state checks).
- **Explicit Gate:** The agent must wait for human or runtime environment validation before editing or writing any code block.

### Phase 3: Execute & Verify
- **Atomic Modification:** Edits must be isolated. Do not modify more than 3 files within a single execution cycle. Check in or commit intermediate states before broadening scope.
- **Post-Execution Verification:** Run the associated test suite or static analyzer immediately following code output.

---

## 2. Agent Workflow Restrictions & Banned Behaviors
- **The Blind Refactor:** Modifying functional code blocks without verifying that a supporting test suite exists and runs successfully.
- **The Code Drift:** Rewriting code blocks outside the explicit scope of the active task. Keep changes strictly bounded to the requested feature or bug fix.

---

## 3. Categorized Pattern Catalog (Baseline)
This catalog maps operational design patterns for AI systems, multi-agent frameworks, and Python service boundaries. All implementations must conform to these blueprints.

### Domain A: Agent State & Memory Management

#### [WP-A001] Immutable State Machine
- **Precedence:** Critical
- **Context Scope:** Multi-agent coordination, chat state, runtime step tracking.
- **Description:** Agent state must never be mutated in-place. Every state transition must output a new, frozen instance of the state container, appended to an append-only transaction ledger.

- **Implementation Blueprint:**
```python
from pydantic import BaseModel, Field
from typing import Tuple

class AgentState(BaseModel):
    model_config = {"frozen": True}
    step_id: int
    memory_buffer: Tuple[str, ...] = Field(default_factory=tuple)
    active_tool: str | None = None

def transition_state(current: AgentState, action_result: str, next_tool: str | None) -> AgentState:
    # Explicitly returns a new instance, keeping the history pure
    return AgentState(
        step_id=current.step_id + 1,
        memory_buffer=current.memory_buffer + (action_result,),
        active_tool=next_tool
    )
```

#### [WP-A002] Context Window Decoupling
Precedence: High
Context Scope: RAG pipelining, long-context ingestion.
Description: Ban raw text passing directly into prompts. Extract token-heavy structural data into semantic chunks or indexed vector IDs, passing only metadata summaries to primary agent reasoning loops.
Domain B: Tool Execution & MCP Integration

#### [WP-B001] Idempotent Tool Execution
Precedence: Critical
Context Scope: File I/O, Database writes, API calls via Model Context Protocol (MCP).
Description: Every agent tool execution modifying system state must inject a unique idempotency_key (UUIDv4) into the transaction boundary to completely block duplicate execution loops.
Implementation Blueprint:

import uuid
from typing import Protocol

class IdempotentStorage(Protocol):
    def has_seen(self, key: uuid.UUID) -> bool: ...
    def mark_seen(self, key: uuid.UUID) -> None: ...

async def execute_system_change(tool_data: dict, token: uuid.UUID, storage: IdempotentStorage) -> dict:
    if storage.has_seen(token):
        return {"status": "skipped", "reason": "Duplicate transaction block triggered."}
    
    storage.mark_seen(token)
    # Process tool logic down here...
    return {"status": "success"}

#### [WP-B002] Multi-Agent Circuit Breaker
Precedence: High
Context Scope: Autonomous execution loops.
Description: Stop runaway loops. Tool calls executing sequentially without changing the system state delta must force-trip a circuit breaker after 5 consecutive cycles, throwing a hard operational refusal.



***

Now that you have the baseline for this file saved, do you want to move to **Phase 3** and generate the `.cursorrules` enforcement engine file, or should we pitch more patterns to expand this catalog right away?