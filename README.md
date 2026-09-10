# Local RAG-Based Document Q&A System

## 1. Project Overview
This project is a local Retrieval-Augmented Generation (RAG) application that allows a user to provide a document and ask contextual questions about its contentIt processes the document, stores it in a vector database, and uses local AI models to answer questions based strictly on the provided text, without relying solely on the model's general knowledge.

## 2. Architecture / Workflow
The application follows this standard RAG pipeline:
Document → Text Extraction → Chunking → Embeddings → Vector Database → Similarity Search → Retrieved Context → LLM → Answer[cite: 1].

## 3. Technologies Used
* **Framework:** LangChain (LCEL architecture)
* **Embedding Model:** `nomic-embed-text` (via Ollama)
* **LLM:** `llama3.1` (via Ollama)
* **Vector Database:** ChromaDB.
* **Document Processing:** PyPDFLoader / TextLoader

## 4. How to Run the Application
1. Place a text-selectable PDF or TXT file into the project directory.
2. Run the application from the terminal: `python rag_app.py`
3. Enter the filename when prompted (e.g., `sample.pdf`).
4. Type your questions in the interactive terminal loop. Type `exit` to quit.

## 5. Conceptual Explanations
* **What embeddings are:** Embeddings convert each text chunk into a high-dimensional vector representation using an embedding model. This mathematical format allows computers to understand the semantic meaning of the text.
* **Why a vector database is required:** It is required to efficiently store the generated vector embeddings alongside their corresponding raw text chunks so they can be rapidly searched later.
* **How similarity search works:** When a user asks a question, the query is converted into an embedding[cite: 1]. The system then searches the vector database by calculating the mathematical distance between the query vector and stored vectors to retrieve the most relevant chunks.
* **Chunk size and overlap:** A chunk size of `1000` characters with an overlap of `200` characters was selected. This size provides enough context for the LLM to understand a full thought, while the overlap ensures that concepts split across chunk boundaries are not lost during retrieval.
* **RAG vs. standard LLM prompting:** RAG fundamentally differs from standard prompting because it forces the LLM to answer using a specific, retrieved context.If the answer cannot be found in that context, the system is instructed to explicitly state that the information is not available, rather than hallucinating an answer based on its general training data.
