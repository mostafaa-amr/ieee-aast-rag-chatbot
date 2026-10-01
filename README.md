# IEEE AAST RAG Chatbot

A Python chatbot that answers questions about IEEE AAST Cairo societies, committees, recruitment, and branch rules using a handbook as its knowledge source.

Built as a student project using Retrieval-Augmented Generation (RAG), pretrained language models, and a streaming Gradio interface.

**This is an educational prototype, not an official IEEE service.**

## Demo

![IEEE AAST Assistant answering a handbook question](chatbot_demo.png)

## Overview

The chatbot retrieves relevant handbook pages before generating an answer. This provides the language model with document-specific evidence instead of relying only on its general knowledge.

The project uses pretrained models without fine-tuning.

## Features

- PDF text extraction with page metadata.
- Overlapping text chunks for semantic search.
- Embedding-based retrieval using cosine similarity.
- Full-page expansion to preserve surrounding context.
- Cross-encoder reranking of candidate pages.
- Answers prompted to stay within the supplied handbook context.
- Streaming responses through a Gradio chat interface.
- Retrieved page numbers displayed below responses.

## How It Works

### Document preparation

1. Upload the handbook PDF.
2. Extract text and preserve page numbers.
3. Remove repeated footers and unnecessary whitespace.
4. Split each page into chunks of up to 120 words with 25-word overlap.
5. Generate normalized embeddings for the chunks.

The tested handbook contained 10 pages and produced 21 chunks with 384-dimensional embeddings.

Embeddings are stored in a NumPy array. A separate vector database is unnecessary for this small document collection.

### Answering a question

1. Encode the question using the embedding model.
2. Search the chunk embeddings and retrieve the top 8 candidates.
3. Expand candidate chunks to their full source pages.
4. Remove duplicate pages.
5. Rerank the candidate pages against the question.
6. Select up to 2 pages as context.
7. Combine the context, question, and answering instructions.
8. Generate and stream the answer.
9. Display the retrieved page numbers below the response.

The model is instructed to acknowledge when the supplied context does not contain the requested information. This reduces unsupported answers but does not eliminate them.

## Models and Tools

| Component | Model or library | Role |
|---|---|---|
| PDF extraction | `pypdf` | Extract handbook text |
| Embeddings | `sentence-transformers/all-MiniLM-L6-v2` | Encode chunks and questions |
| Similarity search | NumPy | Rank chunks using normalized embeddings |
| Reranking | `cross-encoder/ms-marco-MiniLM-L6-v2` | Rank candidate pages against the question |
| Answer generation | `Qwen/Qwen2.5-1.5B-Instruct` | Generate answers using retrieved context |
| Model execution | PyTorch and Transformers | Load and run models |
| Interface | Gradio | Display streaming responses |

The embedding model and reranker run on CPU. The language model runs on GPU using `float16`.

## Tested Environment

The chatbot was tested in Google Colab using an NVIDIA Tesla T4 GPU.

| Dependency | Tested version |
|---|---|
| pypdf | 6.18.1 |
| NumPy | 2.1.3 |
| PyTorch | 2.11.0+cu128 |
| Transformers | 5.16.1 |
| Sentence Transformers | 5.7.0 |
| Gradio | 6.26.0 |

`requirements.txt` records the main dependency versions. It specifies `torch==2.11.0`; the tested Colab installation used the CUDA build `2.11.0+cu128`.

The notebook includes its own installation cells. These do not automatically read `requirements.txt`, and some installation commands use version ranges or unpinned packages. Future runs may therefore install different versions.

The requirements file records the main packages, rather than a complete environment lock.

## Run in Google Colab

### What you need

- A Google account with access to Google Colab.
- A GPU runtime; the project was tested with a Tesla T4.
- An authorized copy of the handbook as a readable PDF.
- Internet access for package and model downloads.

### Steps

1. Download `IEEE_AASTMT_RAG_Chatbot.ipynb` from this repository.
2. Open Google Colab and upload the notebook.
3. Select **Runtime → Change runtime type → T4 GPU**.
4. Select **Runtime → Run all**.
5. Upload your authorized copy of the handbook when prompted.
6. Wait for package installation, model downloads, and tests to finish.
7. Open the Gradio link printed by the interface cell.
8. Ask a question about the handbook.

### Restarting the demo

The chatbot runs inside the Colab runtime. GitHub stores the project files; it does not host the running chatbot.

The Gradio link is temporary and depends on the Colab runtime remaining active. GPU availability is subject to Colab limits.

After the runtime is deleted or reset:

1. Reconnect to a GPU runtime.
2. Run the notebook cells in order.
3. Upload the handbook again when prompted.
4. Use the newly generated Gradio link.

## Knowledge Source

The project was developed using the IEEE AAST Cairo Student Branch handbook.

The handbook is not bundled with this repository. Users must provide an authorized copy when running the notebook.

The notebook extracts text from readable PDFs. Scanned documents would require an additional OCR step.

The chatbot can only retrieve information from the document uploaded during the current session. It does not browse the internet for updates.

## Example Questions

- What are the steps to join IEEE AAST Cairo?
- Which society teaches AI and machine learning?
- What is the difference between IES and PES?
- What does the branding team do?
- What happens after three unexcused absences?
- What is the exact recruitment deadline for 2026?

The deadline question tests how the chatbot handles information missing from the handbook.

Ask complete questions. Each question is processed independently; previous chat messages are not included in the model's context.

## Evaluation

The following sample tests were manually reviewed during development:

| Test | Observed result |
|---|---|
| Recruitment process | Included all four stages: application, HR screening, technical interview, and onboarding |
| AI and machine learning society | Identified the Computer Society |
| Three unexcused absences | Answered probation |
| Missing recruitment deadline | Declined to invent a date |
| Out-of-scope general-knowledge question | Declined to answer |
| Inline page citations | Inconsistent |

These are sample checks, not a comprehensive accuracy benchmark.

Initial testing revealed unsupported details in generated answers. Stricter instructions and examples improved behavior on the evaluated questions, but do not guarantee that every answer is correct.

Retrieved page numbers identify the context supplied to the model. They do not independently verify every statement in its response.

## Limitations

- Answers depend on the contents and quality of the supplied handbook.
- The model can still generate unsupported claims or decline an answer that is present in the document.
- Inline citations may be omitted or incorrect.
- Retrieved page numbers show supplied context, not independently verified support for every claim.
- Conversation history is displayed but is not used by the model.
- Broad questions may require evidence from more than two pages.
- The interface requires an active Colab runtime.
- GPU availability and runtime duration depend on Colab.
- Some notebook installation commands use version ranges or unpinned packages, so future installations may behave differently.
- The project has been tested in Colab; running it elsewhere may require changes to file uploading and GPU setup.

## Future Improvements

- More extensive retrieval and answer evaluation.
- Better citation validation.
- Conversation-aware retrieval for follow-up questions.
- Support for multiple documents.
- Persistent deployment.
- Align notebook installation commands with the recorded dependency versions.
- Record a complete, reproducible environment.

## Acknowledgments

Built using Qwen, Sentence Transformers, Hugging Face Transformers, PyTorch, NumPy, pypdf, and Gradio.
