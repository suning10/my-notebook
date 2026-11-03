# RAG performance has decreased 

- to evaluate a RAG [eval](../evaluateRag.md)

Step 1 

Identify the issue 
- pull from logs or user feedback 
- create test cases with known answer 

Step 2
Identify what has been changed
- Data: broken parsing / re-ingestion(indexing) / duplicates / inactive documents 
- Model(Embedding) does the model change
- Chunking: meata data missing (extract incorrectly) chunk size overlap 
- queries, if user tend to ask different questions 
  - LLM to develop new skills 
- Retrieval 
  - top-k 
  - reranker 
  - similarity threshold

Step 3 
Find which stage
- Retrieval 
  - Recall @5 low?
  - MRR low? - retrieved but rank low
  - Resolve
    - Hybrid + BM25 + Reranker(bge-reranker) + Query Rewrite + semantic chunking + metadata filter
- generation(retrieved but generate wrong)
  - update the prompt instruction 
  - Haullucination 
    - detect: Score faithfulness with DeepEvals (or simple LLM as a Judge)
    - instruct the LLM to use prvoided context, if not there say no
  - Answer from its own knowledge instead of retrieved 
  - conflicts 
    - include metadata