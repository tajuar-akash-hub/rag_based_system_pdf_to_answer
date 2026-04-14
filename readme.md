# PDF RAG Chatbot Documentation

This project is a Streamlit-based Retrieval-Augmented Generation (RAG) chatbot for PDF files. It uses PDF text extraction, embeddings, similarity search, and a Groq model to answer user questions using only the uploaded document content.

## Project Structure

- `app.py`: Streamlit user interface and application flow.
- `rag_logic.py`: RAG utility functions for reading PDFs, splitting text, building embeddings, retrieving relevant passages, and calling the Groq API.
- `.env`: Local environment variables, including the Groq API key.
- `requirements.txt`: Python dependencies.

## How It Works

1. User uploads one or more PDF files in the Streamlit app.
2. The app reads and extracts text from each PDF.
3. Text is split into overlapping chunks.
4. Each chunk is converted into an embedding using Sentence Transformers.
5. The app stores chunks, embeddings, and sources in memory.
6. When the user asks a question, the app embeds the question, finds the most similar chunks, and constructs a prompt.
7. The prompt is sent to the Groq API, which returns an answer based on the retrieved context.
8. The chat history and retrieved context are displayed in the UI.

## Environment

The app loads environment variables from `.env` if present. The following variable is required:

- `GROQ_API_KEY`: Groq Cloud API key.

Optional variables:

- `GROQ_BASE_URL`: Base Groq URL, default is `https://api.groq.com`.
- `GROQ_MODEL`: Groq model name, default is `openai/gpt-oss-20b`.

Example `.env`:

```text
GROQ_API_KEY=api_key
GROQ_BASE_URL=https://api.groq.com
GROQ_MODEL=openai/gpt-oss-20b
```

## Running the App

1. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
2. Start the app:
   ```bash
   streamlit run app.py
   ```

---

# app.py Function Documentation

## `init_state()`

Purpose:
- Initialize Streamlit session state values used by the app.

What it does:
- Creates `messages` if not already present.
- Creates `vector_store` as an empty vector store structure.
- Creates `retrieved` to hold the latest retrieved context.

Why it matters:
- Session state keeps chat history and vector store data across UI interactions.

## `main()`

Purpose:
- Define the Streamlit UI and manage the main application flow.

What it does:
- Configures the page layout and title.
- Displays the sidebar settings for chunk size, overlap, retrieval count, and context display.
- Displays the PDF file uploader.
- Processes uploaded PDFs and builds the vector store.
- Shows warnings when no PDF is uploaded.
- Renders the chat history.
- Handles user questions and triggers retrieval + Groq API calls.
- Displays retrieved context and a clear chat button.

Why it matters:
- `main()` is the entrypoint that connects the UI to the RAG logic.

---

# rag_logic.py Function Documentation

## `load_env_file()`

Purpose:
- Load environment variables from a local `.env` file.

What it does:
- Reads `.env` line by line.
- Skips empty lines and comments.
- Parses `KEY=VALUE` pairs.
- Sets environment variables only if they are not already set.

Why it matters:
- Allows secure local configuration without hard-coding keys in code.

## `get_embedding_model()`

Purpose:
- Load the Sentence Transformer model used for embeddings.

What it does:
- Instantiates `SentenceTransformer("all-MiniLM-L6-v2")`.

Why it matters:
- Used by all embedding functions to convert text into numeric vectors.

## `read_pdf(file_bytes)`

Purpose:
- Extract plain text from PDF bytes.

What it does:
- Wraps raw PDF bytes in an in-memory stream.
- Reads PDF pages with `PyPDF2.PdfReader`.
- Concatenates text from each page.
- Normalizes whitespace.

Why it matters:
- Converts PDF documents to raw text for embedding and retrieval.

## `split_text(text, chunk_size=400, overlap=100)`

Purpose:
- Divide a long text into smaller overlapping chunks.

What it does:
- Splits text into words.
- Creates chunks of up to `chunk_size` words.
- Moves forward by `chunk_size - overlap` words for each chunk.

Why it matters:
- Smaller chunks improve retrieval accuracy and make prompt context manageable.

## `embed_texts(texts)`

Purpose:
- Convert a list of text chunks into normalized embeddings.

What it does:
- Loads the embedding model.
- Encodes text into vectors.
- Normalizes each vector to unit length.

Why it matters:
- Normalized embeddings allow cosine similarity to work correctly.

## `build_vector_store(uploaded_files, chunk_size=400, overlap=100)`

Purpose:
- Build an in-memory vector store from uploaded PDF files.

What it does:
- Reads and extracts text from each uploaded PDF.
- Splits each document into chunks.
- Stores chunk text and source labels.
- Computes embeddings for all chunks.
- Returns a dictionary containing chunks, embeddings, and sources.

Why it matters:
- This vector store is the core retrieval database used to answer user questions.

## `find_similar_chunks(question, store, top_k=4)`

Purpose:
- Retrieve the most relevant text chunks for a user question.

What it does:
- Embeds the question.
- Computes similarity scores against stored chunk embeddings.
- Selects the top `top_k` chunks by score.
- Returns the matching chunks with source labels and scores.

Why it matters:
- This is the retrieval step of RAG, selecting information the model should use.

## `format_context(retrieved)`

Purpose:
- Format retrieved chunks into a single prompt context.

What it does:
- Joins the retrieved chunk text and source labels with separators.

Why it matters:
- Creates clear source-aware context for the model prompt.

## `build_prompt(context, question)`

Purpose:
- Build the final text prompt sent to Groq.

What it does:
- Adds instructions to answer only from the context.
- Combines context and the user question into one prompt string.

Why it matters:
- Ensures the model knows to use retrieved document text and not hallucinate.

## `ask_groq(prompt_text)`

Purpose:
- Send the prompt to Groq Cloud and return the generated answer.

What it does:
- Reads the Groq API key from environment variables.
- Builds the correct Groq endpoint URL.
- Sends a POST request to `/openai/v1/responses`.
- Parses Groq's JSON response and returns the answer text.

Why it matters:
- This is the generation step of RAG, turning retrieved context into the final answer.

---

## Notes

- The app currently uses the Groq API via its OpenAI-compatible endpoint.
- The RAG flow is separated cleanly: UI in `app.py`, logic in `rag_logic.py`.
- The document retrieval is entirely based on uploaded PDF content.
- The model prompt includes only retrieved context, which enforces true RAG behavior.
