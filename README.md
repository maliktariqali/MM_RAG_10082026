# Multimodal RAG

This project is a Multimodal Retrieval-Augmented Generation (RAG) application. It is intended to ingest and retrieve information from multiple content types, such as text, images, tables, or documents, and use the relevant retrieved context to generate grounded answers with a large language model.

## Prerequisites

- [Python 3.12](https://www.python.org/downloads/)
- [uv](https://docs.astral.sh/uv/), used to manage Python versions, the virtual environment, and project packages
- Command Prompt (`cmd`) on Windows

## Environment setup

Open **Command Prompt** and move into the project directory:

```cmd
cd /d "C:\Personal\Python\Krish Naik Gen AI Bootcamp\Projects\MM_RAG_10082026"
```

List the Python versions available through `uv`:

```cmd
uv python list
```

Create the virtual environment with Python 3.12:

```cmd
uv venv env --python 3.12
```

Activate the environment in Command Prompt:

```cmd
env\Scripts\activate.bat
```

After activation, `(env)` appears at the beginning of the command prompt.

## Install dependencies

Install the packages listed in `requirement.txt`:

```cmd
uv pip install -r requirement.txt
```

If packages are added or changed, update `requirement.txt` so the environment can be reproduced by other users.

## Running the project

Activate the virtual environment before running any project command:

```cmd
env\Scripts\activate.bat
```

Then run the project's Python entry point once it is available. For example:

```cmd
python app.py
```

Replace `app.py` with the appropriate application or script filename used by the project.

## Typical multimodal RAG workflow

1. Load source content such as PDFs, text files, images, or tables.
2. Extract or prepare content in a format suitable for retrieval.
3. Create embeddings and store them in a vector database.
4. Retrieve the most relevant content for a user question.
5. Pass the retrieved context to the language model to generate an answer grounded in the source material.

## Deactivating the environment

When you are done working, deactivate the virtual environment:

```cmd
deactivate
```
