
# Si Lam's Digital Twin

This application is a Gradio chat interface that answers questions about Si
Lam's background, education, professional experience, certifications, and
interests. It uses retrieval-augmented generation (RAG) so responses are
grounded in the local text files under [`data/`](data/).

## Live Demo: https://huggingface.co/spaces/silam2026/digital-twin
## How it works

When `app.py` starts, it:

1. Reads the education, overview, and professional-experience documents.
2. Splits each document into chunks of approximately 300 characters with
   30-character overlap.
3. Creates OpenAI `text-embedding-3-small` embeddings for all chunks.
4. Stores the embeddings and source metadata in a persistent ChromaDB
   collection named `digital_twin`.
5. Clears and rebuilds the collection on every startup.
6. Launches a Gradio chat interface.

For each user message, the application:

1. Embeds the question.
2. Retrieves the three most relevant document chunks from ChromaDB.
3. Adds those chunks to the system context.
4. Sends the conversation to OpenAI `gpt-4.1-mini`.
5. Returns the generated response in the chat interface.

The system instructions tell the model not to invent information and to say
that it does not know when the local documents do not contain an answer.

## Features

- RAG over local personal and professional documents
- Conversation history through Gradio's `ChatInterface`
- OpenAI embeddings and chat completions
- Persistent local ChromaDB storage
- Optional Pushover notifications when the model uses the notification tool
- A dice-roll tool for testing function calling
- Example prompts in the Gradio interface

## Project files

```text
digital-twin/
├── app.py
├── requirements.txt
├── SILAM.jpg
├── README.md
└── data/
    ├── document_education.txt
    ├── document_overview.txt
    └── document_professional.txt
```

`chroma_db_digital_twin/` is generated at runtime when the application starts.
It contains local vector data and is not required as source input because the
collection is rebuilt from the files in `data/`.

## Requirements

- Python 3.10 or newer
- An OpenAI API key
- Network access to the OpenAI API
- The packages listed in [`requirements.txt`](requirements.txt)

The application currently uses these OpenAI models:

- `text-embedding-3-small` for document and query embeddings
- `gpt-4.1-mini` for chat responses

## Installation and startup

From this directory, create a virtual environment and install dependencies:

### Windows PowerShell

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
$env:OPENAI_API_KEY = "your-openai-api-key"
python app.py
```

### macOS/Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
export OPENAI_API_KEY="your-openai-api-key"
python app.py
```

Gradio attempts to open the application in a browser. If it does not, open
the local URL printed by Gradio in the terminal.

Always run the command from this directory. The application uses relative
paths such as `./data/document_overview.txt` and
`./chroma_db_digital_twin`.

## Environment variables

### Required

| Variable | Description |
|---|---|
| `OPENAI_API_KEY` | API key used for embeddings and chat completions |

The program exits immediately with `API Key is missing!` if this variable is
not set.

### Optional

| Variable | Description |
|---|---|
| `PUSHOVER_USER` | Pushover user key for notifications |
| `PUSHOVER_TOKEN` | Pushover application token for notifications |

If either Pushover variable is missing, the notification tool returns a
configuration message instead of sending a notification. The notification
request is sent to `https://api.pushover.net/1/messages.json`.

## Tool calls

The model is given two function tools:

### `send_notification`

Sends a Pushover message. It is intended for situations such as:

- A visitor wants to get in touch, hire, or collaborate.
- The model does not know the answer to a question about Si Lam.

Configure both Pushover environment variables before using this feature.

### `dice_roll`

Returns a random integer from 1 through 6. This is included as a simple
example of model-driven function calling.

## Example questions

- `Show me your certifications`
- `What is your background?`
- `Do you like soccer?`
- `What is your work experience?`

## Updating the knowledge base

Edit one or more files in [`data/`](data/) and restart the application. The
documents are read and embedded again during startup, and the existing
`digital_twin` collection is deleted and rebuilt.

Do not place API keys or other secrets in the data files. Review personal
information before sharing this application publicly.

## Troubleshooting

### `API Key is missing!`

Set `OPENAI_API_KEY` in the same terminal session used to launch `app.py`.

### File-not-found errors for the data files

Change to the directory containing `app.py` before running it:

```powershell
cd "AI Engineering Part 1\digital-twin"
python app.py
```

### Avatar image is not displayed

The code references `SILAM.JPG`, while the checked-in image is named
`SILAM.jpg`. This is normally tolerated on Windows but can fail on
case-sensitive filesystems. Rename the file or update the reference in
`app.py` so the capitalization matches.

### Startup takes longer than expected

The application regenerates embeddings for every source chunk on every
startup. This requires an OpenAI API request and may take time depending on
document size and network speed.

## Security and privacy

This application processes personal and professional information. Before
deploying it or publishing the repository:

- Review and sanitize the files under `data/`.
- Keep `OPENAI_API_KEY`, `PUSHOVER_USER`, and `PUSHOVER_TOKEN` out of source
  control.
- Do not commit `.env` files, virtual environments, or generated vector data.
- Confirm that sending unknown questions to a personal notification service is
  appropriate for your deployment.

## License

No license is currently specified for this application.
