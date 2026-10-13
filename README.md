# FINOVA — Goal-Based Personal Finance Planning Engine

Flask + vanilla JS web app. **Python calculates; the LLM (Gemini) parses language, reasons and explains.**

## Run locally
    pip install -r requirements.txt
    cp .env.example .env        # add GEMINI_API_KEY
    python app.py               # http://localhost:5000

## Deploy on Render
1. Push this folder to GitHub, then Render -> New -> Blueprint (reads `render.yaml`).
2. In the dashboard set `GEMINI_API_KEY` (Google AI Studio) and `ADMIN_EMAILS`. The key is read server-side only.
3. `render.yaml` uses the **free** instance type and no disk, so it costs nothing. Trade-off: the free filesystem is wiped when the
   service restarts, spins down after idle time, or is redeployed, so accounts, plans and ratings are temporary (fine for a demo/class).
   To keep data you must pay for a Render Disk (set `DATA_DIR` to its mount path) or move the database to an external service.
4. Remove `ALLOW_DEMO_RESET` and set the `SMTP_*` variables when you want real emailed reset codes.

Without `GEMINI_API_KEY` everything still works: Python computes all figures and explanations are rule-based. The UI says so.

## The four LLM tasks (all validated by Python before use)
| Task | Where | Python validation |
|---|---|---|
| 1 Parse/validate inputs | Financial Information: "Describe your finances" and Excel upload | each extracted number must appear in the user's text; ranges, units, conflicts checked in `engine.py` |
| 2 Reason + prioritise goals | Savings & Investment | LLM must restate the exact Python ranking order and mention the primary goal; figures must be grounded |
| 3 Trade-offs of what-if / stress tests | Results, Savings | figures must match engine outputs; ungrounded fields are discarded |
| 4 RAG chatbot | Contact & Support | BM25 over `data/docs.md` + FAQ, plus the signed-in user's own data only; verbatim system instructions in `ai.py` |

Errors (timeout, 429, 5xx, bad key, blocked, invalid JSON) are retried where sensible, never leak the key, and fall back to Python-generated explanations with a visible notice.

## Tests
    python -m unittest discover -s tests -p "test_*.py"        # 40 tests: engine, exact-arithmetic verification, API, security, mock-Gemini
    pip install playwright && playwright install chromium
    python tests/e2e_server.py &  python tests/e2e_browser.py   # full browser walk-through against a mock Gemini

`tests/mock_gemini.py` is a local fake of the Gemini endpoint with failure modes. **No test uses a real Gemini key**; test with your real key after deploying.

## Structure
`engine.py` calculations/validation/Excel · `verify.py` independent checks · `narrative.py` rule-based text · `ai.py` Gemini + RAG · `app.py` API/security · `static/` front end · `data/` FAQ + docs
