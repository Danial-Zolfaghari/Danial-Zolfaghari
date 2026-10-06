# AI Data Analyzer

## Problem

Make a very large Elasticsearch environment usable through natural language for operators who should not need to know index names, field mappings, or Elasticsearch Query DSL.

The difficult part is not generating a query. The difficult part is ensuring that an LLM does **not invent schema, silently guess ambiguous terms, or produce an answer without evidence**.

## Design goals

- Natural-language input for non-technical users.
- Read-only access to Elasticsearch.
- Explicit clarification when a term has multiple plausible meanings.
- Deterministic query generation after planning.
- Evidence attached to every answer.
- Local-model support where infrastructure or privacy requires it.
- A persistent semantic layer that can learn mappings without treating hypotheses as facts.

## Architecture

```text
User question
    ↓
Ambiguity / clarification
    ↓
Retrieval
    ↓
Semantic World Map
    ↓
Planner
    ↓
Deterministic query representation
    ↓
Safe Elasticsearch execution
    ↓
Evidence collection
    ↓
Grounded answer
```

### 1. Clarification first

Ambiguous operator language is treated as a data-quality problem. If the system cannot map a phrase to a confirmed concept with sufficient confidence, it asks instead of guessing.

### 2. Semantic World Map

A persistent mapping layer links human concepts to observed schema knowledge.

Entries can be confirmed, stale, incomplete, or hypothetical. Hypotheses do not automatically become trusted query inputs.

### 3. Planner, then deterministic execution

The model helps interpret intent and plan the analysis, but the final Elasticsearch request is generated through constrained deterministic logic rather than free-form model output.

### 4. Safe data access

The execution layer is read-only and validates generated operations before they reach Elasticsearch.

### 5. Evidence chain

The answer is downstream of retrieved evidence. The system keeps a trace from user intent → plan → query → result → response so unsupported claims can be rejected.

## Scale considerations

The design targets **multi-billion-document Elasticsearch environments** with large and evolving schemas. That changes the engineering problem: full-schema prompting is not viable, retrieval must be selective, and schema trust has to be maintained over time.

## Local AI stack

The system has been designed around local model workflows including:

- Ollama
- Qwen-family models
- DeepSeek-family models
- embedding models
- reranking
- retrieval / RAG-style components

Model choice is treated as replaceable infrastructure; the safety and evidence model lives in the pipeline around the LLM.

## Engineering lessons

- LLM confidence is not schema confidence.
- Retrieval quality matters more than sending more context.
- A semantic mapping layer needs lifecycle states, not just key/value pairs.
- Read-only enforcement belongs below the model layer.
- Evidence should be a first-class artifact, not an afterthought.

## Confidentiality

This case study intentionally omits internal schemas, field names, hostnames, addresses, customer information, and deployment-specific thresholds.
