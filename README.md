# PDF Chatbot

Upload a PDF, then ask questions about it. FastAPI + LangChain + LangGraph on the backend, React (Vite) on the frontend.

## Security note

Your original `_env` file had a real OpenAI key written in plain text. Rotate that key in your OpenAI dashboard if you haven't already, and never paste real keys into files you share.

## Which AI service this uses now

This project no longer uses OpenAI (which needs paid credits). Instead:
- **Embeddings** (used when you upload a PDF and when you ask a question) run **locally on your machine** via `sentence-transformers` — free, no account, no internet needed after the first model download (~90MB, one-time).
- **Answer generation** (turning retrieved text into a reply) uses **Hugging Face's free Inference API** — you need a free Hugging Face account and a free access token, but no billing.

### Get your free Hugging Face token
1. Go to https://huggingface.co/join and make a free account (if you don't have one).
2. Go to https://huggingface.co/settings/tokens
3. Click "New token", give it "Read" access, and copy it.

## Backend setup

```bash
cd backend
python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env          # then paste your HF token into .env
uvicorn main:app --reload
```

In `.env`, it should look like:
```
HUGGINGFACEHUB_API_TOKEN=hf_yourtokenhere
```

Backend runs at `http://localhost:8000`.

## Frontend setup

```bash
cd frontend
npm install
npm run dev
```

Frontend runs at `http://localhost:5173`.

## How it works

1. `POST /upload-pdf` — reads the PDF, splits it into chunks, embeds them, and adds them to a Chroma vectorstore that persists across requests (`backend/chroma_db/`).
2. `POST /ask` — runs a LangGraph pipeline (`graph.py`) with two nodes: `retrieve` (pulls the most relevant chunks for the question) and `generate` (asks a free Hugging Face-hosted model, `HuggingFaceH4/zephyr-7b-beta` by default, to answer using only those chunks). You can change the model by setting `HF_LLM_REPO_ID` in `.env`.
3. You can upload multiple PDFs — they all go into the same vectorstore, so questions are answered from everything uploaded so far. If you want per-document isolation instead, say so and I'll add a `document_id` filter.

## What's still worth adding

- Chat history isn't passed to the LLM yet, so follow-up questions ("what about the second one?") won't have prior context. Easy to add if you want it.
- No delete/reset endpoint for the vectorstore — right now it just keeps growing.
- No loading spinner/error styling beyond plain text.
