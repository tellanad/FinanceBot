# FinanceBot

FinanceBot is a small notebook project for exploring financial questions with AI. It brings together two useful starting points: learning from finance articles and looking up company news.

The idea is straightforward: give the bot some reading material, let it find the passages relevant to a question, and use those passages to help write an answer. A separate news agent experiments with calling a tool to fetch headlines when a question needs more recent information.

This is a work in progress. The main place to explore it is [`src/financebot.ipynb`](src/financebot.ipynb).

## What it does today

- Loads the first 10 chapter links from a Zerodha Varsity sitemap.
- Splits the page text into smaller chunks and stores their embeddings in Chroma.
- Uses retrieved passages and the current conversation history to answer follow-up questions.
- Provides a Gradio chat interface, currently titled **DocumentQABot**.
- Runs a separate company-news agent that returns headlines, links, publishers, and publication dates when available.

The Gradio chat uses the document retrieval chain. The news agent runs in the final notebook cell and is not connected to that chat interface yet.

## Getting started

You'll need Python 3.11, `uv`, an OpenAI API key with API access, and a notebook-capable editor such as VS Code or Antigravity. Internet access is needed to download articles, search news, and call the models.

### 1. Install the dependencies

From the project folder, run:

```bash
uv sync --locked
```

This creates the project's `.venv` and installs the versions recorded in `uv.lock`.

### 2. Add your API key

Create a file named `.env` in the project root, alongside `pyproject.toml`:

```dotenv
OPENAI_API_KEY=your-openai-api-key
```

Save the file before running the notebook. `.env` is already listed in `.gitignore`; keep the actual key out of notebook cells and outputs.

Embedding the documents and generating answers use the OpenAI API and can incur charges. Re-running the embedding cell sends the chunks for embedding again.

### 3. Select the notebook kernel

Open `src/financebot.ipynb`, click the kernel selector, and choose the Python environment at:

```text
.venv/bin/python
```

`ipykernel` is included in the dependencies. If the environment does not appear, you can register a named kernel:

```bash
uv run python -m ipykernel install --user --name financebot --display-name "Python (FinanceBot)"
```

Then select **Python (FinanceBot)** in your editor.

### 4. Run the notebook in order

Start with the imports and environment setup, then work down through document loading, embeddings, retrieval, and the chat interface. The first run takes longer because it downloads the pages and creates the embeddings.

The Gradio cell displays a chat window and prints a local URL. It currently launches with `share=True`, which also creates a public link. For local use, change that line to:

```python
demo.launch(share=False, debug=False)
```

The final cell runs the separate news agent. Its current example asks for news about **Micron**; change the company in the user message to try another search.

Running `python main.py` only prints a greeting. It does not start FinanceBot.

## How the pieces fit together

For document questions, the notebook follows this path:

```text
Varsity pages → text chunks → embeddings → Chroma → relevant passages → answer
```

The splitter uses a 1,000-character chunk size with a target overlap of 200 characters. The overlap helps keep nearby context together. For follow-up questions, a history-aware retriever first reformulates the question using the conversation so it can search for the right passages.

The answer model is `gpt-4o-mini`, with temperature set to `0`. Embeddings are created with `OpenAIEmbeddings()`; the installed version defaults to `text-embedding-ada-002`. The answer prompt asks for a short response based on the retrieved context.

The news agent uses LangChain's `create_agent` and a custom `company_news` tool. That tool searches Yahoo Finance through `yfinance.Search` using a company name. It returns headline metadata rather than downloading full articles, and distinguishes a failed lookup from an empty result.

## A few things to keep in mind

Only the first 10 sitemap pages are loaded, so the chatbot's reading material is limited. You can change `urls[:10]` to expand it, but that also increases download time and embedding usage.

There is no configured persistent Chroma directory or durable conversation store. Treat the index and chat history as temporary, and rebuild the notebook state after restarting the kernel.

News coverage depends on Yahoo's search results. An empty response does not mean that no news exists, and the returned headlines are not guaranteed to be from today. Follow the source links for the full story.

This is a learning prototype for exploring finance concepts, not a source of personalized investment advice. Check the source material before relying on an answer.

## If something goes wrong

**The API key is missing:** save `.env` in the project root and rerun the environment setup. To explicitly search from the notebook's working directory and reload the saved value, use:

```python
from dotenv import find_dotenv, load_dotenv

load_dotenv(find_dotenv(usecwd=True), override=True)
```

**A variable such as `llm`, `retriever`, or `rag_chain` is undefined:** run the earlier cells in order. Restarting the kernel clears those variables.

**A copied Gradio example fails:** this project uses Gradio 6. The chatbot no longer takes `type="messages"`, and a theme belongs in `demo.launch(theme="soft")`, rather than `gr.Blocks()`.

## Project layout

```text
FinanceBot/
├── src/
│   └── financebot.ipynb   # Document chatbot and company-news experiment
├── main.py               # Starter script; not the app entry point
├── pyproject.toml        # Project dependencies
├── uv.lock               # Locked dependency versions
├── .python-version       # Python 3.11
└── README.md
```

## Where this could go next

Useful next steps would be connecting the news tool to the chat interface, showing citations alongside document answers, saving the vector index between runs, and testing conversation history as the app grows.
