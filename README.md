# VeriGenius

### AI-Assisted UVM Verification Environment Generator

VeriGenius is a web-based interface that takes a DUT specification and generates a structured **UVM verification environment blueprint**.

The frontend is built with **Streamlit** and communicates with a separate FastAPI backend that runs the LangGraph-based generation workflow.

---

## Overview

Designing a UVM verification environment from a DUT specification requires identifying interfaces, transactions, sequences, agents, tests, coverage, and other verification components.

VeriGenius automates this initial planning stage.

The user provides a DUT specification through the web interface, and VeriGenius sends it to the backend generation workflow. The generated verification artifacts are then displayed and made available for download.

### Workflow

```text
DUT Specification
       │
       ▼
┌──────────────────┐
│  VeriGenius UI   │
│    Streamlit     │
└────────┬─────────┘
         │
         │ HTTP POST
         ▼
┌──────────────────┐
│   FastAPI API    │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ LangGraph        │
│ Generation       │
│ Workflow         │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ UVM Blueprint    │
└──────────────────┘
```

---

## Features

* Upload DUT specifications in:

  * PDF
  * DOCX
  * Markdown
* Paste specifications directly into the interface
* Extract specification text before generation
* Generate a structured UVM verification blueprint
* Display generated SystemVerilog files in the browser
* Download individual generated artifacts
* Download all generated artifacts as a ZIP
* Review gap-analysis information
* Submit feedback for future amendment/refinement stages

---

## Supported Input

### PDF

Text is extracted from PDF pages before being sent to the backend.

### DOCX

Paragraphs and tables are extracted from Word documents.

### Markdown

Markdown specifications are passed directly to the generation pipeline.

### Manual Input

A specification can also be pasted directly into the text area.

---

## Generated Artifacts

Depending on the specification and backend workflow, VeriGenius can produce artifacts such as:

```text
filled_boilerplate.sv
structural_uvc.sv
uvm_sequences.sv
uvm_tests.sv
symbol_registry.json
gap_analysis.md
```

### Example

```text
Generated UVM Files

├── filled_boilerplate.sv
├── structural_uvc.sv
├── uvm_sequences.sv
├── uvm_tests.sv
├── symbol_registry.json
└── gap_analysis.md
```

Each generated file can be previewed in the browser and downloaded individually.

The complete set can also be downloaded as:

```text
VeriGenius_Output.zip
```

---

## Frontend Architecture

The frontend is implemented using **Streamlit**.

```text
frontend/
└── app.py
```

The application is responsible for:

1. Receiving the DUT specification.
2. Extracting text from PDF/DOCX/Markdown files.
3. Validating the extracted specification.
4. Sending the specification to the backend API.
5. Receiving the generated blueprint.
6. Displaying generated artifacts.
7. Providing individual and ZIP downloads.
8. Collecting user feedback.

The backend generation logic is maintained separately from this public frontend repository.

---

## Backend

The frontend communicates with a separate FastAPI backend.

```text
Streamlit Frontend
        │
        │ POST /generate
        ▼
FastAPI Backend
        │
        ▼
LangGraph Workflow
        │
        ▼
Generated UVM Blueprint
```

The backend repository is maintained separately.

---

## Local Setup

### 1. Clone the repository

```bash
git clone https://github.com/Alien-is-here/verigenius.git
cd verigenius
```

### 2. Create a virtual environment

Windows:

```powershell
python -m venv venv
venv\Scripts\activate
```

Linux/macOS:

```bash
python -m venv venv
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Start the backend

The backend must be running separately.

For local development:

```bash
python -m uvicorn backend.api:app --reload
```

The API will be available at:

```text
http://127.0.0.1:8000
```

Health check:

```text
http://127.0.0.1:8000/health
```

### 5. Start the Streamlit frontend

In a second terminal:

```bash
streamlit run frontend/app.py
```

The Streamlit application will open at:

```text
http://localhost:8501
```

---

## API Communication

The frontend sends the extracted specification to:

```text
POST /generate
```

The specification is sent as a Markdown file regardless of the original input format.

For example:

```python
response = requests.post(
    f"{BACKEND_URL}/generate",
    files={
        "file": (
            "specification.md",
            specification.encode("utf-8"),
            "text/markdown"
        )
    },
    timeout=300
)
```

This allows PDF and DOCX files to be converted to text by the frontend before entering the backend workflow.

---

## Project Structure

```text
verigenius/
│
├── frontend/
│   └── app.py
│
├── docs/
│   ├── architecture.md
│   ├── workflow.md
│   └── usage.md
│
├── examples/
│   ├── fifo_spec.md
│   └── sample_output/
│
├── screenshots/
│
├── README.md
├── requirements.txt
└── .gitignore
```

---

## Example Use Case

A user provides a specification such as:

```text
Module: simple_fifo

DATA_WIDTH = 8
DEPTH = 16

Inputs:
- clk
- rst_n
- wr_en
- rd_en
- data_in

Outputs:
- data_out
- full
- empty
```

VeriGenius processes the specification and produces a structured verification blueprint containing relevant UVM components and verification artifacts.

---

## Design Goal

VeriGenius is intended to assist verification engineers during the **initial UVM environment planning and generation stage**.

The generated output should be treated as a starting point for engineering review rather than as an automatically verified final verification environment.

Human review and validation remain part of the verification process.

---

## Technology

### Frontend

* Python
* Streamlit
* Requests
* PyPDF2
* python-docx

### Backend

* Python
* FastAPI
* LangGraph
* LLM-based generation workflow

---

## Project Status

Current capabilities include:

* [x] Streamlit web interface
* [x] PDF specification input
* [x] DOCX specification input
* [x] Markdown specification input
* [x] Manual specification input
* [x] FastAPI communication
* [x] UVM blueprint generation
* [x] Generated artifact preview
* [x] Individual file downloads
* [x] ZIP download
* [x] Feedback collection

### Planned Improvements

* [ ] Feedback-driven amendment workflow
* [ ] Improved specification validation
* [ ] Additional UVM output components
* [ ] Deployment of the complete frontend/backend system
* [ ] Expanded example specifications and generated blueprints

---

## Author

**Alina Naveed**

GitHub: [Alien-is-here](https://github.com/Alien-is-here)

---

## License

See the repository license for usage and distribution terms.
