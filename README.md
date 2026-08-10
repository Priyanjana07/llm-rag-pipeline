# llm-rag-pipeline

Built an end-to-end Retrieval-Augmented Generation pipeline for document ingestion, semantic retrieval, and groq-powered QNA.

## Architecture

```text
Documents
    │
    ▼
Document Loaders
    │
    ▼
Text Splitting / Chunking
    │
    ▼
Embedding Model
    │
    ▼
ChromaDB Vector Store
    │
    │
    │        User Query
    │             │
    │             ▼
    │       Query Embedding
    │             │
    └─────────────┤
                  ▼
           Semantic Search
                  │
                  ▼
           Top-K Relevant Chunks
                  │
                  ▼
                Context
                  │
                  ▼
              LLM / Groq
                  │
                  ▼
              Final Answer
Features
1. PDF and Text Document Ingestion:
Documents are loaded using LangChain document loaders.
The loaders convert raw files into LangChain Document objects containing:
page_content — actual document text
metadata — information about the document such as the source and page number.

2. Text Chunking for Retrieval:
Large documents are divided into smaller chunks.
Chunking allows the retrieval system to search smaller, more relevant sections instead of passing an entire document to the LLM.

3. Sentence Transformer Embeddings:
Each chunk is converted into a numerical vector using a Sentence Transformer model.
The current embedding model is:
all-MiniLM-L6-v2
These vectors represent the semantic meaning of the text.

4. Persistent ChromaDB Vector Store:
The embeddings and their associated document information are stored in ChromaDB.
Each stored entry contains:
Unique ID
Document text
Metadata
Embedding

5. Semantic Similarity Search:
When a user asks a question, the question is also converted into an embedding.
The query embedding is compared against the stored document embeddings.
The system retrieves the most semantically similar chunks using cosine similarity.
The retrieved chunks are combined into a context string.
The context is then provided to the LLM together with the original query (Augmentation).
Similarity score threshold controls the minimum similarity score required for a chunk to be included.
This prevents low-relevance chunks from being passed to the LLM.

6. Top-K Document Retrieval:
The LLM uses the retrieved context to generate the final answer.
This reduces the need for the model to rely only on its pretrained knowledge and allows it to answer questions using information contained in the provided documents.
Top_K is a retrieval parameter that controls how many relevant chunks must be retrieved.

7. Context Construction from Retrieved Chunks:
After semantic search, the retriever returns the most relevant chunks combined into one context string.
That context is then placed into the prompt sent to the LLM.

8. Groq LLM-Powered Question Answering:
Once the context and question are ready, your application sends them to the Groq-hosted LLM through LangChain's ChatGroq.
The LLM reads the retrieved context and generates the answer.
This is the generation part of Retrieval-Augmented Generation.

Entire Pipeline:
RETRIEVAL:
Documents → Chunks → Embeddings → ChromaDB → Relevant Chunks
                         ↓
GENERATION:
Relevant Chunks → Context → Prompt → Groq LLM → Answer

Tech Stack:
Python
LangChain
Sentence Transformers
ChromaDB
Groq LLM API
NumPy
scikit-learn
Jupyter Notebook
uv (python dependency)
