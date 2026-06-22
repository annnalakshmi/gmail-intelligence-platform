# Architecture & Design Document

# 1. System Architecture

Frontend
↓
Webhook/API
↓
n8n Workflows
↓
Gmail API
Gemini API
NVIDIA NIM
Supabase + pgvector
↓
AI Chat Agent

---

# Workflow Architecture

## Workflow 1 : Gmail Sync

Schedule Trigger
↓
Gmail Get Many Messages
↓
Loop Over Items
↓
Extract Email Data
↓
Supabase Upsert

---

## Workflow 2 : Email Summarization

Webhook
↓
Gemini
↓
Summary
↓
Supabase Update

---

## Workflow 3 : Email Categorization

Webhook
↓
Gemini
↓
Category
↓
Supabase Update

---

## Workflow 4 : Thread Summary

Webhook
↓
Get Thread Messages
↓
Combine Messages
↓
Gemini
↓
Thread Summary
↓
Supabase Upsert

---

## Workflow 5 : Embeddings

Webhook
↓
Prepare Text
↓
NVIDIA NIM Embeddings
↓
Parse Vector
↓
Supabase Update

---

## Workflow 6 : AI Chat Agent (RAG)

Webhook
↓
Generate Query Embedding
↓
Vector Search
↓
Top K Emails
↓
Gemini
↓
Generate Answer
↓
Respond JSON

---

## Workflow 7 : Compose Email

Webhook
↓
Gemini
↓
Format Email
↓
Gmail Send
↓
Response

---

## Workflow 8 : Thread-aware Reply

Webhook
↓
Get Thread Messages
↓
Build Context
↓
Gemini Draft
↓
Reply Message
↓
Respond JSON

---

# 2. Database Schema

## emails table

Fields

- id
- message_id
- thread_id
- sender
- recipient
- subject
- body
- snippet
- summary
- category
- embedding
- history_id
- created_at

Indexes

- message_id unique
- thread_id
- vector index on embedding

---

## threads table

Fields

- thread_id
- thread_summary
- updated_at

---

# 3. AI Design

## Email Summarization

Gemini generates summaries for individual emails.

## Thread Summarization

Entire thread messages are combined and summarized.

## Embeddings

NVIDIA NIM generates vectors which are stored in pgvector.

## Retrieval Augmented Generation

Question
↓
Embedding
↓
Similarity Search
↓
Top K Emails
↓
Gemini
↓
Answer

## Hallucination Prevention

Prompt:

Answer only from the provided emails.

If information is not found, reply:

"I could not find that information in the email knowledge base."

## Source Attribution

Responses contain:

- Sender
- Subject
- Thread ID

---

# 4. Gmail API Strategy

## Initial Sync

Fetch all emails.

## Incremental Sync

Use historyId.

## Pagination

Use nextPageToken.

## Rate Limiting

429 responses handled using exponential backoff.

---

# 5. Technology Decisions

Workflow Engine : n8n

Database : Supabase

Vector Database : pgvector

LLM : Google Gemini

Embeddings : NVIDIA NIM

Backend : Node.js

Frontend : React

---

# 6. Trade-offs

Due to time limitations:

- Newsletter deduplication is partially implemented.
- Queue workers are simplified.
- Batch processing is minimal.

Future improvements:

- Redis queue
- Better reranking
- Multi-user support
- Streaming responses
