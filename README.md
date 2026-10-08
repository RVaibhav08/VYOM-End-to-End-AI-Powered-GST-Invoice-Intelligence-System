VYOM+ — GST Invoice Intelligence
> **Hackathon Project | Document AI for GST Invoice Processing**
1. Project Name
VYOM+ GST Invoice Intelligence
VYOM+ is a document-intelligence system designed to turn GST invoices and transaction documents into structured, reviewable financial data.
2. Problem Statement
Real-world GST invoices can arrive as Excel, CSV, PDF, JPEG/JPG, or PNG documents. They may contain printed, digitally generated, scanned, or handwritten information.
Manually transferring these documents into accounting systems is repetitive and error-prone. The technical challenge is not only to read text, but to identify what each piece of information represents and determine whether the resulting financial record is internally consistent.
VYOM+ addresses this by building a format-aware pipeline that can:
identify the input format,
choose the appropriate processing path,
extract relevant invoice information,
structure the information consistently,
validate financial relationships,
identify missing or uncertain fields,
and return machine-readable output for downstream workflows.
A particular focus is placed on difficult handwritten and low-quality invoices while maintaining support for printed and digitally generated documents.

3. Project Overview
VYOM+ is a multi-stage document-processing system for GST invoices and transaction documents.
The workflow is designed to:
Accept an invoice or transaction document.
Identify the input format.
Route the document to the appropriate processing pipeline.
Extract text or structured source data.
Use open-weight AI to understand and structure invoice information.
Validate extracted values and identify inconsistencies or uncertainty.
Return standardized JSON and/or tabular financial data.
Present the result through a simple upload and inspection interface.
The architecture deliberately separates format-specific processing from the common AI extraction and validation layers. This keeps the system modular and makes it possible to improve OCR, PDF parsing, model configuration, or validation rules independently.

4. Proposed Solution
VYOM+ uses a format-aware processing architecture:
```text
JPG / PNG  →  Image Processing → OCR ─────────────┐
PDF        →  Text Extraction / OCR ──────────────┤
XLSX / CSV →  Parser / Normalization ─────────────┤
                                                    ▼
                                          Open-Weight AI
                                             (Gemma 4)
                                                    │
                                                    ▼
                                          Structured Invoice
                                                Schema
                                                    │
                                                    ▼
                                           Validation Engine
                                                    │
                                      ┌─────────────┴─────────────┐
                                      ▼                           ▼
                                   VALID                    NEEDS_REVIEW
                                      │                           │
                                      └─────────────┬─────────────┘
                                                    ▼
                                             JSON / Table
                                                    │
                                                    ▼
                                               Inspection UI
```
The central design principle is:
> **OCR reads the document. AI understands the document. Validation checks whether the result makes sense.**
This is intentionally more than a basic OCR demonstration. OCR is used for text recognition where needed, the open-weight AI model performs semantic extraction, and deterministic code performs financial consistency checks.

5. Objectives
The project aims to:
Build an end-to-end GST invoice intelligence pipeline.
Support Excel, CSV, PDF, JPG/JPEG, and PNG inputs.
Automatically identify the input format.
Extract invoice and GST-related information.
Handle printed, digital, scanned, and handwritten documents.
Integrate an open-weight AI model meaningfully into the core workflow.
Convert document content into structured invoice records.
Validate extracted financial information deterministically.
Detect missing, inconsistent, or uncertain information.
Provide machine-readable JSON and structured tabular output.
Provide a simple interface for uploading and inspecting documents.
Keep the architecture modular for future extension.

6. Target Users / Use Case
Target Users
The system is aimed at workflows involving:
accounting and finance teams,
small and medium businesses,
bookkeeping workflows,
invoice-processing teams,
developers building accounting automation systems,
organizations processing larger numbers of GST invoices.
Primary Use Case
A user uploads an invoice. VYOM+ identifies how it should be processed, extracts the relevant information, validates the result, and displays the resulting structured record together with any warnings.
Typical fields include:
seller information,
buyer information,
invoice number,
invoice date,
GSTIN,
product/service details,
quantity,
rate,
amount,
tax information,
subtotal,
total amount.

7. Open-Source AI Technology Selected
Selected Model
Gemma 4 — open-weight AI model
Gemma 4 is intended to act as the main AI document-understanding and structured-extraction component.
Supporting Components
The AI layer is supported by open-source tools including:
Tesseract OCR — text recognition for image-based documents.
Pillow — image loading and preprocessing.
PyMuPDF — PDF text extraction and page processing.
FastAPI — backend API layer.
Python — application and data-processing language.
The exact model configuration can be adjusted during implementation depending on available hardware and benchmark results.

8. Why This Technology Was Selected
The problem requires more than simple text recognition. Invoice documents contain many numbers, dates, identifiers, and tables, and the system must determine the semantic role of each value.
For example, it must distinguish between:
invoice numbers and unrelated numbers,
invoice dates and other dates,
item quantities and monetary amounts,
tax values and totals,
actual values and OCR mistakes,
present fields and genuinely missing information.
An open-weight model is appropriate because the challenge requires meaningful use of open-source/open-weight AI in the technical architecture. Keeping the model in the project stack also gives the team more control over the model configuration and local inference strategy.
Gemma 4 receives normalized OCR or document text and converts that information into the project's standard invoice schema. Its output is then passed to deterministic validation rather than being accepted blindly.

9. AI's Role in the System
AI is a core processing component, not an optional chatbot feature.
Input to the model
Depending on the document type, the model receives:
cleaned OCR text,
extracted PDF text,
normalized spreadsheet information,
and surrounding document context required for field interpretation.
Model task
Gemma 4 is used to:
identify relevant invoice fields,
extract seller and buyer information,
identify invoice number and date,
identify GSTIN and tax information,
extract invoice line items,
structure names, quantities, rates, and amounts,
interpret noisy OCR output,
represent missing values explicitly,
identify uncertain information,
produce schema-conformant structured output.
Model output
The expected output is structured invoice data such as:
```json
{
  "seller": "Example Seller",
  "invoice_number": "INV-001",
  "invoice_date": "2026-10-03",
  "gstin": null,
  "items": [],
  "subtotal": 200,
  "gst_amount": 36,
  "total_amount": 236
}
```
The model is not treated as the final source of truth. Validation code checks numerical and completeness relationships after extraction.

10. System Architecture
```text
                    ┌───────────────────────┐
                    │        User           │
                    │     Uploads File      │
                    └───────────┬───────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │    FastAPI Backend    │
                    │    Input Detection    │
                    └───────────┬───────────┘
                                │
              ┌─────────────────┼─────────────────┐
              │                 │                 │
              ▼                 ▼                 ▼
        ┌───────────┐      ┌───────────┐     ┌────────────┐
        │ JPG/PNG   │      │    PDF    │     │ XLSX/CSV   │
        │  Pipeline │      │  Pipeline │     │  Pipeline  │
        └─────┬─────┘      └─────┬─────┘     └──────┬─────┘
              │                  │                  │
              ▼                  ▼                  ▼
        ┌───────────┐      ┌───────────┐     ┌────────────┐
        │ Tesseract│      │ Text / OCR│     │   Parser   │
        │    OCR    │      │ Processing│     │ & Normalize│
        └─────┬─────┘      └─────┬─────┘     └──────┬─────┘
              │                  │                  │
              └──────────────────┼──────────────────┘
                                 ▼
                    ┌───────────────────────┐
                    │   AI / Model Layer   │
                    │        Gemma 4       │
                    └───────────┬───────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │   Validation Engine   │
                    │ • Required fields     │
                    │ • Numeric checks      │
                    │ • Total checks        │
                    │ • Uncertainty flags  │
                    └───────────┬───────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │   Structured Output   │
                    │     JSON / Table      │
                    └───────────┬───────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │   User Inspection UI  │
                    └───────────────────────┘
```
Model / Tool Interaction
Tesseract and PDF parsers produce document content.
Normalization prepares that content for consistent model input.
Gemma 4 maps the content into the common invoice schema.
Validation code checks the model output.
The backend exposes the final result to the interface.
This separation keeps tool responsibilities explicit and makes failures easier to trace.

11. Component-Level Architecture
1. Upload Layer
Receives supported files and passes them to the backend.
2. Input Detection Layer
Identifies the file type and selects the corresponding processor.
3. Document Processing Layer
Contains format-specific processing components:
image processor,
PDF processor,
Excel processor,
CSV processor.
4. OCR Layer
Uses Tesseract for image-based text recognition where required.
5. AI Extraction Layer
Uses Gemma 4 to interpret document content and populate the standard invoice schema.
6. Validation Layer
Checks:
required fields,
missing information,
numeric values,
line-item totals,
subtotal consistency,
total amount relationships,
potentially uncertain results.
7. Output Layer
Produces standardized JSON and tabular records.
8. User Interface / Backend Interaction
The frontend sends uploaded documents to FastAPI, receives processing results, and displays extracted fields and validation warnings.

12. Data / Information Flow
```text
Input Document
      │
      ▼
Identify File Type
      │
      ├── Image ──► Preprocess ──► OCR
      │
      ├── PDF ───► Text Extraction
      │                │
      │                └── no usable text ──► Render ──► OCR
      │
      └── XLSX/CSV ──► Structured Parser
                         │
                         ▼
                  Clean / Normalize
                         │
                         ▼
                      Gemma 4
                         │
                         ▼
                Structured Invoice Data
                         │
                         ▼
                     Validation
                         │
                  ┌──────┴──────┐
                  ▼             ▼
               Valid       Needs Review
                  │             │
                  └──────┬──────┘
                         ▼
                   JSON / Table
                         │
                         ▼
                    Inspection UI
```
The system preserves the distinction between extracted information and validated information. This prevents uncertain AI output from being silently presented as verified financial data.

13. Agentic Workflow (if applicable)
An agentic workflow is not required for the core version.
The main pipeline is intentionally controlled:
```text
Detect → Process → Extract → Validate → Output
```
This keeps invoice processing predictable and reproducible.
If implementation time allows, an optional routing/review agent may be explored for cases such as:
selecting a specialized processing strategy,
requesting additional processing when extraction quality is poor,
routing uncertain documents for deeper analysis.
Any agentic component remains subordinate to the main document-processing and validation pipeline. It is not a replacement for deterministic validation.

14. Technology Stack
Layer	Technology
Programming Language	Python
Backend	FastAPI
AI Model	Gemma 4
OCR	Tesseract OCR
Image Processing	Pillow
PDF Processing	PyMuPDF
Structured Data Processing	Python data-processing libraries
API / Testing	FastAPI Swagger / OpenAPI
Version Control	Git / GitHub
Output	JSON / Structured Tables
Frontend	Lightweight web interface
The selected components are intentionally modular so they can be replaced or upgraded without redesigning the complete workflow.

15. Expected Features
The planned system should support:
multi-format document upload,
automatic input-type detection,
JPG/JPEG/PNG invoice processing,
PDF invoice processing,
Excel processing,
CSV processing,
OCR-based text extraction,
AI-powered invoice information extraction,
GST-related field extraction,
line-item extraction,
structured JSON output,
structured tabular output,
missing-field detection,
financial consistency checks,
invoice subtotal and line-item validation,
uncertainty and review warnings,
a simple document inspection interface,
modular architecture for future improvements.
Special attention is given to handwritten GST invoices while maintaining support for printed and digitally generated invoices.

16. Implementation Approach
Phase 1 — Backend Foundation
Create the FastAPI backend.
Implement file upload.
Detect input formats.
Establish project structure and configuration.
Phase 2 — Image Invoice Pipeline
Process JPG/JPEG/PNG files.
Preprocess images with Pillow where useful.
Run OCR using Tesseract.
Clean and normalize OCR output.
Phase 3 — Structured AI Extraction
Define a standard invoice JSON schema.
Prompt Gemma 4 for structured extraction.
Handle missing fields without inventing values.
Validate that model output is parseable.
Phase 4 — Validation
Validate required invoice fields.
Calculate line-item totals.
Compare extracted and calculated values.
Detect missing totals and inconsistencies.
Mark records as `valid` or `needs_review`.
Phase 5 — PDF / Excel / CSV Pipelines
Add PDF text extraction.
Add OCR fallback for scanned PDFs.
Parse Excel and CSV files.
Normalize structured information into the common schema.
Phase 6 — User Interface
Build a lightweight upload interface.
Display extracted information.
Display validation warnings.
Allow inspection of structured output.
Phase 7 — Testing and Evaluation
Test with:
printed invoices,
digitally generated invoices,
handwritten invoices,
low-quality images,
scanned PDFs,
incomplete invoices,
inconsistent totals,
different supported file formats.
Deployment / Execution Strategy
For the hackathon, the first target is a reproducible local execution path:
```text
Clone repository
      ↓
Install dependencies
      ↓
Configure model / environment
      ↓
Start FastAPI backend
      ↓
Start frontend
      ↓
Upload document
      ↓
Process and inspect result
```
A production deployment can later be added using a cloud or containerized environment, but it is not a prerequisite for the core qualifier implementation.

17. Expected Final Output
For a successfully processed invoice, the system should return a structured record similar to:
```json
{
  "seller": "Example Seller",
  "buyer": "Example Buyer",
  "invoice_number": "INV-001",
  "invoice_date": "2026-10-03",
  "gstin": "GSTIN_VALUE",
  "items": [
    {
      "name": "Product A",
      "quantity": 2,
      "rate": 100,
      "amount": 200
    }
  ],
  "subtotal": 200,
  "gst_amount": 36,
  "total_amount": 236
}
```
The response should also carry validation state where appropriate:
```json
{
  "status": "needs_review",
  "warnings": [
    "GSTIN was not detected.",
    "Subtotal does not match calculated item total."
  ]
}
```
The final interface should make it clear:
what was extracted,
what was validated,
what is missing or inconsistent,
and what requires human review.
The output is designed to be machine-readable and suitable for downstream accounting workflows.

18. Future Scope / Scalability
Possible extensions include:
stronger handwritten invoice understanding,
vision-language model support,
field-level confidence scores,
human-in-the-loop correction workflows,
automatic re-processing of low-confidence fields,
support for additional financial documents,
database integration,
accounting software integration,
batch invoice processing,
cloud deployment,
scalable asynchronous processing,
model optimization and quantization for local inference,
multilingual and regional invoice support,
improved GST-specific validation rules,
audit logs for document processing and corrections.
The modular architecture is intended to allow individual OCR, AI, validation, and interface components to scale or be replaced independently.

19. Open-Source Dependencies / Components
Component	Purpose
Gemma 4	AI-based document understanding and structured extraction
Tesseract OCR	Text recognition from image-based documents
FastAPI	Backend API
Pillow	Image loading and processing
PyMuPDF	PDF text extraction and page processing
Python ecosystem	Data processing and application development
Git / GitHub	Source control and project hosting
Open-Source AI Integration
The open-source/open-weight AI component is part of the core processing path rather than an optional add-on:
```text
Document
   ↓
OCR / Parser
   ↓
Normalized Content
   ↓
Gemma 4
   ↓
Structured Invoice Schema
   ↓
Deterministic Validation
```
The repository should document the exact model configuration, dependency versions, and applicable licenses used in the final implementation.

20. Expected Challenges and Mitigation
Challenge 1 — Handwritten Text
Problem: Irregular writing, abbreviations, and unclear characters can reduce OCR quality.
Mitigation: Use image preprocessing and OCR strategies suitable for difficult documents, then use contextual AI extraction and review warnings.
Challenge 2 — OCR Errors
Problem: OCR may confuse characters and numbers, especially in low-quality images.
Mitigation: Preserve OCR output for traceability, apply controlled preprocessing, and use contextual extraction rather than relying on individual OCR tokens.
Challenge 3 — Complex Invoice Layouts
Problem: Invoices use different table structures and field positions.
Mitigation: Use semantic extraction rather than depending on one fixed template or fixed coordinates.
Challenge 4 — Missing Information
Problem: Some invoices may not contain buyer details, GSTIN, totals, or other expected fields.
Mitigation: Represent missing values explicitly and generate validation warnings instead of inventing values.
Challenge 5 — Incorrect Financial Values
Problem: OCR or AI extraction may produce numbers that do not agree with line-item calculations.
Mitigation: Implement deterministic checks for line-item totals, subtotals, tax values, and other numeric relationships.
Challenge 6 — Ambiguous AI Output
Problem: AI may interpret noisy document information incorrectly.
Mitigation: Use a fixed schema, structured JSON output, deterministic validation, and a human-review state.
Challenge 7 — Hardware Constraints
Problem: Local open-weight models can require significant computational resources.
Mitigation: Select an appropriate model configuration, optimize inference where possible, and use quantization or a compatible inference strategy where necessary.
Challenge 8 — Different File Types
Problem: Different formats require different processing strategies.
Mitigation: Use modular, format-specific processors with a common downstream extraction and validation layer.

---
Project Status
> **Current stage: Hackathon / Qualifier Proposal**
This README describes the intended technical architecture and implementation direction. During the final hackathon, the exact implementation, model configuration, interface, and dependency versions may evolve based on available time, hardware, test results, document quality, and integration constraints.
The repository should clearly distinguish between what is implemented, what is planned, and what is experimental.
---
Repository Structure
A practical implementation can be organized along these lines:
```text
VYOM+/
├── app/
│   ├── api/
│   ├── processors/
│   ├── ocr/
│   ├── extraction/
│   ├── validation/
│   └── schemas/
│
├── frontend/
├── tests/
├── samples/
├── requirements.txt
├── README.md
└── .gitignore
```
The exact structure may change as implementation progresses.
---
Conclusion
VYOM+ is designed as an end-to-end document-intelligence pipeline rather than a simple OCR demo.
The central workflow is:
Ingest → Understand → Validate → Structure → Review
The architecture combines format-specific document processing, open-weight AI, deterministic validation, and a user-facing inspection layer. This makes the system technically substantial enough for a hackathon while keeping its individual components understandable and independently extensible.
