
# Indian Industries Association (IIA) HR Policy Assistant

An end-to-end, production-grade **Retrieval-Augmented Generation (RAG) Chatbot** engineered to query, extract, and reference policies from the dense 47-page official **IIA Employee Manual**. Built with LangChain, FAISS, Sentence-Transformers, and Google Gemini, this system features a rigorous evaluation suite utilizing the **RAGAS** framework and custom **LLM-as-Judge** parameters.

## 📌 Project Overview

In business operations and HR administration, accuracy is non-negotiable. General-purpose LLMs struggle with domain-specific question answering because they lack access to internal documents and tend to confidently hallucinate numbers, dates, or specific regulatory clauses.

This project addresses this limitation by developing an intelligent, grounded Q&A system for the **Indian Industries Association (IIA)**. 

### Core System Behaviors:
1. **Grounded Answering:** Restricts generation strictly to retrieved text blocks to eliminate hallucination.
2. **Precise Citations:** Every answer dynamically references the exact page number (e.g., `(p. 19)`) of the source PDF.
3. **Strict Out-of-Domain Safety:** Automatically detects out-of-scope inquiries (e.g., external financial rules, general knowledge) and triggers a deterministic refusal response: *"I cannot find this in the IIA HR Policy document."*

---

## 🏗️ Architecture & Pipeline Design

```
[OFFLINE INGESTION PIPELINE]
  PDF Manual (47 pages) ➡️ Header Normalisation ➡️ Recursive Character Splitting (800 / 150)
  ➡️ Normalised Vector Embeddings (all-MiniLM-L6-v2) ➡️ Local FAISS Vector Store

[ONLINE GENERATION QUERY TIME]
  User Query ➡️ Query Vectorisation ➡️ Similarity Search (Top-4 Context Chunks) 
  ➡️ System Prompt Augmentation ➡️ Gemini-3.5-Flash-Lite ➡️ Cited Answer Output
```

- **Document Loading & Preprocessing:** Uses `PyPDFLoader` page-by-page. Applied custom regex filters to strip recurring headers and sanitize layouts.
- **Chunking Strategy:** Implemented `RecursiveCharacterTextSplitter` targeting syntactic paragraph and sentence boundaries to avoid splitting critical numeric constraints in half.
- **Vector Database:** Local `FAISS` index saved directly to disk, ensuring 100% reproducible retrieval setups without dependency on cloud vector services.
- **Generator:** Injected strict constraint rules into `Gemini-3.5-Flash-Lite` via `langchain-google-genai` to govern tone, precision, and fallback refusal.

---

## 📊 Rigorous Evaluation & Performance Benchmarking

The system was evaluated against **17 target benchmark queries** (14 complex In-Domain policy questions, 3 Out-of-Domain questions) using **RAGAS (0.2.15)** alongside an automated **LLM-as-Judge**.

### 📈 Performance Metrics

| Metric | Score | Key Indicated Capability |
| :--- | :---: | :--- |
| **Retrieval Hit@4 (In-Domain)** | **100%** | System successfully located the correct policy page for all test scenarios. |
| **Mean Reciprocal Rank (MRR)** | **0.917** | The single most relevant page chunk was retrieved at **Rank 1** in almost all queries. |
| **RAGAS Faithfulness** | **0.929** | Near-zero hallucination. Answers are completely grounded in source facts. |
| **RAGAS Answer Relevancy** | **0.719** | Answers remained direct and on-topic, though slightly penalised by safety refuels. |
| **RAGAS Context Precision / Recall** | **0.798 / 0.851** | Strong contextual boundaries, minimizing noisy information. |
| **Out-of-Domain Refusal Rate** | **100%** | Complete safety control. System refused all non-HR/unmentioned questions. |
| **In-Domain False Refusal Rate** | **7% (1 of 14)** | Highly conservative fallback thresholding. |
| **LLM-as-Judge Faithfulness / Relevance** | **4.71 / 5** | Secondary evaluation validating accuracy and precise adherence to context rules. |

### 🔬 Chunking Strategy Experimentation
We benchmarked multiple chunking strategies to optimize the vector search space:
* **Small (300 chars / 50 overlap):** 293 chunks | Hit@4: 1.00 | MRR: 0.911
* **Medium (800 chars / 150 overlap) [Selected]:** 125 chunks | Hit@4: 1.00 | **MRR: 0.917**
* **Large (1200 chars / 200 overlap):** 84 chunks | Hit@4: 1.00 | MRR: 0.821 (Suffered from semantic dilution)

---

## 💡 Key Learnings & AI Engineering Takeaways

- **Prompt-vs-Retrieval Balance:** A highly restrictive "do not guess" prompt dramatically lowers hallucinations (high Faithfulness) but can occasionally trigger false refusals (such as on Q3 discipline policies) if the context parser misses exact structural overlaps.
- **Granularity of Chunks:** Larger chunk configurations dilute the similarity score of pinpointed numeric values, lowering MRR performance. Tuning boundary splits is the highest leverage task in retrieval engineering.
- **Rate-Limiting in Free Tiers:** Production pipelines querying external LLM endpoints must implement robust exponential backoff handling to manage structural API resource exhaustion (429 codes) gracefully during bulk evaluation runs.
- **Evaluation Discrepancies:** Automated LLM-as-judge prompts are routinely more lenient than mathematical evaluation metrics like RAGAS. Developing custom benchmark datasets is crucial for production tracking.

---

## 🛠️ Installation & Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-username/iia-hr-rag-chatbot.git
   cd iia-hr-rag-chatbot
   ```
2. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```
3. **Configure Keys:**
   Set your Google Gemini API key as an environment variable:
   ```bash
   export GOOGLE_API_KEY="your_api_key_here"
   ```
4. **Run the local Chat CLI:**
   Uncomment and run the interface file inside python or run:
   ```python
   from model_setup import chat_cli
   chat_cli()
   ```
