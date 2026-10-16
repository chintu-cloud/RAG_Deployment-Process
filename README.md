

# Amazon Bedrock + OpenSearch RAG

Complete notes • Architecture • Workflow • Code • Interview preparation

Bedrock

AI models

OpenSearch

Retrieval

Documents

Knowledge

## 📚 Table of contents

01–05

RAG fundamentals, architecture, Bedrock, and chunking

06–10

Embeddings, OpenSearch storage, and query vectors

11–15

Similarity scores, prompts, generation, and end-to-end flow

16–20

Semantic search, Top-K, and model responsibilities

21–25

PDF example, stored records, vectors versus text

26–29

Architecture summary, takeaways, and interview answer

# 01. 📌 Project Overview

This project demonstrates a Retrieval-Augmented Generation (RAG) architecture using Amazon Bedrock and Amazon OpenSearch.

### Technologies used

Amazon Bedrock

Provides foundation models for embeddings and text generation.

Amazon Titan Embeddings

Converts document chunks and user questions into numerical vectors.

Amazon OpenSearch Service

Stores vectors and associated content, then retrieves similar records.

Python application

Connects the services and orchestrates the complete workflow.

### The main idea

### User Question

"What is Amazon S3?"

### Bedrock Embedding Model

Convert question into a vector

### OpenSearch Vector Search

Retrieve Top-K relevant chunks

### Bedrock LLM

Generate answer using retrieved context

### Final Answer

# 02. 🧠 What Is RAG?

RAG stands for Retrieval-Augmented Generation.

It combines two processes:

- Retrieval: Find relevant information from an external knowledge source.
- Generation: Use an LLM to generate an answer based on the question and retrieved information.

### Example

USER QUESTION

What is Amazon S3?

RETRIEVED CONTEXT

- Amazon S3 is an object storage service.
- S3 provides scalable storage.
- S3 stores objects inside buckets.
- S3 supports multiple storage classes.

GENERATED ANSWER

Amazon S3 is an AWS object storage service that provides scalable storage for data, stores objects in buckets, and supports multiple storage classes.

### Why do we need RAG?

- Answer questions about private or internal documents.
- Retrieve updated information without retraining the LLM.
- Provide relevant context to the model.
- Support document-based question answering.
- Help reduce unsupported answers.

### Important

RAG does not guarantee correct answers. Retrieval quality, prompt design, source quality, and LLM behavior still matter.

# 03. 🏗️ High-Level Architecture

RAG contains two main workflows:

- Ingestion phase: Prepare documents and store embeddings.
- Query phase: Retrieve relevant information and generate an answer.

## INGESTION PIPELINE

### Documents

PDF / TXT / DOCX

### Document Loader

Extract text

### Chunking

Split into passages

### Titan Embeddings

Generate document vectors

### OpenSearch Index

Text + vector + metadata

## QUERY PIPELINE

### User Question

Receive question

### Query Embedding

Generate query vector

### OpenSearch Search

Retrieve Top-K records

### Prompt Construction

Question + retrieved context

### Bedrock LLM

Generate final response

# 04. 🔀 Why Are There Two Bedrock Operations?

A common beginner question is:

"Why do we call Bedrock twice?"

Because embedding and generation are two different tasks.

## Operation 1: Embedding

Text → Vector

Used for semantic retrieval and similarity search.

## Operation 2: Generation

Prompt → Answer

Used for question answering and text generation.

Embedding example

text
```
"Amazon S3 is an object storage service."
                  |
                  v
           Titan Embeddings
                  |
                  v
       [0.12, -0.34, 0.78, ...]
```

Generation example

text
```
Question + Retrieved Context
              |
              v
          Bedrock LLM
              |
              v
          Final Answer
```

The embedding model and generation model can be different foundation models.

# 05. 📥 Data Ingestion and Chunking

Before users ask questions, documents must be processed.

text
```
Documents
    |
    v
Document Loader
    |
    v
Extract Text
    |
    v
Split Into Chunks
    |
    v
Generate Embeddings
    |
    v
Store in OpenSearch
```

### Example document

text
```
Amazon S3 is an object storage service.

Amazon S3 provides scalable storage for data.

S3 stores objects inside buckets.

Amazon S3 supports multiple storage classes.
```

### Split into chunks

CHUNK 1

Amazon S3 is an object storage service.

CHUNK 2

Amazon S3 provides scalable storage for data.

CHUNK 3

S3 stores objects inside buckets.

CHUNK 4

Amazon S3 supports multiple storage classes.

### Why chunking?

- Improves retrieval granularity.
- Avoids sending entire documents to the LLM.
- Makes it easier to identify relevant passages.
- Helps manage context length.

Chunk size and overlap should be chosen according to document structure and retrieval performance.

# 06. 🔢 Generate Embeddings

Each chunk is sent to the embedding model.

text
```
Chunk 1 ---> Titan Embeddings ---> Vector 1
Chunk 2 ---> Titan Embeddings ---> Vector 2
Chunk 3 ---> Titan Embeddings ---> Vector 3
Chunk 4 ---> Titan Embeddings ---> Vector 4
```

### Illustrative vectors

text
```
Chunk 1
[0.12, -0.34, 0.78, ...]

Chunk 2
[0.42, 0.18, -0.22, ...]

Chunk 3
[-0.15, 0.81, 0.34, ...]

Chunk 4
[0.52, -0.27, 0.63, ...]
```

These are example values, not actual Titan output.

### Important concept

An embedding is a numerical representation of text produced by an embedding model.

The vectors should be compatible with the vector field's configured dimensions in OpenSearch. For example, a 1,024-dimensional embedding requires a compatible vector field.

# 07. 🗄️ Store Vectors in OpenSearch

OpenSearch stores the original text, its vector, and metadata.

text
```
+-------------------------------------+
|           OpenSearch                |
+-------------------------------------+
| Text: Amazon S3 is an object         |
|       storage service.              |
|                                     |
| Vector: [0.12, -0.34, 0.78, ...]    |
|                                     |
| Source: s3-documentation.pdf         |
| Page: 10                            |
+-------------------------------------+
```

### What does each field do?

| Field | Purpose |
| --- | --- |
| `text` | Original content |
| `vector` | Used for vector similarity search |
| `source` | Identifies the document |
| `page` | Identifies the source page |

Remember: OpenSearch stores the vector to retrieve the record and the original text to provide context to the LLM.

# 08. 💬 User Asks a Question

Example:

text
```
"What is Amazon S3?"
```

The application receives the question and prepares to search the knowledge base.

text
```
+----------------------+
|         User         |
+----------+-----------+
           |
           v
+----------------------+
|  Python Application  |
+----------+-----------+
           |
           v
     Query Processing
```

In the basic RAG pipeline, retrieval happens before generation.

# 09. 🎯 Convert the Question Into a Vector

text
```
"What is Amazon S3?"
          |
          v
   Titan Embeddings
          |
          v
      Query Vector
```

Illustrative output:

text
```
[0.12, -0.34, 0.78, ...]
```

The query embedding must be compatible with the embeddings stored during ingestion.

# 10. 🔍 Send the Query Vector to OpenSearch

OpenSearch compares the query vector with the indexed document vectors.

### Query Vector

Vector 1

Vector 2

Vector 3

### Similarity Search

### Top-K Results

# 11. 📊 What Does Similarity Mean?

Suppose OpenSearch returns the following illustrative scores.

Chunk 7

0.92

Chunk 2

0.88

Chunk 15

0.83

Chunk 4

0.61

Chunk 20

0.42

Illustrative scores only. Score interpretation depends on the configured similarity metric and search method.

If the application selects Top-K = 3, it chooses the three highest-ranked results in this example.

# 12. 📄 OpenSearch Returns the Actual Text

The application receives records containing relevant text and metadata.

text
```
Chunk 7:
"Amazon S3 is an object storage service."

Chunk 2:
"S3 provides scalable object storage."

Chunk 15:
"S3 supports multiple storage classes."
```

The application uses the text from the matching records to construct the LLM prompt.

text
```
Query Vector
     |
     v
OpenSearch
     |
     v
Top-K Records
     |
     v
Original Text Chunks
     |
     v
LLM Context
```

# 13. 📝 Build the RAG Prompt

The prompt combines the retrieved context and the user's question.

## RAG Prompt

SYSTEM INSTRUCTION

Answer the question using the supplied context. If the answer cannot be determined from the context, say so.

CONTEXT

Amazon S3 is an object storage service.

S3 provides scalable object storage.

S3 supports multiple storage classes.

QUESTION

What is Amazon S3?

A good prompt helps the model stay focused on the retrieved information. Retrieved content should still be treated as untrusted input.

# 14. ✨ Bedrock LLM Generates the Answer

The second Bedrock operation generates the response.

### Question + Context

### Amazon Bedrock LLM

### Generated Answer

Amazon S3 is an AWS object storage service that stores objects in buckets and supports multiple storage classes.

The application then returns the answer to the user.

# 15. 🔄 Complete End-to-End Flow

## INGESTION

Documents

Document Loader

Text Chunking

Titan Embeddings

Vectors

OpenSearch: Text + Vector + Metadata

## QUERY

User Question

Query Embedding

OpenSearch Vector Search

Top-K Text Chunks

Question + Context

Bedrock LLM

Final Answer

# 16. 🧩 Simple Architecture Summary

The project can be understood as two separate pipelines.

Pipeline A — Build the knowledge base

text
```
Documents
    ↓
Chunking
    ↓
Titan Embeddings
    ↓
Vectors
    ↓
OpenSearch
```

Pipeline B — Answer questions

text
```
Question
    ↓
Titan Embeddings
    ↓
Query Vector
    ↓
OpenSearch Search
    ↓
Top-K Chunks
    ↓
Context + Question
    ↓
Bedrock LLM
    ↓
Answer
```

# 17. 🔎 Why Do We Need Embeddings?

Traditional keyword search focuses on matching words. Semantic search uses embeddings to retrieve content with related meaning.

## Keyword Search

Query: "object storage"

Looks for matching terms and their occurrences.

## Semantic Search

Query: "Where can I store files in AWS?"

May retrieve "Amazon S3 is an object storage service."

Semantic search is useful when a question and the relevant passage use different words to express related concepts.

# 18. 🗄️ Why Do We Need OpenSearch?

The embedding model creates the vector. OpenSearch stores and searches it.

### Bedrock Embeddings

Creates vectors from text.

### OpenSearch

Stores vectors, text, and metadata; performs retrieval.

OpenSearch is the retrieval layer in this architecture. Other vector databases can also be used to implement RAG.

# 19. ✨ Why Do We Need an LLM?

OpenSearch retrieves relevant content; the LLM generates the final natural-language answer.

| Component | Main job |
| --- | --- |
| Embedding model | Convert text into vectors |
| OpenSearch | Retrieve relevant records |
| LLM | Generate the answer |
| Python application | Connect the workflow |

The LLM uses the question and retrieved text to construct a readable response rather than returning only the search results.

# 20. 🧠 Two Different Bedrock Models or Operations

### Embedding

text
```
Text
  ↓
Embedding Model
  ↓
Numerical Vector
```

Used for semantic retrieval and similarity search.

### Generation

text
```
Prompt
  ↓
Foundation Model / LLM
  ↓
Generated Text
```

Used for question answering, summarization, and natural-language generation.

The embedding and generation models can be different, as long as the embedding configuration remains compatible between ingestion and querying.

# 21. 📄 Example: RAG With an AWS Documentation PDF

Suppose we have `aws-s3-documentation.pdf`.

STEP 1

### Load

Extract text from the PDF.

STEP 2

### Split

Create smaller text chunks.

STEP 3

### Embed

Generate a vector for each chunk.

STEP 4

### Store

Index text, vectors, and metadata in OpenSearch.

STEP 5

### Retrieve

Embed the question and search for the Top-K chunks.

STEP 6

### Generate

Pass context and question to the Bedrock LLM.

# 22. 💡 The Most Important Mental Model

# RAG = Retrieve + Generate

### RETRIEVE

Embed the question and find relevant chunks.

### GENERATE

Give the LLM the context and question.

# 23. 🧾 What Is Stored in OpenSearch?

Example record:

json
```
{
  "text": "Amazon S3 is an object storage service.",
  "vector": [0.12, -0.34, 0.78],
  "source": "s3-documentation.pdf",
  "page": 10
}
```

Important note

The vector is abbreviated for illustration. The actual embedding must contain the dimensions produced by the selected model.

The record contains:

- Text: Original chunk.
- Vector: Numerical embedding.
- Source: Document identifier.
- Page: Source location, when available.

# 24. 🎯 What Does Top-K Mean?

Suppose OpenSearch contains 1,000 chunks.

Instead of passing all 1,000 chunks to the LLM, the application retrieves a smaller set of relevant results.

### 1,000 Chunks

### Similarity Search

### Top-K = 3

Select three results in this example.

### Bedrock LLM

Receives the question and selected context.

The best Top-K value depends on retrieval quality, document structure, and the model's context limits.

# 25. 🔢 Vector Is Not Text

A vector may look like:

text
```
[0.12, -0.34, 0.78, ...]
```

This is a numerical representation, not a readable sentence.

The application retrieves the associated text:

text
```
Vector
  ↓
OpenSearch Search
  ↓
Matching Record
  ↓
Original Text Chunk
  ↓
LLM Context
```

Example:

> "Amazon S3 is an object storage service."

The original text, rather than the numerical vector alone, is used as context for the LLM.

# 26. 🏁 Final Architecture Diagram

## INGESTION

Document

Chunking

Titan Embeddings

Vectors

OpenSearch: Text + Vector + Metadata

## QUERY

User Question

Query Embedding

OpenSearch Vector Search

Top-K Text Chunks

Question + Context

Bedrock LLM

Final Answer

# 27. ⭐ Key Takeaways

### 01

### Documents are chunked

Document → Smaller chunks

### 02

### Chunks become vectors

Chunk → Embedding model → Vector

### 03

### Vectors are stored

Text + Vector + Metadata → OpenSearch

### 04

### Questions become vectors

Question → Embedding model → Query vector

### 05

### Similar chunks are retrieved

Query vector → Search → Top-K results

### 06

### The LLM generates the answer

Question + Context → Bedrock LLM → Answer

# 28. 🎤 One-Line Interview Explanation

"During ingestion, I split documents into chunks, generate embeddings using Amazon Bedrock with Titan Embeddings, and store the vectors alongside the original text and metadata in OpenSearch. During a user query, I embed the question, perform vector similarity search to retrieve the Top-K relevant chunks, and pass those chunks along with the question to a Bedrock LLM to generate the final context-aware answer."

 Copy interview answer

# 29. 🌈 Complete Formula

## 📥 Ingestion

Document

Chunk

Embedding

OpenSearch

## 📤 Query

Question

Query Vector

Top-K Chunks

LLM → Answer

# The Final Mental Model

Embedding = Convert text into numbers

OpenSearch = Find relevant information

LLM = Generate the answer

# RAG = Retrieve + Generate

Final takeaway: Bedrock provides the AI models, OpenSearch retrieves relevant knowledge, and your Python application connects the components to build a RAG-based AI assistant.
