# Local Document RAG System

A **local Retrieval-Augmented Generation (RAG) system that allows users to ask questions about their PDF or text documents. The system processes the document, splits it into smaller chunks, converts those chunks into vector embeddings, stores them in **ChromaDB**, retrieves the most relevant content for a question, and generates a grounded answer using a locally running **Ollama LLM**.

The system is designed to answer questions **only from the provided document** and avoids generating information that is not present in the retrieved context.

---

## Features

*  Supports **PDF and text (`.txt`) documents**
*  Splits documents into manageable text chunks
*  Generates embeddings using `nomic-embed-text`
*  Stores embeddings locally using **ChromaDB**
*  Performs similarity search to find relevant document sections
*  Generates answers using **Llama 3.1** through Ollama
*  Uses the retrieved document context to ground responses
*  Prevents unsupported answers by instructing the LLM not to speculate
*  Runs completely locally after the required models and packages are installed
*  Interactive question-answering loop

---

## Architecture

```text
                 ┌──────────────────┐
                 │   PDF / TXT File │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │ Document Loader  │
                 │ PyPDFLoader /    │
                 │ TextLoader       │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │ Text Chunking    │
                 │ Recursive        │
                 │ Character Splitter│
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │ Embedding Model  │
                 │ nomic-embed-text │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │    ChromaDB      │
                 │ Vector Database  │
                 └────────┬─────────┘
                          │
                    User Question
                          │
                          ▼
                 ┌──────────────────┐
                 │ Similarity Search│
                 │      Top 3       │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │    Llama 3.1     │
                 │   via Ollama     │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │    Final Answer  │
                 └──────────────────┘
```

---

## Technologies Used

| Technology                     | Purpose                                 |
| ------------------------------ | --------------------------------------- |
| Python                         | Main programming language               |
| LangChain                      | RAG pipeline and document processing    |
| PyPDFLoader                    | Loading PDF documents                   |
| TextLoader                     | Loading text documents                  |
| RecursiveCharacterTextSplitter | Splitting documents into chunks         |
| Ollama                         | Running local LLMs and embedding models |
| Llama 3.1                      | Generating answers                      |
| nomic-embed-text               | Creating document embeddings            |
| ChromaDB                       | Local vector database                   |

---

## Requirements

Make sure the following are installed:

* Python **3.9+**
* Ollama
* Llama 3.1 model
* `nomic-embed-text` embedding model

---

## Installation

### 1. Clone the repository

```bash
git clone <your-repository-url>
cd <your-project-directory>
```

### 2. Create a virtual environment

Windows:

```bash
python -m venv venv
venv\Scripts\activate
```

Linux/macOS:

```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Install Python dependencies

```bash
pip install langchain langchain-community langchain-text-splitters langchain-ollama langchain-chroma
```

Alternatively, create a `requirements.txt` file:

```text
langchain
langchain-community
langchain-text-splitters
langchain-ollama
langchain-chroma
```

Then install:

```bash
pip install -r requirements.txt
```

---

## Ollama Setup

Install Ollama on your system and download the required models.

Pull the Llama 3.1 model:

```bash
ollama pull llama3.1
```

Pull the embedding model:

```bash
ollama pull nomic-embed-text
```

Verify the installed models:

```bash
ollama list
```

The output should include:

```text
llama3.1
nomic-embed-text
```

Make sure the Ollama service is running before starting the Python application.

---

## Project Structure

A simple project structure can be:

```text
RAG-Document-QA/
│
├── main.py
├── requirements.txt
├── README.md
│
└── chroma_db/
    └── ...
```

`chroma_db/` is automatically created when the document is indexed.

---

## Running the Application

Run the Python program:

```bash
python main.py
```

The program will ask for the path of the document:

```text
Enter path to your document (e.g., sample.pdf):
```

Example:

```text
sample.pdf
```

The system then performs the following steps:

```text
[Step 1: Document Processing]
Loading document...

[Step 2: Chunking]
Splitting document...

[Step 3 & 4: Embeddings & Vector Database Indexing]
Creating embeddings...
Storing vectors in ChromaDB...
```

After indexing is complete:

```text
==================================================
 RAG SYSTEM READY: Type your question or 'exit' to quit.
==================================================
```

You can now enter questions about the document.

---

## Example

Suppose the uploaded document contains information about **Cloud Computing**.

You could ask:

```text
Ask a question: What are the main characteristics of cloud computing?
```

The system first performs a similarity search and displays the retrieved chunks:

```text
--- Performing Similarity Search ---
[Chunk 1] (Distance Score: 0.2451 | Page: 2)
Cloud computing provides...

[Chunk 2] (Distance Score: 0.3187 | Page: 3)
The major characteristics include...

[Chunk 3] (Distance Score: 0.3522 | Page: 5)
...
```

It then sends the retrieved context to Llama 3.1:

```text
--- Generating Grounded Response ---

Final Answer:
Cloud computing has several key characteristics, including...
```

---

## How the RAG Pipeline Works

### 1. Document Loading

The application determines the document type based on its file extension.

For PDF files:

```python
loader = PyPDFLoader(file_path)
```

For other supported text files:

```python
loader = TextLoader(file_path)
```

The document is then loaded into LangChain documents.

---

### 2. Chunking

Large documents are divided into smaller sections using:

```python
RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)
```

This means each chunk can contain approximately **1000 characters**, with **200 characters overlapping** between consecutive chunks.

The overlap helps preserve context between chunks.

---

### 3. Embedding Generation

Each chunk is converted into a numerical vector using:

```text
nomic-embed-text
```

These vectors represent the semantic meaning of the text.

---

### 4. Vector Database

The embeddings and their corresponding document chunks are stored in:

```text
ChromaDB
```

The database is persisted in:

```text
./chroma_db
```

This allows the vector data to remain available locally.

---

### 5. Similarity Search

When the user asks a question, the question is converted into an embedding and compared with stored document embeddings.

The application retrieves the **3 most relevant chunks**:

```python
vector_db.similarity_search_with_score(query, k=3)
```

The retrieved chunks are then provided to the LLM as context.

---

### 6. Response Generation

The retrieved content is inserted into the system prompt:

```text
Context:
{context}
```

Llama 3.1 then generates an answer based on that context.

The prompt specifically instructs the model:

```text
Answer the question using ONLY the provided context.
```

If the required information is not present, the model is instructed to respond:

```text
The requested information is not available in the document.
```
---

## Grounded Responses

This project follows a basic **grounded RAG approach**.

Instead of sending the entire document to the LLM, the application:

```text
Document
   ↓
Chunks
   ↓
Embeddings
   ↓
Vector Database
   ↓
Relevant Chunks
   ↓
LLM
   ↓
Answer
```

This reduces the amount of irrelevant information provided to the model and helps keep responses focused on the document.

---

## Supported Files

Currently supported:

```text
.pdf
.txt
```

For PDF files, `PyPDFLoader` is used.

For text files, `TextLoader` is used.

---

## Exiting the Application

To exit the question-answering loop, type:

```text
exit
```
---

## Important Notes

* The document must exist at the path provided to the application.
* Ollama must be installed and running.
* Both `llama3.1` and `nomic-embed-text` must be available locally.
* The quality of answers depends on document quality, chunking, embedding quality, and retrieval accuracy.
* The similarity score displayed by ChromaDB is a **distance**, so lower values generally indicate greater similarity for the configured distance metric.
* The current implementation creates/updates a ChromaDB collection during ingestion.

---

## Limitations

The current version has a few limitations:

1. Only PDF and text documents are directly supported.
2. The entire document is indexed whenever the program starts.
3. There is no graphical user interface.
4. There is no authentication or multi-user support.
5. Retrieval is limited to the top 3 chunks.
6. The system does not currently provide citations or source references in the generated answer.
7. Scanned/image-only PDFs may require OCR before useful text can be retrieved.

---

## RAG Concepts Demonstrated

This project demonstrates several important concepts in modern AI applications:

* **Document ingestion**
* **Text chunking**
* **Embeddings**
* **Vector databases**
* **Semantic similarity search**
* **Information retrieval**
* **Retrieval-Augmented Generation (RAG)**
* **Prompt engineering**
* **Local LLM inference**
* **Grounded question answering**

---

Built using Python, LangChain, Ollama, ChromaDB, and Llama 3.1.
