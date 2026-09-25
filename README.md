# VeriGenius phase 1: Generic Single-UVC UVM Environment Generator

VeriGenius Phase 1 implements a **specification-driven generator** that automatically constructs a complete **Single-UVC UVM (Universal Verification Methodology) Verification Environment** directly from a hardware specification document (`spec_document.docx`).

---

## 🎯 Phase 1 Core Claims & Architectural Principles

1. **Purely Specification-Driven**: The only required input from the user is `spec_document.docx`.
2. **Zero DUT-Specific Logic (Strict Genericity)**:
   - 0 hardcoded signal names
   - 0 DUT-specific branches or heuristics (`if counter:`, `if fifo:`, `if hadamard:`)
   - 0 prompt biases or domain-specific fallbacks
3. **Single Contract — Canonical Symbol Registry**:
   - Node 01b extracts and normalizes all interface signals, clock, reset, timing model, and transactions into `symbol_registry.json`.
   - Every downstream generator node derives code strictly from this registry.
4. **Generic Timing Models Supported**:
   - **Synchronous Single-Cycle**: Clock-synchronized sampling.
   - **Streaming Handshake**: Valid/ready transfer acceptance.
   - **Completion Multi-Cycle**: Dynamic transaction correlation queue (`in_q[$]`).
5. **Structural Scoreboard Only**:
   - Observes transaction counts and validates analysis-port TLM connectivity (`report_phase`).
   - Algorithmic/golden reference model prediction is intentionally excluded (Phase 2 milestone).
6. **RTL Optional**:
   - Generates complete, syntactically clean UVM environments without requiring RTL.
   - If RTL is provided, emits top-level binding wrapper (`tb_top.sv`) and simulator manifest (`file.f`).

---

## 📦 Deliverables Produced

For any arbitrary DUT specification, VeriGenius produces:
- `symbol_registry.json` : Canonical symbol registry (signals, widths, timing model, transactions)
- `structural_uvc.sv`    : Interface, Sequence Item, Driver, Monitor, Sequencer, Agent, Scoreboard, Env
- `uvm_sequences.sv`     : Generic directed, random, corner-case, and idle sequence library
- `uvm_tests.sv`         : Base test, Smoke test, Regression test, Stress test
- `file.f`               : Simulator compilation manifest
- `tb_top.sv`            : Top-level wrapper (when RTL is provided)

---

## 🛠️ Prerequisites & Setup

### 1. Python Environment
- **Python 3.10+** (Python 3.11 recommended)
- Install dependencies:
  ```bash
  pip install -r requirements.txt
  ```

### 2. LLM API Key Configuration
VeriGenius uses an LLM provider (Groq or OpenRouter) for natural-language extraction and normalization.

Set your API key in **`key.txt`**:
```text
groq_key=gsk_your_groq_api_key_here
```
*Or set via environment variable:*
```bash
export GROQ_API_KEY="gsk_your_groq_api_key_here"
```

---

## 🚀 How to Run with Your Own DUT Specification

### Option A: Command-Line Arguments (Recommended)
You can point the pipeline to any `.docx` specification document anywhere on your filesystem:

```bash
# 1. Spec only (Generates UVC, Sequences, Tests, Registry)
python uvm_workflow.py --spec /path/to/your_spec.docx --out ./my_dut_output

# 2. Spec + RTL (Also generates tb_top.sv and simulator manifest)
python uvm_workflow.py --spec /path/to/your_spec.docx --rtl /path/to/your_dut.sv --out ./my_dut_output
```

### Option B: Drop-in File
Place your specification file as `spec_document.docx` in the `files_project/` directory and execute:
```bash
python uvm_workflow.py
```
All artifacts will be generated directly in the current directory.

---

## 🔍 Independent Genericity Verification

To independently verify that the generator contains **ZERO** DUT-specific cheating, hardcoded signal names, or branches:

```bash
python check_genericity.py
```
**Expected Output:**
```text
=================================================================
VERIGENIUS PHASE 1: STATIC GENERICITY GUARD AUDIT
=================================================================
  [PASSED (0 leaks)    ] node_01_extractor.py
  [PASSED (0 leaks)    ] node_01b_symbol_registry.py
  [PASSED (0 leaks)    ] node_02_gap_checker.py
  [PASSED (0 leaks)    ] node_03a_struct_filler.py
  [PASSED (0 leaks)    ] node_03b_sequence_library.py
  [PASSED (0 leaks)    ] node_03c_test_generator.py
  [PASSED (0 leaks)    ] node_04a_scoreboard_enhancer.py
  [PASSED (0 leaks)    ] node_04c_lint.py
  [PASSED (0 leaks)    ] node_05_validator.py
  [PASSED (0 leaks)    ] assembly_emitter.py
  [PASSED (0 leaks)    ] spec_consistency_checker.py
  [PASSED (0 leaks)    ] uvm_workflow.py
=================================================================
GENERICITY GUARD: PASS — 0 DUT-specific generator branches or leaks found.
=================================================================
```

---

## 🔬 Reproducing Benchmark Experiments (Counter, FIFO, Hadamard)

We provide three reference validation experiments in the `experiments/` directory:
- `experiments/counter/` (Synchronous 4-bit Counter)
- `experiments/fifo/`    (Synchronous FIFO)
- `experiments/hadamard/`(8x8 Multi-cycle Transform Unit)

Run all three sequentially through the exact same generator pipeline:
```bash
python ../experiments/run_experiments.py
```
All three generate complete UVM environments and compile with **Exit code 0** under Cadence Xcelium 23.09.

---

## 💻 Simulating Generated UVM Environments

### Cadence Xcelium:
```bash
cd <output_directory>
xrun -uvm -compile structural_uvc.sv uvm_sequences.sv uvm_tests.sv
```
*(Or if RTL was supplied: `xrun -uvm -f file.f +UVM_TESTNAME=smoke_test`)*

### Synopsys VCS:
```bash
cd <output_directory>
vcs -sverilog -ntb_opts uvm structural_uvc.sv uvm_sequences.sv uvm_tests.sv
```

### Siemens Questa / ModelSim:
```bash
cd <output_directory>
vlog +acc structural_uvc.sv uvm_sequences.sv uvm_tests.sv
```

---

## 🏛️ Pipeline Node Architecture

```
spec_document.docx
       │
       ▼
[Node 01: Extractor] ────────► Natural Language Hardware Facts
       │
       ▼
[Node 01b: Registry] ────────► Canonical Symbol Registry (symbol_registry.json)
       │
       ├─────────────────────────┬─────────────────────────┐
       ▼                         ▼                         ▼
[Node 03a: UVC]           [Node 03b: Seqs]          [Node 03c: Tests]
structural_uvc.sv         uvm_sequences.sv          uvm_tests.sv
       │                         │                         │
       └─────────────────────────┼─────────────────────────┘
                                 │
                                 ▼
                     [Node 04a: Scoreboard Pass-Through]
                                 │
                                 ▼
                     [Node 04c: IEEE 1800.2 Deterministic Lint]
                                 │
                                 ▼
                     [Node 05: Structural Validator]
                                 │
                                 ▼
                     [Assembly Emitter] ──► Output Deliverables & Manifest
```
