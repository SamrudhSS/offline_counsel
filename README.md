# ClauseGuard

**A private, on-device assistant that helps you understand a rental agreement.**
It explains clauses in plain language, shows the exact passage and page behind every statement, says so when the document is silent, and declines to give legal verdicts.

> **Not legal advice.** ClauseGuard is for preliminary document understanding. It is not a lawyer and cannot decide whether a clause is legal, valid or enforceable. Consult a qualified legal professional for decisions that matter.

Built for ZeroDay OSS Hacktoberfest 2026 (IIIT Dharwad), Problem Statement 2: Private Legal Document Triage Agent.

[TEAM: replace every [bracketed] item before submitting.]

---

## What it does

- Reads a **text-based PDF** (or `.txt`) rental agreement **on your own laptop**
- Answers questions with a **plain-language explanation**, a **verbatim quote** and the **page number**
- Separates **"stated in the document"** from **"possible interpretation"** and **"possible concern"**
- Says **"The document does not state this."** when nothing relevant is found
- Refuses questions such as "is this enforceable?" and suggests who to ask instead
- Warns when a document contains text that tries to instruct the AI
- Keeps everything **in memory** and can wipe the session with one click

## Scope and limits

| In scope | Not supported |
|---|---|
| Residential rental agreements | Employment contracts, notices, service agreements |
| English | Other languages |
| Text-based PDF and plain text | Scanned or image-only PDFs (clear message shown) |
| Preliminary understanding | Legal validity, enforceability, legal advice |

Known limitations: [fill in from `docs/TEST_RESULTS.md`].

## Privacy

Documents are processed locally. The AI model runs on your machine through Ollama on `127.0.0.1`. We tested with networking disabled and monitored connections during processing. Evidence: `docs/evidence/`. Full threat model, test procedure and the list of every network touchpoint: [`docs/PRIVACY.md`](docs/PRIVACY.md).

What we **cannot** protect against: a compromised device, physical access, operating-system swap files, or screenshots you take yourself.

## Requirements

| Item | Tested value |
|---|---|
| OS | [Windows 11 / Ubuntu ... ] |
| CPU / RAM | [Ryzen 7 7735HS, 16 GB] |
| GPU | [none / RTX 2050 4 GB] |
| Python | [3.11.x] |
| Ollama | [version] |
| Model | [e.g. `gemma3n:e2b` or `gemma4:e2b-it-qat`], size [x GB] |
| Speed | [x tokens/s, x s per answer] (see `docs/benchmarks.md`) |
| Peak RAM | [x GB] |

Minimum recommended: 16 GB RAM, about [x] GB free disk, no GPU required.

## Setup (needs internet; **no user documents involved**)

1. Install [Python 3.11+](https://www.python.org/), [Git](https://git-scm.com/) and [Ollama](https://ollama.com/).
2. Get the model (accept the model's licence where required):
   ```
   ollama pull [MODEL_TAG]
   ```
3. Clone and install:
   ```
   git clone [REPO_URL]
   cd clauseguard
   python -m venv .venv
   .venv\Scripts\activate          # macOS/Linux: source .venv/bin/activate
   pip install -r requirements.txt
   ```
4. Download the embedding model once:
   ```
   python scripts/download_embedding_model.py
   ```
5. Set the model name in `src/config.py` (or environment variable `CLAUSEGUARD_MODEL`).

## Run (works offline)

```
set HF_HUB_OFFLINE=1
set TRANSFORMERS_OFFLINE=1
streamlit run app.py --server.address 127.0.0.1
```
(macOS/Linux: use `export` instead of `set`.) Make sure Ollama is running and bound to `127.0.0.1`.

Try the sample leases in `sample_documents/` (all fictional).

## Reproduce the demo

See the script in [`docs/PROJECT_MASTER.md`](docs/PROJECT_MASTER.md) section 14. Demo recording: [link].

## How it works

```
PDF -> pdfplumber -> clause splitter -> local embeddings -> top passages
    -> local Gemma (Ollama) -> verification (quotes must match the document) -> answer
```
The model explains; code guarantees. Citations, refusals, silence and the disclaimer are enforced by code. Details: [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

## How Gemma is used

Gemma ([variant]) runs locally through Ollama and writes the plain-language explanations and classifies findings (obligation, right, penalty, deadline, termination, other) from the retrieved passages. It never decides what the document says, whether evidence exists, or whether to refuse; code does.

## Testing

```
pytest -q
```
Results, metrics and limitations: [`docs/TEST_RESULTS.md`](docs/TEST_RESULTS.md).

## Data and licences

- Sample documents: fictional, written by the team, released under the project licence.
- Embedding model: `sentence-transformers/all-MiniLM-L6-v2` ([licence per model card]).
- Gemma: [licence of the chosen variant; accept it from the official model card].
- No private or third-party documents are included in this repository.

## Our main technical contribution

[2-3 sentences, for example: a verifiable-citation pipeline where every quote shown to the user is checked against the extracted text, plus a repeatable offline-privacy test procedure.]

## Troubleshooting

| Problem | Fix |
|---|---|
| "Model not ready" | Start Ollama; run `ollama list`; run `ollama pull [MODEL_TAG]` |
| "Embedding model not found" | Run `python scripts/download_embedding_model.py` once while online |
| Very slow answers | Close other apps; use the smaller/QAT model; see `docs/benchmarks.md` |
| "Looks like a scanned document" | Scanned PDFs are not supported; use a text PDF |
| Port already in use | Close other Streamlit/Ollama instances |

## Contributing, security, conduct

See [CONTRIBUTING.md](CONTRIBUTING.md), [SECURITY.md](SECURITY.md) and [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).

## Licence

[MIT] (see [LICENSE](LICENSE)).
