# Multimodal Document Assistant (MM-RAG)

MM-RAG is a Streamlit application for asking grounded questions about PDF documents. It extracts selectable text, OCR text, tables, and embedded images; indexes that evidence in Qdrant; then uses an OpenAI chat model to answer questions with page-level citations.

The UI resumes work when possible: previously parsed artifacts and already-indexed PDFs are detected and reused.

## Features

- Upload a PDF or choose one already in `data/` or `data/uploads/`.
- Extract selectable text, full-page OCR, embedded images and their OCR, and tables.
- Save parsing artifacts, rendered page images, extracted images, JSON records, and Markdown output.
- Split textual page content into chunks while retaining tables and image records as individual retrieval units.
- Create dense OpenAI embeddings and store them in Qdrant using cosine similarity.
- Avoid duplicate/stale vectors when re-indexing a document.
- Send retrieved image evidence to the multimodal model alongside text and table context.
- Return grounded answers with citations such as `[report.pdf p.4]`, source cards, and the images used by the model.

## Architecture

```text
PDF
  |-- selectable text + full-page OCR ----|
  |-- tables -----------------------------|--> LangChain documents --> OpenAI embeddings --> Qdrant
  |-- embedded images + image OCR --------|
                                                               |
Question --> Qdrant retrieval --> text/table context + original retrieved images --> OpenAI chat model --> cited answer
```

Image retrieval is currently based on indexed OCR/text from the image record. When an image is retrieved, its original local file is attached to the chat-model request for visual inspection.

## Prerequisites

- Python 3.12 or later
- An OpenAI API key
- A Qdrant Cloud cluster or another reachable Qdrant instance
- [Tesseract OCR](https://github.com/tesseract-ocr/tesseract) installed and available on `PATH`, or the absolute path to `tesseract.exe`

Tesseract is required for OCR. Without it, parsing can still begin, but OCR-dependent evidence will be unavailable or contain failure markers.

## Setup

Create and activate a virtual environment. These commands use PowerShell:

```powershell
cd "C:\path\to\MM_RAG_10082026"
py -3.12 -m venv env
.\env\Scripts\Activate.ps1
```

Install dependencies:

```powershell
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Alternatively, use `uv`:

```powershell
uv venv env --python 3.12
.\env\Scripts\Activate.ps1
uv pip install -r requirements.txt
```

## Configuration

Copy the sample environment file and fill in the required values:

```powershell
Copy-Item .env.example .env
```

Required variables:

```dotenv
OPENAI_API_KEY=your_openai_api_key
QDRANT_API_KEY=your_qdrant_api_key
QDRANT_Cluster_Endpoint=https://your-cluster-url
```

`QDRANT_URL` is also supported by the source code and takes precedence over `QDRANT_Cluster_Endpoint` when both are set. The remaining settings are optional:

```dotenv
# Defaults shown below
QDRANT_COLLECTION_NAME=mm-rag-documents
OPENAI_CHAT_MODEL=gpt-4.1-mini
OPENAI_EMBEDDING_MODEL=text-embedding-3-large
OPENAI_EMBEDDING_DIMENSION=3072
TESSERACT_PATH=C:\Program Files\Tesseract-OCR\tesseract.exe
```

Keep `.env` private; it is excluded from version control.

## Run the application

With the virtual environment activated, start Streamlit from the project root:

```powershell
streamlit run ui/app.py
```

Then open the local URL printed by Streamlit.

## Using the UI

1. In the sidebar, select an existing PDF or upload a new one.
2. Click **Parse PDF** if reusable parsed artifacts are not already available.
3. Choose a Qdrant collection and click **Ingest to Qdrant** if the document is not already indexed.
4. Adjust the chat model, retrieval `Top-K`, maximum retrieved images, or the current-document filter as needed.
5. Ask a question, or use a suggested prompt. The answer includes evidence sources and any images supplied to the model.

The app fingerprints PDF contents with SHA-256. It uses that fingerprint to restore parsed output and identify previously indexed documents. Re-parse and re-index buttons are available when you want to refresh a document.

## Data and generated artifacts

```text
data/
  uploads/                         # PDFs uploaded through the UI
  parsed_pdf_output/<pdf-name>/
    page_records.json              # selectable text, page OCR, page metadata
    image_records.json             # embedded image paths and OCR text
    table_records.json             # extracted tables in raw, Markdown, and CSV forms
    rag_ready_documents.md         # serialized RAG-ready documents
    extracted_tables.md            # human-readable extracted tables
    extracted_images/              # images extracted from the PDF
    page_images/                   # rendered pages used for OCR
    parse_manifest.json            # source PDF name and SHA-256 fingerprint
```

Generated PDF artifacts can occupy substantial disk space. The repository includes sample documents and outputs for development; avoid committing private source documents or credentials.

## Python modules

| Module | Responsibility |
| --- | --- |
| `ui/app.py` | Streamlit workflow, artifact/index restoration, document inspection, and chat UI. |
| `src/parsing.py` | PDF text/image extraction with PyMuPDF, OCR with Tesseract, table extraction with pdfplumber, and LangChain document creation. |
| `src/ingestion.py` | Chunk preparation, OpenAI embeddings, Qdrant collection/index creation, and document upserts. |
| `src/retriever.py` | Dense Qdrant retrieval with scores and metadata filters. |
| `src/generation.py` | Context assembly, retrieved-image attachment, OpenAI generation, and source metadata preparation. |
| `prompt_library/prompt.py` | Grounding, citation, and suggested-chat prompts. |

## Defaults and behavior

- Text and OCR page documents are chunked at 2,000 characters with 120-character overlap by default; the UI exposes both settings during indexing.
- Tables are stored as Markdown and indexed without text chunking.
- Embedded images are indexed through their OCR/text representation. Up to four retrieved images are attached to a generation request by default.
- The answer generator uses a temperature of `0.0`, a maximum text context of 30,000 characters, and resizes attached images to at most 1,600 pixels on either side.
- Ingestion creates payload indexes for document ID, filename, PDF SHA-256, content type, and page number to support filtering and restoration.

## Command-line smoke tests

The core modules include simple `__main__` examples. After configuring `.env`, invoke them from the project root:

```powershell
python src/parsing.py
python src/ingestion.py
python src/retriever.py
python src/generation.py
```

The examples use the sample PDF path defined in the files. For normal use, prefer the Streamlit workflow.

## Limitations

- OCR and table extraction quality depends on the source PDF and Tesseract/table-detection capabilities.
- Image relevance depends on OCR-extracted image text, not native visual-vector search. A future enhancement could add image summaries or multimodal embeddings.
- Answers are constrained to retrieved context and images, so insufficient retrieval should result in an explicit inability to answer rather than unsupported claims.
