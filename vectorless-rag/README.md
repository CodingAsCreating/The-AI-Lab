# Vectorless Reasoning-Based RAG

A from-scratch, educational implementation of **vectorless RAG** — retrieval-augmented generation that uses LLM reasoning over a document's natural structure instead of embeddings and a vector database.

Inspired by Microsoft's writeup on [Vectorless Reasoning-Based RAG](https://techcommunity.microsoft.com/blog/azuredevcommunityblog/vectorless-reasoning-based-rag-a-new-approach-to-retrieval-augmented-generation/4502238) and Vectify AI's [PageIndex](https://github.com/VectifyAI/PageIndex).

## What's different from "classic" RAG?

| Classic vector RAG | Vectorless reasoning RAG |
|---|---|
| Chunk → embed → store in vector DB | Keep natural pages/sections |
| Cosine similarity to retrieve top-k | LLM reasons over a tree-of-contents to pick pages |
| Opaque "why was this retrieved?" | Fully traceable: the LLM names the pages it chose and why |
| Needs embedding model + vector store | Just an LLM and a PDF |

The tradeoff: more LLM calls per query (you're paying for reasoning instead of a `pgvector` lookup). Great for small/medium document corpora where explainability and accuracy matter more than millisecond latency — finance, legal, compliance, research.

## Sample document

We use **[NIST AI 600-1: Artificial Intelligence Risk Management Framework — Generative AI Profile](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf)** (U.S. NIST, public domain). It's:

- On-topic for GenAI
- Heavily structured (numbered sections, subsections, page-aligned content)
- Public domain, so it's safe to redistribute in this repo

A copy is included at [`data/nist_ai_600-1.pdf`](data/nist_ai_600-1.pdf).

## Quickstart

```bash
cd vectorless-rag
pip install -r requirements.txt
export ANTHROPIC_API_KEY=sk-ant-...   # or put it in a .env file
jupyter notebook vectorless_rag.ipynb
```

Then run the cells top-to-bottom. The notebook walks through:

1. **Load & page-split** the PDF (no chunking, no embeddings)
2. **Build a tree-of-contents** — one short LLM-generated summary per page
3. **Reasoning-based retrieval** — give the ToC to an LLM, ask which pages are relevant
4. **Answer synthesis** — read the chosen pages, produce a cited answer

## Notes

- Defaults to `claude-haiku-4-5` for cheap indexing and `claude-sonnet-4-6` for answers. Swap the constants at the top of the notebook to use any other Claude model.
- Indexing the sample PDF (~64 pages) costs roughly one Haiku call per page. A pre-built `data/toc.json` is committed so you can skip indexing entirely and jump straight to retrieval — delete that file to re-run the indexing step yourself.
- The committed notebook already has outputs from a real run, so you can read through it on GitHub without executing anything.
