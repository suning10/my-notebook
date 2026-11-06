# how to handle conflict 

- Situation:
  - SOP conflicts between department, eg. create xx 
  - May retrieve the chunks not relevant to user's department 
- Tasks
  - Resolve this issue
- Action: [conflict](../RAGConflict.md)
  - Add metadata when ingest the file
  - namespace / collection isolation (~db)
  - pull user context and inject user's department 



### metadata eg

```json
document_metadata = {
    "department": "procurement",
    "transaction_code":
    "process_type": 
    "sop_version": "2.1",
    "last_updated": "2024-01-15"
}

```