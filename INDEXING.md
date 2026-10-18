# Local document index

The CME prototype indexes files from the folder set as `knowledge.directory` in the private `config.local.json`. Use `knowledge.retrieval: fts5`.

Supported formats: Markdown, TXT, HTML, HTM, DOCX, XLSX, XLS, XLSM, CSV, JSON.

Open the local chat UI and choose **Reindexar documentos locais**. Data is extracted into the local SQLite FTS5 index at `state/knowledge_fts.sqlite3`; this index is excluded from Git and must itself be treated as private. Press reindex again after changing the source documents. Removed files disappear from the index at the next scan.

This is full-text retrieval, not semantic embeddings or RAG. Search by meaning, hybrid retrieval and stronger evidence verification are future work. Spreadsheet cells are indexed as text, without evaluating formulas. Scanned or image-based content is not processed. No data is uploaded to a remote vector database by the indexer.

Default safety restrictions: 10 MB per file, up to 1000 files. Local-only operation. Never publish the reindex endpoint without administrator authentication.
