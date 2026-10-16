

# ⚡ Amazon Bedrock + OpenSearch RAG

<p align="center">
<img src="https://img.shields.io/badge/AI-Generative%20AI-00F5FF?style=for-the-badge&logo=openai&logoColor=black" alt="Generative AI"/>
<img src="https://img.shields.io/badge/AWS-Amazon%20Bedrock-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Amazon Bedrock"/>
<img src="https://img.shields.io/badge/Embeddings-Amazon%20Titan-BD00FF?style=for-the-badge" alt="Titan Embeddings"/>
<img src="https://img.shields.io/badge/Vector%20Search-OpenSearch-00D4FF?style=for-the-badge" alt="OpenSearch"/>
<img src="https://img.shields.io/badge/Language-Python-00FF9C?style=for-the-badge&logo=python&logoColor=black" alt="Python"/>
</p>

<p align="center">
<b>⚡ RETRIEVE • AUGMENT • GENERATE ⚡</b>
<br/>
<i>A Complete Beginner-Friendly Guide to Building RAG Applications on AWS</i>
</p>

## 🌌 Project Overview

This project explains a simple **Retrieval-Augmented Generation (RAG)** architecture using:

- 🟠 **Amazon Bedrock** — Provides access to embedding models and large language models.
- 🟣 **Amazon Titan Embeddings** — Converts text into numerical vectors.
- 🔵 **Amazon OpenSearch** — Stores vectors and performs similarity searches.
- 🐍 **Python** — Connects the different components.
- 🧠 **RAG** — Retrieves relevant information before generating an answer.

### 🎯 Main Objective

Build an application that can answer questions using information from our own documents.

Instead of asking an LLM to answer a question only from its trained knowledge, the application first searches a knowledge base and retrieves relevant text.

The retrieved information is then passed to the LLM as context.

### ⚡ Basic RAG Architecture

```
                    ┌─────────────────────┐
                    │       User          │
                    │ "What is Amazon S3?"│
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │  Your Application   │
                    │   Python / API      │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   Amazon Bedrock    │
                    │ Titan Embeddings    │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │    Query Vector     │
                    │ [0.12,-0.34,0.78...]│
                    │    1024 dimensions  │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │    OpenSearch       │
                    │  Vector Search      │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Top K Similar Chunks│
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   Prompt + Context  │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   Amazon Bedrock    │
                    │        LLM          │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │    Final Answer     │
                    └─────────────────────┘

            
```

---


## 🧠 1. What Is RAG?

RAG stands for:

> **Retrieval-Augmented Generation**

It combines two important operations:

- 🔍 **Retrieval:** Find relevant information from a knowledge base.
- ✨ **Generation:** Use an LLM to generate an answer using the retrieved information.

### 💬 Example

Suppose a user asks:

```
What is Amazon S3?
```

A traditional LLM might answer using information learned during training and the instructions in the prompt.

A RAG application first searches its own knowledge base.

The knowledge base contains:

```
Chunk 1:
Amazon S3 is an object storage service.

Chunk 2:
Amazon S3 provides scalable storage for data.

Chunk 3:
S3 stores objects inside buckets.

Chunk 4:
Amazon S3 supports multiple storage classes.
```

The application retrieves relevant chunks and sends them to the LLM.

The LLM generates an answer such as:

```
Amazon S3 is an AWS object storage service that
provides scalable storage for data. It stores
objects inside buckets and supports multiple
storage classes.
```

### 🎯 Why Is RAG Useful?

- It can use private company documents.
- It can retrieve information from an organization's knowledge base.
- It can use updated documents after they have been indexed.
- It can provide answers grounded in retrieved source material.
- It can include source references to help users verify answers.

**Important:** RAG does not guarantee that every answer is correct. Retrieval quality, document quality, prompt design, and LLM behavior all affect the result.

---

## 🏗️ 2. High-Level Architecture

The architecture contains an ingestion pipeline and a query pipeline.

```
flowchart TD
    U["👤 User Question"] --> APP["🐍 Python Application"]
    APP --> EMB["🟣 Titan Embeddings"]
    EMB --> QV["🔢 Query Vector"]
    QV --> OS[("🔵 Amazon OpenSearch")]
    OS --> TOP["📚 Top-K Relevant Chunks"]
    TOP --> PROMPT["📝 Question + Context"]
    PROMPT --> LLM["🟠 Bedrock LLM"]
    LLM --> ANS["✅ Final Answer"]

    style U fill:#111827,stroke:#00F5FF,color:#FFFFFF
    style APP fill:#111827,stroke:#BD00FF,color:#FFFFFF
    style EMB fill:#111827,stroke:#BD00FF,color:#FFFFFF
    style QV fill:#111827,stroke:#00D4FF,color:#FFFFFF
    style OS fill:#111827,stroke:#00D4FF,color:#FFFFFF
    style TOP fill:#111827,stroke:#00FF9C,color:#FFFFFF
    style PROMPT fill:#111827,stroke:#FF9900,color:#FFFFFF
    style LLM fill:#111827,stroke:#FF9900,color:#FFFFFF
    style ANS fill:#111827,stroke:#00FF9C,color:#FFFFFF
```

### 🧩 Component Responsibilities

| Component | Responsibility |
| --- | --- |
| Python application | Coordinates the workflow |
| Titan Embeddings | Converts text into vectors |
| Query vector | Represents the user's question numerically |
| OpenSearch | Stores vectors and retrieves matching records |
| Retrieved chunks | Provide relevant context |
| Bedrock LLM | Generates the answer |
| User interface | Displays the answer |

### 🔥 Main Idea

```
Embedding Model = Convert text into numbers

OpenSearch = Find relevant information

LLM = Generate a human-readable answer

RAG = Retrieve information first, then generate
```

---

## 🔄 3. Why Do We Call Bedrock Twice?

A common beginner question is:

> Why are we calling Amazon Bedrock two times?

Because there are two different jobs.

### 🟣 Operation 1 — Embedding

The embedding model converts text into a numerical vector.

```
Text
  |
  v
Titan Embedding Model
  |
  v
Numerical Vector
```

Example:

```
Input:
"Amazon S3 is an object storage service."

Output:
[0.12, -0.34, 0.78, ...]
```

The vector is used for semantic retrieval.

### 🟠 Operation 2 — Generation

The generation model converts the question and retrieved context into an answer.

```
Question + Context
        |
        v
    Bedrock LLM
        |
        v
  Natural-Language Answer
```

### Important Difference

| Operation | Input | Output |
| --- | --- | --- |
| Embedding | Text | Numerical vector |
| Generation | Prompt and context | Generated text |

These are separate model operations. They may use different Bedrock model IDs.

---

## 📥 4. Two Main Phases of RAG

RAG has two major phases.

### Phase 1 — Data Ingestion

This phase prepares documents before users ask questions.

```
flowchart TD
    A["📄 Documents"] --> B["Document Loader"]
    B --> C["Text Extraction"]
    C --> D["Chunking"]
    D --> E["Titan Embeddings"]
    E --> F["Document Vectors"]
    F --> G[("OpenSearch Index")]
    G --- H["Text + Vector + Metadata"]

    style A fill:#111827,stroke:#00D4FF,color:#FFFFFF
    style D fill:#111827,stroke:#BD00FF,color:#FFFFFF
    style E fill:#111827,stroke:#BD00FF,color:#FFFFFF
    style G fill:#111827,stroke:#00F5FF,color:#FFFFFF
```

Steps:

1. Load documents.
2. Extract text.
3. Split text into chunks.
4. Generate an embedding for every chunk.
5. Store the original text, vector, and metadata in OpenSearch.

### Phase 2 — Query Processing

This phase happens when a user asks a question.

```
flowchart TD
    A["👤 User Question"] --> B["Titan Query Embedding"]
    B --> C["Query Vector"]
    C --> D["OpenSearch Vector Search"]
    D --> E["Top-K Results"]
    E --> F["Original Text Chunks"]
    F --> G["Question + Context"]
    G --> H["Bedrock LLM"]
    H --> I["Final Answer"]

    style A fill:#111827,stroke:#00F5FF,color:#FFFFFF
    style B fill:#111827,stroke:#BD00FF,color:#FFFFFF
    style D fill:#111827,stroke:#00D4FF,color:#FFFFFF
    style E fill:#111827,stroke:#00FF9C,color:#FFFFFF
    style H fill:#111827,stroke:#FF9900,color:#FFFFFF
    style I fill:#111827,stroke:#00FF9C,color:#FFFFFF
```

**Remember:** Ingestion creates the searchable knowledge base. Query processing uses that knowledge base to answer questions.

---

## 📄 5. Example Documents

Suppose we have a document containing:

```
Amazon S3 is an object storage service.

Amazon S3 provides scalable storage for data.

S3 stores objects inside buckets.

Amazon S3 supports multiple storage classes.
```

We split it into smaller chunks.

### Chunk 1

```
Amazon S3 is an object storage service.
```

### Chunk 2

```
Amazon S3 provides scalable storage for data.
```

### Chunk 3

```
S3 stores objects inside buckets.
```

### Chunk 4

```
Amazon S3 supports multiple storage classes.
```

Each chunk is treated as an individual piece of information during indexing.

### Why Do We Split Documents?

- A large document may contain information about many different topics.
- Smaller chunks can make retrieval more precise.
- The application can retrieve only the sections relevant to a question.
- The LLM receives a manageable amount of context.

In real applications, chunking may use token limits, sentence boundaries, paragraph boundaries, and overlapping text.

---

## 🔢 6. Generate Embeddings

Each chunk is sent to the embedding model.

```
flowchart LR
    A["Chunk 1"] --> E1["Titan Embeddings"]
    B["Chunk 2"] --> E2["Titan Embeddings"]
    C["Chunk 3"] --> E3["Titan Embeddings"]
    E1 --> V1["Vector 1"]
    E2 --> V2["Vector 2"]
    E3 --> V3["Vector 3"]

    style A fill:#111827,stroke:#00D4FF,color:#FFFFFF
    style B fill:#111827,stroke:#00D4FF,color:#FFFFFF
    style C fill:#111827,stroke:#00D4FF,color:#FFFFFF
    style E1 fill:#111827,stroke:#BD00FF,color:#FFFFFF
    style E2 fill:#111827,stroke:#BD00FF,color:#FFFFFF
    style E3 fill:#111827,stroke:#BD00FF,color:#FFFFFF
    style V1 fill:#111827,stroke:#00FF9C,color:#FFFFFF
    style V2 fill:#111827,stroke:#00FF9C,color:#FFFFFF
    style V3 fill:#111827,stroke:#00FF9C,color:#FFFFFF
```

### Example Vectors

```
Chunk 1:
[0.12, -0.34, 0.78, ...]

Chunk 2:
[0.42, 0.18, -0.22, ...]

Chunk 3:
[-0.15, 0.81, 0.34, ...]

Chunk 4:
[0.52, -0.27, 0.63, ...]
```

These vectors are illustrative, not actual outputs from Amazon Titan.

### 🧠 What Is an Embedding?

An embedding is a numerical representation of text that captures useful semantic relationships.

Texts with similar meanings can have nearby vectors according to the chosen similarity measure.

For example:

```
Query:
"Where can I store files in AWS?"

Document:
"Amazon S3 is an object storage service."
```

The two texts do not contain exactly the same words, but their meanings are related.

### Vector Dimensions

An embedding model produces a vector with a specific number of dimensions.

For example, certain Amazon Titan Text Embeddings models support 1,024-dimensional embeddings.

The actual vector dimensions depend on the model and its configuration.

**Important:** The embedding dimensions must match the OpenSearch vector field mapping.

---

## 🗄️ 7. Store Vectors in OpenSearch

OpenSearch stores the original text and its vector representation.

A simplified record looks like this:

```
{
  "text": "Amazon S3 is an object storage service.",
  "vector": [0.12, -0.34, 0.78],
  "source": "s3-documentation.pdf",
  "page": 10,
  "chunk_id": "chunk-001"
}
```

The vector above is abbreviated for explanation. A real vector must contain the correct number of dimensions.

### What Does OpenSearch Store?

| Field | Purpose |
| --- | --- |
| `text` | Original chunk text |
| `vector` | Embedding of the chunk |
| `source` | Original document name |
| `page` | Page or source location |
| `chunk_id` | Unique identifier |

### Why Store Both Text and Vector?

The vector is used to find similar records.

The original text is needed to provide readable information to the LLM.

```
Query Vector
     |
     v
OpenSearch
     |
     v
Matching Record
     |
     v
Original Text Chunk
     |
     v
Bedrock LLM
```

OpenSearch can return the matching document fields along with the search score.

---

## 🙋 8. User Asks a Question

After indexing, a user asks:

```
What is Amazon S3?
```

The application receives the question.

```
User
 |
 | "What is Amazon S3?"
 v
Python Application
```

The application does not normally send the question directly to vector search as plain text when using vector-only retrieval.

Instead, it first generates a compatible query embedding.

---

## 🔢 9. Convert the Question into a Vector

The question is sent to the embedding model.

```
"What is Amazon S3?"
        |
        v
Titan Embedding Model
        |
        v
Query Vector
```

Conceptually:

```
[0.12, -0.34, 0.78, ...]
```

This is only an illustration. The actual query vector depends on the input and the selected model.

### Document Embedding vs Query Embedding

| Type | Input | Purpose |
| --- | --- | --- |
| Document embedding | A stored text chunk | Represents the chunk |
| Query embedding | A user's question | Represents the search request |

Both should use compatible embedding models and configurations.

---

## 🔍 10. Perform Vector Search

The application sends the query vector to OpenSearch.

OpenSearch compares the query vector with vectors stored in the index.

```
                 QUERY VECTOR
                      |
          +-----------+-----------+
          |           |           |
          v           v           v
       Vector 1    Vector 2    Vector 3
          |           |           |
          +-----------+-----------+
                      |
                      v
              SIMILARITY SEARCH
                      |
                      v
                TOP-K RESULTS
```

The search process ranks documents using the configured vector-search method.

### Example

Suppose the index contains 1,000 chunks.

The query is:

```
"What is Amazon S3?"
```

The application requests the most relevant results.

```
1000 stored chunks
        |
        v
Vector similarity search
        |
        v
Top 3 results
```

The selected records are then used as context for generation.

---

## 📊 11. Understand Similarity Scores

OpenSearch can return search scores with the matching records.

Example:

| Chunk | Illustrative Score |
| --- | --- |
| Chunk 7 | 0.92 |
| Chunk 2 | 0.88 |
| Chunk 15 | 0.83 |
| Chunk 4 | 0.61 |
| Chunk 20 | 0.42 |

These numbers are illustrative and do not represent actual search results.

### What Does a Score Mean?

The score helps rank results according to the configured search method.

However, the meaning of the score depends on the similarity metric and OpenSearch configuration.

A higher score does not automatically mean that a document has the same percentage probability of being correct.

### What Happens Next?

If the application requests Top-K = 3, it selects the three highest-ranked results.

```
Chunk 7  -> Selected
Chunk 2  -> Selected
Chunk 15 -> Selected

Chunk 4  -> Not selected
Chunk 20 -> Not selected
```

The selected chunks are passed to the next stage.

---

## 📚 12. Retrieve the Original Text

This is one of the most important steps in RAG.

OpenSearch does not need to return only vectors or scores. It can return the matching records, including the original text stored in each record.

For example:

```
Chunk 7:
"Amazon S3 is an object storage service."

Chunk 2:
"Amazon S3 provides scalable storage for data."

Chunk 15:
"Amazon S3 supports multiple storage classes."
```

The application collects the returned text.

```
Top-K Search Results
        |
        v
Original Text Chunks
        |
        v
Context for the LLM
```

**Important:** The application normally sends readable text and relevant metadata to the LLM, not just the numerical vectors.

---

## 📝 13. Build the RAG Prompt

The application combines the user question with the retrieved text.

This is called **prompt augmentation**.

### Example Prompt

```
SYSTEM INSTRUCTION:

Answer the question using the provided context.
Do not invent facts that are not supported by
the available information.

If the answer cannot be found in the context,
explain that the available information is insufficient.

CONTEXT:

Amazon S3 is an object storage service.

Amazon S3 provides scalable storage for data.

S3 supports multiple storage classes.

QUESTION:

What is Amazon S3?
```

### What Does the Prompt Do?

- Provides the LLM with relevant source information.
- Tells the LLM what question to answer.
- Encourages the LLM to stay grounded in the context.
- Helps reduce unsupported answers.

A well-designed production prompt should also account for untrusted retrieved text, source attribution, and the possibility that the context does not answer the question.

---

## 🤖 14. Generate the Final Answer

Now the second model operation happens.

The application sends the question and context to a text-generation model through Amazon Bedrock.

```
Question + Retrieved Context
             |
             v
      Amazon Bedrock LLM
             |
             v
       Generated Answer
```

### Example Answer

```
Amazon S3 is an AWS object storage service that
provides scalable storage for data. It stores
objects inside buckets and supports multiple
storage classes for different access requirements.
```

The application then returns the generated answer to the user.

### Important Distinction

OpenSearch retrieves information.

The LLM generates the response.

The application connects the two components and controls how retrieved information is used.

---

## 🌐 15. Complete End-to-End Architecture

The complete process combines ingestion and query processing.

```
flowchart TB
    subgraph ING["📥 INGESTION PHASE"]
        DOC["Documents / PDFs"] --> LOAD["Document Loader"]
        LOAD --> SPLIT["Text Chunking"]
        SPLIT --> EMB["Titan Embeddings"]
        EMB --> VEC["Document Vectors"]
        VEC --> INDEX[("OpenSearch Index")]
        SPLIT -. "Store original text" .-> INDEX
    end

    subgraph QUERY["⚡ QUERY PHASE"]
        USER["User Question"] --> QEMB["Titan Query Embedding"]
        QEMB --> QV["Query Vector"]
        QV --> SEARCH["OpenSearch Vector Search"]
        SEARCH --> TOP["Top-K Relevant Chunks"]
        TOP --> PROMPT["Question + Context"]
        USER --> PROMPT
        PROMPT --> LLM["Bedrock LLM"]
        LLM --> ANSWER["Final Answer"]
    end

    INDEX --> SEARCH

    style DOC fill:#111827,stroke:#00D4FF,color:#FFFFFF
    style SPLIT fill:#111827,stroke:#BD00FF,color:#FFFFFF
    style EMB fill:#111827,stroke:#BD00FF,color:#FFFFFF
    style INDEX fill:#111827,stroke:#00D4FF,color:#FFFFFF
    style USER fill:#111827,stroke:#00F5FF,color:#FFFFFF
    style SEARCH fill:#111827,stroke:#00D4FF,color:#FFFFFF
    style TOP fill:#111827,stroke:#00FF9C,color:#FFFFFF
    style LLM fill:#111827,stroke:#FF9900,color:#FFFFFF
    style ANSWER fill:#111827,stroke:#00FF9C,color:#FFFFFF
```

### Ingestion Formula

```
Document
   |
   v
Load and Extract Text
   |
   v
Split into Chunks
   |
   v
Generate Embeddings
   |
   v
Store Text + Vector + Metadata
   |
   v
OpenSearch
```

### Query Formula

```
User Question
   |
   v
Generate Query Embedding
   |
   v
Query Vector
   |
   v
OpenSearch Vector Search
   |
   v
Retrieve Top-K Text Chunks
   |
   v
Build Prompt
   |
   v
Bedrock LLM
   |
   v
Final Answer
```

---

## 🧩 16. Why Do We Need Embeddings?

Traditional keyword search looks for matching terms.

For example:

```
Query:
"object storage"
```

Keyword search looks for terms such as:

```
object
storage
```

Vector search uses embeddings to find semantically related content.

Example:

```
Query:
"Where can I store files in AWS?"
```

A related document might say:

```
"Amazon S3 is an object storage service."
```

The two sentences use different words, but they are semantically related.

### Key Benefits

- Semantic retrieval.
- Better handling of paraphrased questions.
- Retrieval based on meaning rather than only exact words.

Vector search is not perfect. Exact identifiers, product codes, and rare technical terms may benefit from combining vector search with keyword search.

---

## 🗄️ 17. Why Do We Need OpenSearch?

The embedding model generates vectors, but the application also needs a searchable store for those vectors.

OpenSearch provides:

- Vector storage.
- Vector similarity search.
- Original text storage.
- Metadata storage.
- Filtering and other search capabilities.

### Responsibility Split

```
Amazon Bedrock Embeddings
            |
            v
       Creates vectors

Amazon OpenSearch
            |
            v
    Stores and searches vectors
```

The application retrieves matching text and sends it to the LLM.

---

## 🧠 18. Why Do We Need an LLM?

OpenSearch retrieves relevant documents, but retrieval alone does not usually provide the complete natural-language response expected by the user.

For example, OpenSearch might return:

```
Chunk 1:
"S3 is an object storage service."

Chunk 2:
"S3 stores objects inside buckets."

Chunk 3:
"S3 supports multiple storage classes."
```

The LLM can combine the retrieved information into a readable answer:

```
Amazon S3 is an object storage service that stores
objects in buckets and supports multiple storage
classes.
```

### Responsibility Split

| Component | Job |
| --- | --- |
| Embedding model | Represents text numerically |
| OpenSearch | Retrieves relevant records |
| LLM | Generates the answer |
| Application | Coordinates the workflow |

---

## ⚡ 19. Embedding vs Generation

Embedding and generation are different operations.

### Embedding

```
Text
  |
  v
Embedding Model
  |
  v
Vector
```

Used for:

- Semantic search.
- Similarity search.
- Retrieval.

### Generation

```
Prompt
  |
  v
LLM
  |
  v
Text Answer
```

Used for:

- Question answering.
- Summarization.
- Text generation.
- Conversational responses.

### Simple Mental Model

```
Embedding = Convert text into numbers

OpenSearch = Find relevant information

LLM = Turn the information into an answer

RAG = Retrieve first, then generate
```

---

## 📄 20. Example: PDF Documentation

Suppose we have an AWS documentation file:

```
aws-s3-documentation.pdf
```

### Step 1 — Load the PDF

```
PDF
 |
 v
Document Loader
 |
 v
Extracted Text
```

### Step 2 — Split the Text

```
Extracted Text
      |
      v
   Chunking
      |
      +-- Chunk 1
      +-- Chunk 2
      +-- Chunk 3
      +-- ...
      +-- Chunk 100
```

### Step 3 — Generate Embeddings

```
Chunk 1   -> Embedding -> Vector 1
Chunk 2   -> Embedding -> Vector 2
Chunk 3   -> Embedding -> Vector 3
...
Chunk 100 -> Embedding -> Vector 100
```

### Step 4 — Store the Records

```
OpenSearch

Chunk 1 + Vector 1 + Metadata
Chunk 2 + Vector 2 + Metadata
Chunk 3 + Vector 3 + Metadata
...
Chunk 100 + Vector 100 + Metadata
```

### Step 5 — User Asks a Question

```
"What is Amazon S3?"
```

### Step 6 — Generate the Query Vector

```
Question
   |
   v
Titan Embeddings
   |
   v
Query Vector
```

### Step 7 — Search OpenSearch

```
Query Vector
     |
     v
Vector Similarity Search
     |
     v
Top-K Results
```

### Step 8 — Build the Prompt

```
Context:
Chunk 7
Chunk 2
Chunk 15

Question:
What is Amazon S3?
```

### Step 9 — Generate the Answer

```
Prompt
  |
  v
Bedrock LLM
  |
  v
Final Answer
```

---

## 📦 21. What Is Stored in OpenSearch?

A simplified OpenSearch record may look like this:

```
{
  "text": "Amazon S3 is an object storage service.",
  "vector": [0.12, -0.34, 0.78],
  "source": "s3-documentation.pdf",
  "page": 10,
  "chunk_id": "s3-page-10-chunk-1"
}
```

The vector is abbreviated for readability. The actual vector must match the dimensions configured for the selected embedding model.

### Three Important Parts

```
TEXT
 +
VECTOR
 +
METADATA
```

**Text** is the readable content.

**Vector** is the numerical representation used for similarity search.

**Metadata** provides information such as document name, page number, chunk identity, and access-control attributes.

---

## 🎯 22. What Is Top-K?

Suppose the knowledge base contains 1,000 chunks.

The user asks:

```
"What is Amazon S3?"
```

We usually do not want to send all 1,000 chunks to the LLM.

Instead, we retrieve a smaller set of relevant chunks.

```
1,000 CHUNKS
      |
      v
VECTOR SEARCH
      |
      v
TOP-K = 3
      |
      v
3 RELEVANT CHUNKS
      |
      v
BEDROCK LLM
```

### Example

```
Chunk 7  -> 0.92
Chunk 2  -> 0.88
Chunk 15 -> 0.83
```

If K equals 3, the application selects three top-ranked records.

### Why Is Top-K Important?

- Controls the number of retrieved chunks.
- Helps manage prompt size and token consumption.
- Can influence answer quality.
- Balances retrieval coverage against irrelevant context.

The best K depends on your documents and evaluation results.

---

## 🔢 23. Vector vs Text

This is a critical concept for beginners.

A vector may look like this:

```
[0.12, -0.34, 0.78, ...]
```

It is a numerical representation, not a readable sentence.

The LLM normally needs the retrieved text to understand the relevant evidence.

### Correct Flow

```
Query Vector
     |
     v
OpenSearch Similarity Search
     |
     v
Matching Record
     |
     v
Original Text Chunk
     |
     v
Question + Context
     |
     v
Bedrock LLM
```

### Remember

**Vectors help find relevant information. Original text provides the information used for generation.**

---

## 📁 24. Suggested Project Structure

A Python implementation could use the following structure:

```
bedrock-opensearch-rag/
|
+-- README.md
+-- requirements.txt
+-- .env.example
|
+-- data/
|   +-- sample.txt
|
+-- src/
|   +-- config.py
|   +-- ingestion.py
|   +-- chunking.py
|   +-- embeddings.py
|   +-- opensearch_client.py
|   +-- retrieval.py
|   +-- generation.py
|   +-- rag_pipeline.py
|
+-- tests/
|   +-- test_chunking.py
|   +-- test_retrieval.py
|
+-- docs/
    +-- architecture.md
```

### File Responsibilities

| File | Responsibility |
| --- | --- |
| `config.py` | Loads application settings |
| `ingestion.py` | Loads documents and indexes chunks |
| `chunking.py` | Splits documents into chunks |
| `embeddings.py` | Calls the embedding model |
| `opensearch_client.py` | Configures the OpenSearch client |
| `retrieval.py` | Searches for relevant chunks |
| `generation.py` | Invokes the LLM |
| `rag_pipeline.py` | Connects retrieval and generation |

This is a suggested structure for organizing an implementation.

---

## 💡 25. Important RAG Concepts

### 1\. Document Ingestion

Preparing source documents so they can be indexed.

### 2\. Chunking

Splitting long documents into smaller pieces.

### 3\. Embeddings

Converting text into numerical representations.

### 4\. Vector Database

Storing and retrieving vector representations. OpenSearch supports vector search alongside other search capabilities.

### 5\. Semantic Search

Retrieving content according to semantic similarity.

### 6\. Metadata

Additional information about a document, such as its source, page, category, or permissions.

### 7\. Top-K Retrieval

Selecting the highest-ranked search results.

### 8\. Prompt Augmentation

Combining the user's question with retrieved context.

### 9\. Grounded Generation

Encouraging the LLM to answer using the supplied evidence.

### 10\. Hybrid Search

Combining keyword search and vector search to improve retrieval for some use cases.

### 11\. Reranking

Reordering retrieved candidates using another relevance-scoring step.

### 12\. Evaluation

Measuring whether retrieval finds relevant information and whether generated answers are accurate and supported by sources.

---

## 🎤 26. Interview Questions

### Q1. What is RAG?

RAG stands for Retrieval-Augmented Generation. It retrieves relevant external information and provides it to an LLM as context for generating an answer.

### Q2. Why use Amazon Bedrock?

Amazon Bedrock provides managed access to supported foundation models and embedding models through APIs.

### Q3. Why use Titan Embeddings?

Titan Embeddings converts text into vectors that can be stored and searched using vector similarity.

### Q4. Why use OpenSearch?

OpenSearch stores vectors and associated text and metadata, then retrieves relevant records using vector search and other search techniques.

### Q5. Why do we call Bedrock twice?

One operation generates embeddings for retrieval. Another invokes a text-generation model to produce the final answer.

### Q6. What is Top-K?

Top-K is the number of highest-ranked search results selected to provide context to the LLM.

### Q7. Why do we store original text?

The vector helps locate relevant records, while the original text supplies readable context for generation.

### Q8. What is the difference between keyword search and vector search?

Keyword search focuses on matching terms. Vector search retrieves records based on relationships represented by embeddings. Hybrid search can combine both methods.

### Q9. Does RAG eliminate hallucinations?

No. RAG can improve grounding, but incorrect retrieval, incomplete context, and generation errors can still cause hallucinations.

### Q10. How can RAG quality be improved?

Improve chunking, evaluate embedding models, tune retrieval parameters, use hybrid search or reranking, enforce access controls, and measure retrieval and answer quality.

### Q11. What happens if OpenSearch returns irrelevant chunks?

The LLM may generate a weak or incorrect answer. The application can improve retrieval, adjust the number of results, use reranking, and return an uncertainty response when evidence is insufficient.

### Q12. What happens if the answer is not present in the documents?

The application should instruct the LLM to say that the available context does not contain sufficient information rather than inventing an answer.

### ⭐ Interview-Ready Explanation

> During ingestion, I split documents into chunks, generate embeddings using Amazon Bedrock Titan Embeddings, and store the vectors along with the original chunks and metadata in Amazon OpenSearch. When a user asks a question, I generate an embedding for the query, perform vector similarity search to retrieve the top-K relevant chunks, and pass those chunks with the question to a Bedrock LLM to generate the final grounded response.

---

## ✅ 27. Key Takeaways

### 1\. Documents Are Chunked

```
Document -> Chunks
```

### 2\. Chunks Become Vectors

```
Chunk -> Titan Embeddings -> Vector
```

### 3\. Vectors Are Stored in OpenSearch

```
Text + Vector + Metadata -> OpenSearch
```

### 4\. The User Question Also Becomes a Vector

```
Question -> Titan Embeddings -> Query Vector
```

### 5\. OpenSearch Performs Similarity Search

```
Query Vector
     |
     v
Stored Vectors
     |
     v
Ranking
     |
     v
Top-K Results
```

### 6\. Original Text Chunks Are Retrieved

```
Search Results
      |
      v
Original Text
```

### 7\. The LLM Receives the Question and Context

```
Question + Context
        |
        v
    Bedrock LLM
        |
        v
    Final Answer
```

---

## 🏆 28. One-Line Interview Explanation

If someone asks:

**"Explain your RAG architecture."**

You can answer:

> "During ingestion, I split documents into chunks, generate embeddings using Amazon Bedrock, and store the vectors with the original text and metadata in OpenSearch. During a user query, I generate an embedding for the question, retrieve the top-K relevant chunks using vector similarity search, and pass those chunks along with the question to a Bedrock LLM to generate a grounded answer."

### 🔥 Final Mental Model

```
              INGESTION

Document
   |
   v
Chunk
   |
   v
Embedding
   |
   v
OpenSearch

                QUERY

Question
   |
   v
Embedding
   |
   v
Vector Search
   |
   v
Top-K Chunks
   |
   v
Question + Context
   |
   v
Bedrock LLM
   |
   v
Answer
```

**RAG = Retrieve + Augment + Generate**

- 🟣 Embedding Model = Convert meaning into numbers.
- 🔵 OpenSearch = Find similar information.
- 🟠 LLM = Use the information to generate an answer.
- ⚡ RAG = Find relevant information first, then generate an answer using that information.

---

## 📚 29. Official Documentation

| Resource | Documentation |
| --- | --- |
| 🟠 Amazon Bedrock | [AWS Bedrock User Guide](<https://docs.aws.amazon.com/bedrock/>) |
| 🟣 Titan Embeddings | [Titan Embeddings Models](<https://docs.aws.amazon.com/bedrock/latest/userguide/titan-embedding-models.html>) |
| 🔵 Amazon OpenSearch Service | [OpenSearch Service Documentation](<https://docs.aws.amazon.com/opensearch-service/>) |
| 🔍 OpenSearch Vector Search | [Vector Search Documentation](<https://docs.opensearch.org/latest/vector-search/>) |
| 🐍 Boto3 | [AWS SDK for Python](<https://boto3.amazonaws.com/v1/documentation/api/latest/index.html>) |
| 📝 GitHub Markdown | [GitHub Flavored Markdown](<https://github.github.com/gfm/>) |
| 📊 Mermaid | [Mermaid Documentation](<https://mermaid.js.org/>) |

---


