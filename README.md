

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

**Retrieval-Augmented Generation (RAG)** is an AI architecture that combines information retrieval with Large Language Model (LLM) generation.

This project explains how to build a RAG pipeline using:

| Technology | Role |
| --- | --- |
| 🟠 Amazon Bedrock | Hosts embedding and text-generation models |
| 🟣 Amazon Titan Embeddings | Converts text into numerical vectors |
| 🔵 Amazon OpenSearch | Stores embeddings and retrieves similar documents |
| 🐍 Python | Connects all components |
| 🧠 RAG | Grounds generated answers in retrieved information |

### 🎯 Project Goal

Build an application that answers user questions using information retrieved from your own documents instead of relying only on an LLM's trained knowledge.

### ⚡ Core Concept

```
          USER QUESTION
                |
                v
       +------------------+
       | Python App       |
       +------------------+
                |
                v
       +------------------+
       | Titan Embeddings |
       +------------------+
                |
                v
          QUERY VECTOR
                |
                v
       +------------------+
       | OpenSearch       |
       | Vector Search    |
       +------------------+
                |
                v
        TOP-K TEXT CHUNKS
                |
                v
       +------------------+
       | Question+Context |
       +------------------+
                |
                v
       +------------------+
       | Bedrock LLM      |
       +------------------+
                |
                v
          FINAL ANSWER
```

---

## 📑 Table of Contents

- 01 — What Is RAG?
- 02 — System Architecture
- 03 — Two Bedrock Operations
- 04 — Data Ingestion Pipeline
- 05 — Document Chunking
- 06 — Generate Embeddings
- 07 — Store Vectors in OpenSearch
- 08 — Query Processing
- 09 — Vector Similarity Search
- 10 — Build the RAG Prompt
- 11 — Generate the Final Answer
- 12 — Complete Architecture Diagram
- 13 — Example OpenSearch Document
- 14 — What Is Top-K?
- 15 — Why Do We Need Each Component?
- 16 — Suggested Project Structure
- 17 — Important Concepts
- 18 — Interview Questions
- 19 — Future Improvements
- 20 — Official Documentation

---

## 🧠 01 — What Is RAG?

**RAG = Retrieval-Augmented Generation**

RAG combines two major operations:

- 🔍 **Retrieval:** Find relevant information from a knowledge base.
- ⚡ **Generation:** Use an LLM to generate an answer from the retrieved information.

### 💬 Example Scenario

**User asks:**

> What is Amazon S3?

**Retrieved context:**

```
Amazon S3 is an object storage service.

S3 provides scalable storage for data.

S3 stores objects inside buckets.

S3 supports multiple storage classes.
```

**Generated answer:**

> Amazon S3 is an AWS object storage service that provides scalable data storage. It stores objects inside buckets and supports multiple storage classes for different access requirements.

### 🔥 Why RAG?

| Traditional LLM | RAG Application |
| --- | --- |
| Primarily relies on learned knowledge and prompt input | Retrieves relevant external information |
| May lack private company information | Can use indexed private documents |
| Can produce unsupported claims | Can be instructed to ground answers in evidence |
| May not know recent internal changes | Can retrieve updated documents after reindexing |

> ⚠️ RAG improves access to relevant information but does not guarantee accuracy. Poor retrieval or misleading source documents can still produce incorrect answers.

---

## 🏗️ 02 — System Architecture

```
flowchart TD
    U["👤 User Question"] --> APP["🐍 Python Application"]
    APP --> EMB["🟣 Amazon Titan Embeddings"]
    EMB --> QV["🔢 Query Vector"]
    QV --> OS["🔵 Amazon OpenSearch"]
    OS --> CH["📚 Top-K Relevant Chunks"]
    CH --> CTX["📝 Question + Context"]
    CTX --> LLM["🟠 Amazon Bedrock LLM"]
    LLM --> ANS["✅ Final Answer"]

    style U fill:#111827,stroke:#00F5FF,color:#FFFFFF
    style APP fill:#111827,stroke:#BD00FF,color:#FFFFFF
    style EMB fill:#111827,stroke:#BD00FF,color:#FFFFFF
    style QV fill:#111827,stroke:#00D4FF,color:#FFFFFF
    style OS fill:#111827,stroke:#00D4FF,color:#FFFFFF
    style CH fill:#111827,stroke:#00FF9C,color:#FFFFFF
    style CTX fill:#111827,stroke:#FF9900,color:#FFFFFF
    style LLM fill:#111827,stroke:#FF9900,color:#FFFFFF
    style ANS fill:#111827,stroke:#00FF9C,color:#FFFFFF
```

### 🧩 Component Responsibilities

| Component | Responsibility |
| --- | --- |
| User | Submits a question |
| Python application | Coordinates the pipeline |
| Titan Embeddings | Converts the question into a vector |
| OpenSearch | Searches the vector index |
| Retrieved chunks | Supply relevant source information |
| Bedrock LLM | Generates the final response |

---

## 🔄 03 — Two Bedrock Operations

A common beginner question is:

**Why do we call Bedrock twice?**

Because embeddings and generation perform different tasks.

### 🟣 Operation 1: Embedding

```
TEXT
  |
  v
TITAN EMBEDDING MODEL
  |
  v
NUMERICAL VECTOR
```

Used for:

- Semantic search
- Vector similarity
- Document retrieval

### 🟠 Operation 2: Generation

```
QUESTION + CONTEXT
        |
        v
    BEDROCK LLM
        |
        v
   NATURAL-LANGUAGE ANSWER
```

Used for:

- Question answering
- Summarization
- Text generation

**Remember:**

- Embedding model = converts text into vectors.
- OpenSearch = finds relevant information.
- LLM = generates an answer from the supplied context.

The embedding and generation models are separate logical operations. Depending on the selected models, they may be different model IDs.

---

## 📥 04 — Data Ingestion Pipeline

Before a user asks questions, documents must be prepared and indexed.

```
flowchart TD
    A["📄 Documents / PDFs"] --> B["Document Loader"]
    B --> C["Text Extraction"]
    C --> D["Chunking"]
    D --> E["Titan Embeddings"]
    E --> F["Document Vectors"]
    F --> G[("OpenSearch Vector Index")]

    style A fill:#111827,stroke:#00D4FF,color:#FFFFFF
    style B fill:#111827,stroke:#BD00FF,color:#FFFFFF
    style C fill:#111827,stroke:#BD00FF,color:#FFFFFF
    style D fill:#111827,stroke:#00F5FF,color:#FFFFFF
    style E fill:#111827,stroke:#BD00FF,color:#FFFFFF
    style F fill:#111827,stroke:#00FF9C,color:#FFFFFF
    style G fill:#111827,stroke:#00D4FF,color:#FFFFFF
```

### Ingestion Steps

1. Load documents such as PDFs, text files, or HTML.
2. Extract the document text.
3. Split the text into manageable chunks.
4. Generate an embedding for each chunk.
5. Store each chunk, vector, and metadata in OpenSearch.

Once ingestion is complete, the application can retrieve relevant chunks when users ask questions.

---

## ✂️ 05 — Document Chunking

Suppose we have this source document:

```
Amazon S3 is an object storage service.

Amazon S3 provides scalable storage for data.

S3 stores objects inside buckets.

Amazon S3 supports multiple storage classes.
```

We split the document into smaller chunks.

### Example Chunks

**Chunk 1**

```
Amazon S3 is an object storage service.
```

**Chunk 2**

```
Amazon S3 provides scalable storage for data.
```

**Chunk 3**

```
S3 stores objects inside buckets.
```

**Chunk 4**

```
Amazon S3 supports multiple storage classes.
```

### 🎯 Why Is Chunking Important?

- Smaller pieces can improve retrieval precision.
- Large documents become easier to index.
- The application can retrieve specific sections.
- The context sent to the LLM can be controlled.

**Important:** Chunk size, overlap, and splitting strategy affect retrieval quality. Real applications often use token limits and overlapping chunks rather than splitting only at sentence boundaries.

---

## 🔢 06 — Generate Embeddings

Each document chunk is passed to the embedding model.

```
CHUNK 1
   |
   v
TITAN EMBEDDINGS
   |
   v
VECTOR 1
```

The same process repeats for all chunks.

### Example Vectors

```
Chunk 1 -> [0.12, -0.34, 0.78, ...]
Chunk 2 -> [0.42,  0.18, -0.22, ...]
Chunk 3 -> [-0.15, 0.81, 0.34, ...]
Chunk 4 -> [0.52, -0.27, 0.63, ...]
```

_These values are illustrative and are not actual Titan model outputs._

### 🧠 What Is an Embedding?

An embedding is a numerical representation of text that captures useful semantic relationships.

Texts with related meanings can have vectors that are close under the configured similarity measure.

For example:

```
Query:
"Where can I store files in AWS?"

Retrieved document:
"Amazon S3 is an object storage service."
```

The words are different, but their meanings are related.

> ⚠️ Vector dimensions depend on the selected embedding model and its configuration. For example, certain Titan Text Embeddings models support 1,024-dimensional vectors. Configure the OpenSearch vector field to match the actual output dimension.

---

## 🗄️ 07 — Store Vectors in OpenSearch

OpenSearch stores the vector along with the original chunk text and associated metadata.

### Example Index Record

| Field | Description |
| --- | --- |
| `text` | Original chunk text |
| `vector` | Embedding generated by Titan |
| `source` | Source document |
| `page` | Page number |
| `chunk_id` | Unique chunk identifier |

### Conceptual Structure

```
OPEN SEARCH INDEX
|
+-- DOCUMENT 1
|   +-- Original Text
|   +-- Vector
|   +-- Metadata
|
+-- DOCUMENT 2
|   +-- Original Text
|   +-- Vector
|   +-- Metadata
|
+-- DOCUMENT 3
    +-- Original Text
    +-- Vector
    +-- Metadata
```

### Why Store the Original Text?

OpenSearch finds matching records using their vector representations. The application then retrieves the original text for the LLM.

```
QUERY VECTOR
     |
     v
OPENSEARCH SEARCH
     |
     v
MATCHING RECORD
     |
     v
ORIGINAL TEXT CHUNK
     |
     v
BEDROCK LLM
```

### Key Point

**The vector is used for retrieval; the original text is used as readable context.**

---

## 💬 08 — Query Processing

Now the user asks:

```
"What is Amazon S3?"
```

The Python application receives the question and generates its embedding.

```
USER QUESTION
      |
      v
PYTHON APPLICATION
      |
      v
TITAN EMBEDDING MODEL
      |
      v
QUERY VECTOR
```

The query embedding should be generated using a compatible model and configuration with the indexed document embeddings.

### Two Types of Embeddings

| Embedding | Input |
| --- | --- |
| Document embedding | A stored document chunk |
| Query embedding | The user's question |

Both represent text as vectors so that the application can search for related content.

---

## 🔍 09 — Vector Similarity Search

The query vector is sent to OpenSearch.

OpenSearch searches for vectors that are similar to the query according to the configured search method.

```
flowchart TD
    Q["🔢 Query Vector"] --> S["OpenSearch Vector Search"]
    S --> R1["Document Vector 1"]
    S --> R2["Document Vector 2"]
    S --> R3["Document Vector 3"]
    R1 --> SCORE["Similarity / Ranking"]
    R2 --> SCORE
    R3 --> SCORE
    SCORE --> TOP["🏆 Top-K Results"]
    TOP --> TXT["📚 Original Text Chunks"]

    style Q fill:#111827,stroke:#BD00FF,color:#FFFFFF
    style S fill:#111827,stroke:#00D4FF,color:#FFFFFF
    style SCORE fill:#111827,stroke:#FF9900,color:#FFFFFF
    style TOP fill:#111827,stroke:#00FF9C,color:#FFFFFF
    style TXT fill:#111827,stroke:#00F5FF,color:#FFFFFF
```

### Example Search Results

| Chunk | Example Score |
| --- | --- |
| Chunk 7 | 0.92 |
| Chunk 2 | 0.88 |
| Chunk 15 | 0.83 |
| Chunk 4 | 0.61 |
| Chunk 20 | 0.42 |

_These scores are illustrative, not benchmark results._

The application retrieves the most relevant records and uses their text to build the LLM context.

**Note:** The interpretation of a search score depends on the similarity metric and OpenSearch configuration. A score is not automatically a calibrated probability of correctness.

---

## 📝 10 — Build the RAG Prompt

The application combines the user's question with the retrieved chunks.

### Example Prompt

```
SYSTEM INSTRUCTION:

Answer the question using the supplied context.
Do not invent unsupported facts.
If the context does not contain the answer,
explain that the available information is insufficient.

CONTEXT:

Amazon S3 is an object storage service.

S3 provides scalable object storage.

S3 supports multiple storage classes.

QUESTION:

What is Amazon S3?
```

### Why Is the Context Important?

The context provides relevant information from the application's knowledge base.

The LLM uses it to generate a response grounded in the retrieved content.

---

## 🤖 11 — Generate the Final Answer

The application sends the question and context to a compatible text-generation model through Amazon Bedrock.

```
QUESTION + CONTEXT
        |
        v
   BEDROCK LLM
        |
        v
   FINAL RESPONSE
```

### Example Answer

> Amazon S3 is an AWS object storage service that provides scalable storage for data. It stores objects inside buckets and supports multiple storage classes for different access requirements.

The application returns the response to the user.

---

## 🌐 12 — Complete End-to-End Architecture

The full RAG system contains two connected pipelines.

```
flowchart TB
    subgraph ING["📥 INGESTION PIPELINE"]
        A["Documents / PDFs"] --> B["Text Extraction"]
        B --> C["Chunking"]
        C --> D["Titan Embeddings"]
        D --> E["Document Vectors"]
        E --> F[("OpenSearch Index<br/>Text + Vector + Metadata")]
    end

    subgraph QUERY["⚡ QUERY PIPELINE"]
        U["User Question"] --> Q["Query Embedding"]
        Q --> V["Query Vector"]
        V --> S["Vector Similarity Search"]
        S --> K["Top-K Chunks"]
        K --> P["Question + Context"]
        U --> P
        P --> L["Bedrock LLM"]
        L --> OUT["Final Answer"]
    end

    F --> S

    style A fill:#111827,stroke:#00D4FF,color:#FFFFFF
    style D fill:#111827,stroke:#BD00FF,color:#FFFFFF
    style F fill:#111827,stroke:#00D4FF,color:#FFFFFF
    style U fill:#111827,stroke:#00F5FF,color:#FFFFFF
    style Q fill:#111827,stroke:#BD00FF,color:#FFFFFF
    style S fill:#111827,stroke:#00D4FF,color:#FFFFFF
    style K fill:#111827,stroke:#00FF9C,color:#FFFFFF
    style L fill:#111827,stroke:#FF9900,color:#FFFFFF
    style OUT fill:#111827,stroke:#00FF9C,color:#FFFFFF
```

### Complete Formula

**Ingestion**

```
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
```

**Query**

```
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
Prompt
   |
   v
Bedrock LLM
   |
   v
Answer
```

---

## 📦 13 — Example OpenSearch Document

A simplified indexed record might look like this:

```
{
  "text": "Amazon S3 is an object storage service.",
  "vector": [0.12, -0.34, 0.78],
  "source": "s3-documentation.pdf",
  "page": 10,
  "chunk_id": "s3-page-10-chunk-1"
}
```

> ⚠️ The vector is abbreviated for readability. A real vector must contain the exact number of dimensions required by the embedding model and OpenSearch index mapping.

### Important Fields

- `text` — readable source content
- `vector` — numerical embedding
- `source` — document provenance
- `page` — source location
- `chunk_id` — chunk identity

The exact index mapping depends on your OpenSearch version and configuration.

---

## 🎯 14 — What Is Top-K?

Suppose your knowledge base contains 1,000 chunks.

Sending every chunk to the LLM would waste tokens and may exceed its context window.

Instead, retrieve only the most relevant chunks.

```
1,000 STORED CHUNKS
        |
        v
VECTOR SEARCH
        |
        v
TOP-K = 3
        |
        v
3 RETRIEVED CHUNKS
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

If `K = 3`, the application retrieves those three records.

### How Do You Choose K?

- A small K reduces context size but may omit useful information.
- A large K may provide more evidence but can introduce irrelevant text.
- The best value depends on document quality, question type, context limits, and evaluation results.

---

## ⚙️ 15 — Why Do We Need Each Component?

| Component | Main Purpose |
| --- | --- |
| 🟣 Titan Embeddings | Converts text into numerical vectors |
| 🔵 OpenSearch | Stores vectors and searches for similar records |
| 🟠 Bedrock LLM | Generates a natural-language answer |
| 🐍 Python | Orchestrates the pipeline |
| 📚 Documents | Provide source knowledge |
| ✂️ Chunking | Divides documents into searchable pieces |

### 🧠 The Most Important Mental Model

```
EMBEDDING MODEL
"Convert text into numbers"

OPENSEARCH
"Find relevant information"

LLM
"Generate an answer using context"

RAG
"Retrieve first, then generate"
```

### Vector Is Not Text

```
Vector
  |
  v
OpenSearch Search
  |
  v
Matching Record
  |
  v
Original Text Chunk
  |
  v
LLM Context
```

The LLM normally receives the retrieved text rather than just the embedding vector.

---

## 📁 16 — Suggested Project Structure

A possible Python project layout:

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

### Suggested Responsibilities

| File | Responsibility |
| --- | --- |
| `config.py` | Configuration management |
| `ingestion.py` | Document loading and indexing |
| `chunking.py` | Text splitting |
| `embeddings.py` | Calls the embedding model |
| `opensearch_client.py` | Connects to OpenSearch |
| `retrieval.py` | Runs vector search |
| `generation.py` | Calls the text-generation model |
| `rag_pipeline.py` | Connects retrieval and generation |

_This is a suggested layout, not a claim that a working implementation is included in this README._

---

## 💡 17 — Important Concepts

### 1\. Embedding

Converts text into a vector representation.

### 2\. Vector Database

Stores and retrieves vector representations. OpenSearch supports vector search alongside other search capabilities.

### 3\. Semantic Search

Finds content based on semantic similarity rather than only exact keyword matches.

### 4\. Chunking

Splits documents into smaller pieces for indexing and retrieval.

### 5\. Top-K

Controls how many of the highest-ranked results are retrieved.

### 6\. Prompt Augmentation

Combines the question with retrieved source text before generation.

### 7\. Grounded Generation

Instructs the LLM to answer using the retrieved evidence rather than inventing unsupported details.

### 8\. Hybrid Search

Combines keyword search and vector search to improve retrieval for some workloads.

---

## 🎤 18 — Interview Questions

\<details\> \<summary\>\<b\>Q1. What is RAG?\</b\>\</summary\>

RAG stands for Retrieval-Augmented Generation. It retrieves relevant external information and provides it to an LLM as context to generate a grounded response.

\</details\>

\<details\> \<summary\>\<b\>Q2. Why use Amazon Bedrock?\</b\>\</summary\>

Amazon Bedrock provides access to supported foundation models and embedding models through managed APIs, enabling applications to build generative AI workflows without hosting the models themselves.

\</details\>

\<details\> \<summary\>\<b\>Q3. Why use Titan Embeddings?\</b\>\</summary\>

Titan Embeddings converts text into vectors that can be indexed and searched using vector similarity.

\</details\>

\<details\> \<summary\>\<b\>Q4. Why use OpenSearch?\</b\>\</summary\>

OpenSearch stores document vectors and associated text and metadata, then retrieves relevant records using vector search and other search techniques.

\</details\>

\<details\> \<summary\>\<b\>Q5. Why call Bedrock twice?\</b\>\</summary\>

One operation generates embeddings for retrieval, while another invokes a text-generation model to produce the final answer.

\</details\>

\<details\> \<summary\>\<b\>Q6. What is Top-K?\</b\>\</summary\>

Top-K is the number of highest-ranked retrieval results selected to provide context to the LLM.

\</details\>

\<details\> \<summary\>\<b\>Q7. Why do we store the original text?\</b\>\</summary\>

The vector supports similarity search, while the original text provides readable evidence for the generation prompt.

\</details\>

\<details\> \<summary\>\<b\>Q8. What is the difference between keyword search and vector search?\</b\>\</summary\>

Keyword search focuses on matching terms, while vector search retrieves records according to relationships represented by embeddings. Hybrid search can combine both approaches.

\</details\>

\<details\> \<summary\>\<b\>Q9. Does RAG eliminate hallucinations?\</b\>\</summary\>

No. RAG can improve grounding, but incorrect retrieval, incomplete context, or poor generation can still result in hallucinations.

\</details\>

\<details\> \<summary\>\<b\>Q10. How can you improve RAG quality?\</b\>\</summary\>

Improve chunking, evaluate embedding models, tune retrieval settings, use hybrid search or reranking, enforce access controls, and measure retrieval and answer quality.

\</details\>

### ⭐ Interview-Ready Explanation

> During ingestion, I split documents into chunks, generate embeddings using Amazon Bedrock Titan Embeddings, and store the vectors along with the original text and metadata in Amazon OpenSearch. When a user asks a question, I generate an embedding for the query, perform vector similarity search to retrieve the top-K relevant chunks, and pass those chunks with the question to a Bedrock LLM to generate a context-grounded answer.

---

## 🚀 19 — Future Improvements

Potential enhancements for a production-grade RAG application:

- 🔎 Hybrid keyword and vector search
- 🎯 Reranking of retrieved chunks
- 📚 Source citations and document references
- ✂️ Improved chunk sizing and overlap
- 🔐 Document-level access controls
- 📊 Retrieval and answer-quality evaluation
- ⚡ Caching and latency optimization
- 🛡️ Prompt-injection defenses for retrieved documents
- 📈 Monitoring, logging, and cost tracking

---

## 📚 20 — Official Documentation

| Resource | Link |
| --- | --- |
| 🟠 Amazon Bedrock | [Official Documentation](<https://docs.aws.amazon.com/bedrock/>) |
| 🟣 Titan Embeddings | [Embedding Models](<https://docs.aws.amazon.com/bedrock/latest/userguide/titan-embedding-models.html>) |
| 🔵 Amazon OpenSearch Service | [Official Documentation](<https://docs.aws.amazon.com/opensearch-service/>) |
| 🔍 OpenSearch Vector Search | [Vector Search Documentation](<https://docs.opensearch.org/latest/vector-search/>) |
| 📝 GitHub Markdown | [GitHub Flavored Markdown](<https://github.github.com/gfm/>) |
| 📊 Mermaid Diagrams | [Mermaid Documentation](<https://mermaid.js.org/>) |

---

\<p align="center"\> \<b\>⚡ RETRIEVE THE KNOWLEDGE. AUGMENT THE CONTEXT. GENERATE THE ANSWER. ⚡\</b\> \<br/\>\<br/\> \<img src="https://img.shields.io/badge/BUILD-00F5FF?style=for-the-badge&labelColor=111827" alt="Build"/\> \<img src="https://img.shields.io/badge/RETRIEVE-BD00FF?style=for-the-badge&labelColor=111827" alt="Retrieve"/\> \<img src="https://img.shields.io/badge/GENERATE-00FF9C?style=for-the-badge&labelColor=111827" alt="Generate"/\> \</p\>

## How to publish it on GitHub

1. Open your GitHub repository.
2. Open `README.md`, or create that file if it does not exist.
3. Paste the complete content above and save or commit the changes.
4. Open the repository's main page to see the rendered README.

Design tip: GitHub controls the page background and sanitizes unsupported HTML/CSS. The dark/neon effect comes from the dark-styled Mermaid nodes, bright badges, and emoji accents; the rest of the README adapts to GitHub's theme.
