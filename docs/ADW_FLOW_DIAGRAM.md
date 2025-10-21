# ADW Agent Flow Diagram

## Complete File Flow for `adw_plan_build_test.py adw`

This document provides a comprehensive visualization of all MD (Markdown) and Python files called when executing the ADW (Anthropic Defined Workflow) system.

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                   ENTRY POINT                                    │
│           adws/adw_plan_build_test.py                           │
│                                                                  │
│  Orchestrates 3 sequential phases via subprocess.run()          │
└─────────────────────────────────────────────────────────────────┘
                           │
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
    ┌──────────┐    ┌──────────┐    ┌──────────┐
    │  PHASE 1 │    │  PHASE 2 │    │  PHASE 3 │
    │   PLAN   │───▶│  BUILD   │───▶│   TEST   │
    └──────────┘    └──────────┘    └──────────┘
```

---

## Phase 1: Planning Phase

### Entry Point
**Python File:** `adws/adw_plan.py`

### Python Modules Imported

| Module | Path | Purpose |
|--------|------|---------|
| `state.py` | `adw_modules/state.py` | State management (load/save JSON) |
| `git_ops.py` | `adw_modules/git_ops.py` | Git operations (branch, commit, push) |
| `github.py` | `adw_modules/github.py` | GitHub API operations |
| `workflow_ops.py` | `adw_modules/workflow_ops.py` | Core workflow functions |
| `utils.py` | `adw_modules/utils.py` | Utility functions (logging, etc.) |
| `data_types.py` | `adw_modules/data_types.py` | Pydantic data models |

### Execution Flow

```
┌──────────────────────────────────────────────────────────────────┐
│ 1. Initialize                                                     │
│    - load_dotenv()                                                │
│    - ensure_adw_id() → workflow_ops.py                           │
│    - ADWState.load() → state.py                                  │
│    - setup_logger() → utils.py                                   │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│ 2. Get Repository Info                                            │
│    - get_repo_url() → github.py                                  │
│    - extract_repo_path() → github.py                             │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│ 3. Fetch GitHub Issue                                             │
│    - fetch_issue() → github.py                                   │
│      Returns: GitHubIssue object (data_types.py)                 │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│ 4. Classify Issue                                                 │
│    - classify_issue() → workflow_ops.py                          │
│    - execute_template() → agent.py                               │
│                                                                   │
│    MD FILE: .claude/commands/classify_issue.md                   │
│    Input: GitHub Issue JSON                                      │
│    Output: /chore, /bug, or /feature                            │
│                                                                   │
│    State Update: issue_class = "/feature"                        │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│ 5. Generate Branch Name                                           │
│    - generate_branch_name() → workflow_ops.py                    │
│    - execute_template() → agent.py                               │
│                                                                   │
│    MD FILE: .claude/commands/generate_branch_name.md             │
│    Input: issue_type, adw_id, Issue JSON                        │
│    Output: feature-issue-adw-{id}-{name}                        │
│                                                                   │
│    Git Operation: create_branch() → git_ops.py                   │
│    State Update: branch_name = "..."                             │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│ 6. Build Implementation Plan                                      │
│    - build_plan() → workflow_ops.py                              │
│    - execute_template() → agent.py                               │
│                                                                   │
│    MD FILE (one of):                                             │
│      - .claude/commands/feature.md (for /feature)                │
│      - .claude/commands/bug.md (for /bug)                        │
│      - .claude/commands/chore.md (for /chore)                    │
│                                                                   │
│    Input: issue_number, adw_id, Issue JSON                       │
│    Output: Creates markdown plan file in specs/                  │
│            specs/issue-{num}-adw-{id}-sdlc_planner-{name}.md    │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│ 7. Find Plan File                                                 │
│    - get_plan_file() → workflow_ops.py                           │
│    - execute_template() → agent.py                               │
│                                                                   │
│    MD FILE: .claude/commands/find_plan_file.md                   │
│    Input: issue_number, adw_id, plan_output text                 │
│    Output: File path to created plan                             │
│                                                                   │
│    State Update: plan_file = "specs/issue-..."                   │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│ 8. Create Commit                                                  │
│    - create_commit() → workflow_ops.py                           │
│    - execute_template() → agent.py                               │
│                                                                   │
│    MD FILE: .claude/commands/commit.md                           │
│    Input: agent_name (sdlc_planner), issue_type, Issue JSON      │
│    Output: Commit message                                         │
│                                                                   │
│    Git Operations:                                                │
│      - commit_changes() → git_ops.py                             │
│      - finalize_git_operations() → git_ops.py                    │
│                                                                   │
│    MD FILE: .claude/commands/pull_request.md                     │
│    Creates/updates PR                                             │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ▼
                   State saved to:
         agents/{adw_id}/adw_state.json
```

---

## Phase 2: Build/Implementation Phase

### Entry Point
**Python File:** `adws/adw_build.py`

### Python Modules Imported

| Module | Path | Purpose |
|--------|------|---------|
| `state.py` | `adw_modules/state.py` | Load existing state |
| `git_ops.py` | `adw_modules/git_ops.py` | Commit and push operations |
| `github.py` | `adw_modules/github.py` | Fetch issue, post comments |
| `workflow_ops.py` | `adw_modules/workflow_ops.py` | implement_plan(), create_commit() |
| `utils.py` | `adw_modules/utils.py` | Logging |
| `data_types.py` | `adw_modules/data_types.py` | Data models |

### Execution Flow

```
┌──────────────────────────────────────────────────────────────────┐
│ 1. Load Existing State                                            │
│    - ADWState.load() → state.py                                  │
│    - Loads: branch_name, plan_file, issue_class                  │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│ 2. Checkout Branch                                                │
│    - subprocess.run(["git", "checkout", branch_name])            │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│ 3. Implement Plan                                                 │
│    - implement_plan() → workflow_ops.py                          │
│    - execute_template() → agent.py                               │
│                                                                   │
│    MD FILE: .claude/commands/implement.md                        │
│    Input: plan_file path                                         │
│    Output: Implementation summary                                 │
│                                                                   │
│    Actions: Reads plan, implements all tasks in codebase         │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│ 4. Create Commit                                                  │
│    - fetch_issue() → github.py                                   │
│    - create_commit() → workflow_ops.py                           │
│    - execute_template() → agent.py                               │
│                                                                   │
│    MD FILE: .claude/commands/commit.md                           │
│    Input: agent_name (sdlc_implementor), issue_type, Issue JSON  │
│    Output: Commit message                                         │
│                                                                   │
│    Git Operations:                                                │
│      - commit_changes() → git_ops.py                             │
│      - finalize_git_operations() → git_ops.py                    │
│                                                                   │
│    Updates existing PR                                            │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ▼
                   State saved to:
         agents/{adw_id}/adw_state.json
```

---

## Phase 3: Testing Phase

### Entry Point
**Python File:** `adws/adw_test.py`

### Python Modules Imported

| Module | Path | Purpose |
|--------|------|---------|
| `agent.py` | `adw_modules/agent.py` | execute_template() |
| `data_types.py` | `adw_modules/data_types.py` | TestResult, E2ETestResult models |
| `github.py` | `adw_modules/github.py` | Issue comments |
| `utils.py` | `adw_modules/utils.py` | Logging, JSON parsing |
| `state.py` | `adw_modules/state.py` | State management |
| `git_ops.py` | `adw_modules/git_ops.py` | Commit and push |
| `workflow_ops.py` | `adw_modules/workflow_ops.py` | create_commit(), classify_issue() |

### Execution Flow

```
┌──────────────────────────────────────────────────────────────────┐
│ 1. Load State & Checkout Branch                                  │
│    - ADWState.load() → state.py                                  │
│    - subprocess.run(["git", "checkout", branch_name])            │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│ 2. Run Unit Tests (with retry loop - up to 4 attempts)           │
│    - run_tests_with_resolution() → adw_test.py                   │
│    - run_tests() → adw_test.py                                   │
│    - execute_template() → agent.py                               │
│                                                                   │
│    MD FILE: .claude/commands/test.md                             │
│    Input: None                                                    │
│    Output: JSON array of TestResult objects                      │
│                                                                   │
│    Runs:                                                          │
│      - Backend: uv run pytest tests/ -v --tb=short               │
│      - Frontend: npx tsc --noEmit                                │
│      - Frontend: npm run build                                   │
└──────────────────────────────────────────────────────────────────┘
                           │
                    ┌──────┴───────┐
                    │ Tests Failed? │
                    └──────┬───────┘
                           │ Yes (and attempts < 4)
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│ 3. Resolve Failed Tests (per test)                               │
│    - resolve_failed_tests() → adw_test.py                        │
│    - execute_template() → agent.py                               │
│                                                                   │
│    MD FILE: .claude/commands/resolve_failed_test.md              │
│    Input: Failed test JSON                                       │
│    Output: Fix summary                                            │
│                                                                   │
│    Actions: Analyzes failure, fixes code, re-runs                │
│    Loop back to step 2 if fixes were made                        │
└──────────────────────────────────────────────────────────────────┘
                           │
                    ┌──────┴───────┐
                    │ Tests Passed? │
                    └──────┬───────┘
                           │ Yes
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│ 4. Run E2E Tests (with retry loop - up to 2 attempts)            │
│    - run_e2e_tests_with_resolution() → adw_test.py               │
│    - run_e2e_tests() → adw_test.py                               │
│    - execute_single_e2e_test() → adw_test.py                     │
│    - execute_template() → agent.py                               │
│                                                                   │
│    MD FILE: .claude/commands/test_e2e.md                         │
│    Input: adw_id, agent_name, test_file                          │
│    Output: E2ETestResult JSON                                    │
│                                                                   │
│    E2E Test Files (in .claude/commands/e2e/):                    │
│      - test_basic_query.md                                       │
│      - test_complex_query.md                                     │
│      - test_random_query.md                                      │
│      - test_input_disable_debounce.md                            │
│      - test_sql_injection.md                                     │
│                                                                   │
│    Tests run sequentially, stop on first failure                 │
└──────────────────────────────────────────────────────────────────┘
                           │
                    ┌──────┴───────┐
                    │ E2E Failed?   │
                    └──────┬───────┘
                           │ Yes (and attempts < 2)
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│ 5. Resolve Failed E2E Tests (per test)                           │
│    - resolve_failed_e2e_tests() → adw_test.py                    │
│    - execute_template() → agent.py                               │
│                                                                   │
│    MD FILE: .claude/commands/resolve_failed_e2e_test.md          │
│    Input: Failed E2E test JSON                                   │
│    Output: Fix summary                                            │
│                                                                   │
│    Actions: Analyzes failure, fixes code, re-runs                │
│    Loop back to step 4 if fixes were made                        │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│ 6. Commit Test Results                                            │
│    - create_commit() → workflow_ops.py                           │
│    - execute_template() → agent.py                               │
│                                                                   │
│    MD FILE: .claude/commands/commit.md                           │
│    Input: agent_name (test_runner), issue_type, Issue JSON       │
│    Output: Commit message                                         │
│                                                                   │
│    Git Operations:                                                │
│      - commit_changes() → git_ops.py                             │
│      - finalize_git_operations() → git_ops.py                    │
│                                                                   │
│    Updates PR with test results                                   │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│ 7. Log Results to GitHub                                          │
│    - log_test_results() → adw_test.py                            │
│    - make_issue_comment() → github.py                            │
│                                                                   │
│    Posts comprehensive test summary to GitHub issue               │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ▼
                Exit with code 0 (pass) or 1 (fail)
```

---

## Complete File Inventory

### Python Files

#### Main Entry Scripts
```
adws/
├── adw_plan_build_test.py  # Orchestrator - runs all 3 phases
├── adw_plan.py             # Phase 1: Planning
├── adw_build.py            # Phase 2: Implementation
└── adw_test.py             # Phase 3: Testing
```

#### Python Modules
```
adw_modules/
├── agent.py         # Claude Code CLI integration, JSONL parsing
├── data_types.py    # Pydantic models (GitHubIssue, TestResult, etc.)
├── github.py        # GitHub API operations
├── git_ops.py       # Git commands (branch, commit, push)
├── state.py         # State management (JSON file I/O)
├── workflow_ops.py  # Core workflow functions (classify, build_plan, etc.)
└── utils.py         # Utility functions (logging, JSON parsing)
```

### MD Command Files

#### Core Workflow Commands
```
.claude/commands/
├── classify_issue.md          # Classify issue as /chore, /bug, or /feature
├── feature.md                 # Create implementation plan for features
├── bug.md                     # Create implementation plan for bugs
├── chore.md                   # Create implementation plan for chores
├── generate_branch_name.md    # Generate git branch name
├── find_plan_file.md          # Locate the created plan file
├── implement.md               # Implement the plan from Phase 1
├── commit.md                  # Generate commit messages
└── pull_request.md            # Create/update pull requests
```

#### Testing Commands
```
.claude/commands/
├── test.md                    # Run unit test suite
├── resolve_failed_test.md     # Fix failed unit tests
├── test_e2e.md                # Run E2E tests
└── resolve_failed_e2e_test.md # Fix failed E2E tests
```

#### E2E Test Definitions
```
.claude/commands/e2e/
├── test_basic_query.md           # Basic query functionality
├── test_complex_query.md         # Complex query scenarios
├── test_random_query.md          # Random query generation
├── test_input_disable_debounce.md # Input debouncing behavior
└── test_sql_injection.md         # SQL injection prevention
```

### State & Output Files

#### State Management
```
agents/{adw_id}/
├── adw_state.json              # Persistent state across phases
│                               # Contains: adw_id, issue_number, branch_name,
│                               #           plan_file, issue_class
```

#### Agent Outputs
```
agents/{adw_id}/
├── sdlc_planner/               # Phase 1 outputs
│   ├── raw_output.jsonl        # Claude Code JSONL output
│   ├── raw_output.json         # Converted JSON
│   └── prompts/
│       └── feature.txt         # Expanded prompt
│
├── sdlc_implementor/           # Phase 2 outputs
│   ├── raw_output.jsonl
│   ├── raw_output.json
│   └── prompts/
│       └── implement.txt
│
└── test_runner/                # Phase 3 outputs
    ├── raw_output.jsonl
    ├── raw_output.json
    └── prompts/
        └── test.txt
```

#### Plan Files
```
specs/
└── issue-{number}-adw-{id}-sdlc_planner-{name}.md  # Implementation plan
```

---

## Data Flow Summary

### State Persistence
The `adw_state.json` file is the central mechanism for passing data between phases:

```json
{
  "adw_id": "abc12345",
  "issue_number": "123",
  "branch_name": "feature-issue-123-adw-abc12345-user-auth",
  "plan_file": "specs/issue-123-adw-abc12345-sdlc_planner-user-auth.md",
  "issue_class": "/feature"
}
```

### Phase Dependencies
```
Phase 1 (Plan) → Creates:
  - branch_name
  - plan_file
  - issue_class
  ↓
Phase 2 (Build) → Requires:
  - branch_name (to checkout)
  - plan_file (to implement)
  ↓
Phase 3 (Test) → Requires:
  - branch_name (to checkout)
  Optional: issue_class (for commit)
```

---

## Key Python Functions by Module

### workflow_ops.py
- `ensure_adw_id()` - Creates/finds ADW ID
- `classify_issue()` - Calls /classify_issue
- `generate_branch_name()` - Calls /generate_branch_name
- `build_plan()` - Calls /feature, /bug, or /chore
- `get_plan_file()` - Calls /find_plan_file
- `implement_plan()` - Calls /implement
- `create_commit()` - Calls /commit
- `format_issue_message()` - Formats GitHub comments

### agent.py
- `execute_template()` - Core function that:
  - Builds Claude Code CLI command
  - Executes slash command
  - Captures JSONL output
  - Parses and returns result

### git_ops.py
- `create_branch()` - Creates and checks out git branch
- `commit_changes()` - Stages and commits changes
- `finalize_git_operations()` - Pushes and creates/updates PR

### github.py
- `get_repo_url()` - Gets GitHub repo URL from git remote
- `extract_repo_path()` - Extracts owner/repo from URL
- `fetch_issue()` - Fetches GitHub issue via CLI
- `make_issue_comment()` - Posts comment to GitHub issue

### state.py
- `ADWState.load()` - Loads state from JSON file
- `state.update()` - Updates state fields
- `state.save()` - Saves state to JSON file
- `state.get()` - Gets state field value

---

## Execution Example

For command: `uv run adw_plan_build_test.py adw`

### Files Executed in Order:

1. **adw_plan_build_test.py** - Entry point
2. **workflow_ops.py** - ensure_adw_id()
3. **adw_plan.py** - PHASE 1 START
   - state.py - Load state
   - utils.py - Setup logger
   - github.py - Get repo, fetch issue
   - workflow_ops.py → agent.py → **classify_issue.md**
   - workflow_ops.py → agent.py → **generate_branch_name.md**
   - git_ops.py - Create branch
   - workflow_ops.py → agent.py → **feature.md** (creates plan)
   - workflow_ops.py → agent.py → **find_plan_file.md**
   - workflow_ops.py → agent.py → **commit.md**
   - git_ops.py - Commit and push
   - git_ops.py → agent.py → **pull_request.md**
   - state.py - Save state
4. **adw_build.py** - PHASE 2 START
   - state.py - Load state
   - git_ops.py - Checkout branch
   - workflow_ops.py → agent.py → **implement.md**
   - github.py - Fetch issue
   - workflow_ops.py → agent.py → **commit.md**
   - git_ops.py - Commit and push
   - state.py - Save state
5. **adw_test.py** - PHASE 3 START
   - state.py - Load state
   - git_ops.py - Checkout branch
   - Loop (up to 4 times):
     - agent.py → **test.md**
     - If failures: agent.py → **resolve_failed_test.md**
   - If tests pass, E2E loop (up to 2 times):
     - For each test file:
       - agent.py → **test_e2e.md** → **test_basic_query.md**
       - agent.py → **test_e2e.md** → **test_complex_query.md**
       - agent.py → **test_e2e.md** → **test_random_query.md**
       - (etc.)
     - If failures: agent.py → **resolve_failed_e2e_test.md**
   - workflow_ops.py → agent.py → **commit.md**
   - git_ops.py - Commit and push
   - github.py - Post test results
   - state.py - Save state

---

## Summary

### Total Files Involved

**Python Files:** 11
- 4 main scripts (adw_plan_build_test.py, adw_plan.py, adw_build.py, adw_test.py)
- 7 modules (agent, data_types, github, git_ops, state, workflow_ops, utils)

**MD Command Files:** 18
- 9 core workflow commands
- 4 testing commands
- 5 E2E test definitions

**State Files:** 1
- agents/{adw_id}/adw_state.json

**Output Files:** Multiple
- Raw JSONL/JSON outputs per agent
- Plan markdown files in specs/
- Execution logs

### Key Integration Points

1. **agent.py** is the bridge between Python and Claude Code MD commands
2. **state.py** is the bridge between phases (Plan → Build → Test)
3. **workflow_ops.py** contains the core workflow logic used by all phases
4. **git_ops.py** handles all git operations (branch, commit, push, PR)
5. **github.py** handles all GitHub interactions (fetch issue, post comments)

This architecture enables a fully automated development workflow where each phase can run independently or as part of a complete pipeline.
