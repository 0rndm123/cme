# Private knowledge folder

Configure `knowledge.directory` in your ignored `config.local.json`.

Use `../cme_private` for a folder outside CME, or `private_knowledge` for a folder inside CME. The internal folder is listed in `.gitignore`.

The chatbot scans `.md` and `.json` files recursively on each request. Existing Markdown parsing expects `## id` headings and `PT:` / `EN:` lines. Full free-form Markdown and RAG have not been implemented.

Do not force-add private documents. Verify with `git status` before pushing. Files previously committed remain in Git history even after adding ignore patterns.
