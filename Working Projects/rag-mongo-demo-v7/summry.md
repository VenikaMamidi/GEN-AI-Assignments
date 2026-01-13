# RAG-Mongo-Demo-v7 Codebase Summary

## Summary of the Program
This is a Retrieval-Augmented Generation (RAG) system for generating test cases from user stories in a health care management application. It uses MongoDB for data storage, vector embeddings for semantic search, and LLMs (via Mistral and Groq APIs) for summarization and generation. The system supports hybrid search (BM25 + vector), reranking, deduplication, and automated test case creation.

## Flow of the Program
1. **Data Ingestion**: Excel files are converted to JSON (test cases and user stories) and stored in MongoDB with vector embeddings.
2. **Query Processing**: User inputs a query, which is preprocessed (normalized, synonyms).
3. **Hybrid Search**: Combines BM25 keyword search and vector similarity search in MongoDB.
4. **Reranking**: Uses Groq's cross-encoder for relevance scoring.
5. **Deduplication**: Removes similar results using cosine similarity (>0.95 threshold).
6. **Summarization**: Groq API generates concise/detailed summaries.
7. **Generation**: TestLeaf API creates JSON test cases based on prompts and context.
8. **Validation**: AJV validates generated JSON against schemas.
9. **Output**: Displays results in HTML format.

## Different Components
- **Client (React App)**: UI for search, data conversion, summarization, and prompt management.
- **Server (Express.js)**: API endpoints for search, summarization, file uploads, and metadata.
- **Scripts (Node.js Utilities)**: Data conversion, embedding generation, and LLM clients.
- **Data Layer**: MongoDB collections for test cases, user stories, and embeddings.

## Key Files and Functions

### Client (`client/` - React)
- `App.js`: Main app component with navigation tabs (Search, Summarize & Dedup, Prompt & Schema, etc.).
- `components/search/HybridSearch.js`: Handles query input, search type selection, and result display.
- `components/processing/SummarizationDedup.js`: Manages search, deduplication, summarization, and cost tracking.
- `components/processing/PromptSchemaManager.js`: Configures prompts, schemas, and generates test cases via API.
- `components/data/ConvertToJson.js`: Uploads Excel files and converts to JSON via server API.

### Server (`server/index.js` - Node.js/Express)
- Main server file with endpoints:
  - `/api/search`: Hybrid search (vector/BM25).
  - `/api/search/bm25`: BM25-only search.
  - `/api/search/hybrid`: Combined search.
  - `/api/search/summarize`: Summarizes results using Groq.
  - `/api/test-prompt`: Generates test cases via TestLeaf API.
  - `/api/upload-excel`: Converts Excel to JSON.
  - `/api/metadata`: Fetches modules, priorities, etc., from MongoDB.
  - `/api/files`: Lists JSON files in data directory.

### Scripts (`src/scripts/` - Node.js)
- `utilities/mistralEmbedding.js`: Generates embeddings using Mistral API (`generateEmbedding`, `generateBatchEmbeddings`).
- `utilities/groqClient.js`: Handles reranking (`rerankDocuments`), summarization (`summarizeResults`), and generation (`generateAnswer`).
- `embeddings/create-userstories-embeddings-batch-mistral.js`: Batch processes user stories for embeddings.
- `data-conversion/excel-to-json.js`: Converts Excel to JSON for test cases.
- `data-conversion/excel-to-userstories.js`: Converts Excel to JSON for user stories.

### Data (`src/data/` - JSON)
- `testcases.json`: Test case data with fields like module, id, title, steps, expectedResults, linkedStories.
- `stories.json`: User story data with fields like key, summary, description, status, priority, epic.
- Configuration files for BM25/vector indexes.

## APIs Used and Their Working
- **MongoDB**: Stores and queries test cases/user stories with vector search (via `$vectorSearch` aggregation).
- **Mistral API**: Generates embeddings for semantic search (input: text, output: 1024-dim vectors).
- **Groq API**: 
  - Reranking: Cross-encoder model scores query-document pairs.
  - Summarization: `llama-3.3-70b-versatile` model creates concise/detailed summaries.
  - Generation: `llama-3.3-70b-versatile` for answers (not directly used in generation step).
- **TestLeaf API**: Generates test case JSON from prompts (endpoint: `/api/test-prompt`, input: prompt + temperature, output: JSON test cases).
- **Express Server APIs**: Internal endpoints for search, upload, and processing.