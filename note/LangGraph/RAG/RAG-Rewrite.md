# Summary

## rewrite Query

- why
  - users query is very broad or plain english, not specific
  - Query have too few tokens
  - vocabulary missmatch
  - multi-turn query
    - need context
    - eg. what is pricing
    - prior turn ask about product x
- How
  - Use a lightweight classifier
    - to **Save Tokens**
    - Simple, clear question
    - Complex, vague question
  - Rewrite
    - Use LLM
    - concurrent firing similar questions
    - decompose the question
      - summarize after gather all the information
    - use LLM to answer the question
      - use answer to do query
- Common Mistake
  - Rewrite completely different semantics
    - validator to check similarity

## when to use 
  - simple rewrite 
    - determine if it is multi round 
    - add glossory 
```python
Is it a chatbot with conversation history?
  YES → always rewrite (condense history + query)
  NO  → skip rewrite entirely

Measure retrieval quality first
Add complexity only if you have data showing it helps
```

## a better design with fallback rewrite
```python
Query
  ↓
[1] Multi-round check → inject history
  ↓
[2] Regex glossary expansion
  ↓
[3] Embed + Vector Search
  ↓
[4] Threshold check
  PASS → rerank → inject → LLM answer
  FAIL → LLM rewrite → re-search
           ↓
        still FAIL → "no information found"
```


## Hybrid Search

- BM25 + Vector Search
  - Vector search top k
  - BM25 rerank this top k
  - RRF (Reciprocal Rank Fuse)
