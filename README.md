# LangGraph Streamlit Chatbot

A simple conversational chatbot built with **LangGraph** and **Streamlit**, powered by NVIDIA's hosted `openai/gpt-oss-20b` model. Conversation memory is handled by a LangGraph checkpointer, and the UI keeps the chat history visible across interactions.

## Features

- Chat interface built with Streamlit's `st.chat_input` and `st.chat_message`
- Stateful conversation flow using LangGraph `StateGraph`
- Conversation memory per thread via `InMemorySaver`
- LLM access through NVIDIA AI Endpoints (`ChatNVIDIA`)
- API keys loaded safely from a `.env` file

## Tech Stack

| Component                                                                                | Purpose                                       |
| ---------------------------------------------------------------------------------------- | --------------------------------------------- |
| [Streamlit](https://streamlit.io/)                                                       | Web UI                                        |
| [LangGraph](https://langchain-ai.github.io/langgraph/)                                   | Graph-based conversation workflow and memory  |
| [LangChain Core](https://python.langchain.com/)                                          | Message types (`HumanMessage`, `BaseMessage`) |
| [langchain-nvidia-ai-endpoints](https://pypi.org/project/langchain-nvidia-ai-endpoints/) | NVIDIA-hosted LLM integration                 |
| python-dotenv                                                                            | Environment variable management               |

## Project Structure

```
.
├── streamlit_frontend.py    # Streamlit UI
├── langgraph_backend.py     # LangGraph chatbot (graph, LLM, checkpointer)
├── requirements.txt
├── .env                     # Your API key
└── README.md
```

> **Note:** The frontend imports `from langgraph_backend import chatbot`, so the backend file must be named `langgraph_backend.py`.

## Prerequisites

- Python 3.10 or higher
- An NVIDIA API key (free from [build.nvidia.com](https://build.nvidia.com/))

## Setup

1. **Clone the repository / copy the files**

   ```bash
   git clone <your-repo-url>
   cd <your-repo-folder>
   ```

2. **Create and activate a virtual environment**

   ```bash
   python -m venv venv

   # Windows
   venv\Scripts\activate

   # macOS / Linux
   source venv/bin/activate
   ```

3. **Install dependencies**

   ```bash
   pip install -r requirements.txt
   ```

4. **Configure your API key**

   Create a `.env` file in the project root:

   ```env
   NVIDIA_API_KEY=nvapi-xxxxxxxxxxxxxxxxxxxxxxxx
   ```

## Running the App

```bash
streamlit run streamlit_frontend.py
```

The app opens at `http://localhost:8501`.

## How It Works

1. The user types a message in the Streamlit chat input.
2. The message is stored in `st.session_state['message_history']` so the UI can re-render past messages on each rerun.
3. The message is sent to the LangGraph `chatbot` with a fixed `thread_id`.
4. The graph runs a single `chat_node`, which calls the NVIDIA LLM with the full message history.
5. `InMemorySaver` stores the conversation state per thread, so the model remembers earlier turns.
6. The AI reply is displayed and appended to the UI history.

```
START ──► chat_node ──► END
```

## Known Limitations

- **In-memory storage:** `InMemorySaver` loses all history when the server restarts. Use `SqliteSaver` or `PostgresSaver` for persistence.
- **Single thread:** `thread_id` is hardcoded as `thread_1`, so all browser sessions share one conversation. Generate a unique ID per session (e.g., `uuid4()` stored in `st.session_state`) for multi-user use.
- **No streaming:** Responses appear only after the full reply is generated.

## Possible Improvements

- Stream tokens with `chatbot.stream(...)` and `st.write_stream`
- Add a "New Chat" button that creates a fresh `thread_id`
- Persist conversations with a SQLite or Postgres checkpointer
- Render replies with `st.markdown` instead of `st.text` for formatted output
- Add tools and an agent loop (web search, calculator, etc.)

## Troubleshooting

| Problem                                  | Fix                                                                                  |
| ---------------------------------------- | ------------------------------------------------------------------------------------ |
| `ModuleNotFoundError: langgraph_backend` | Make sure the backend file is named `langgraph_backend.py` and is in the same folder |
| Authentication / 401 error               | Check `NVIDIA_API_KEY` in `.env` and restart the app                                 |
| Model not found                          | Verify the model name is available on [build.nvidia.com](https://build.nvidia.com/)  |

## License

MIT
