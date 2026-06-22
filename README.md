# HackerRank Resume Grader

An AI-powered web app that scores and annotates your resume using the same evaluation rubric HackerRank uses to screen software engineering intern candidates — giving you a transparent, actionable score before you ever apply.

---

## What It Does

Upload a PDF resume and get back:

- **Overall score out of 120** with a letter-grade tier (Excellent / Strong / Average / Needs Improvement)
- **Four category scores** matching HackerRank's actual hiring criteria
- **Color-coded resume viewer** where every section is highlighted by the category it contributes to, with the AI's evidence shown inline
- **Key strengths** and **areas to improve** written in plain English
- **Bonus points and deductions** breakdown (GSoC, startup experience, missing links, tutorial-only projects, etc.)

---

## Scoring Rubric

| Category | Max Points | What It Measures |
|---|---|---|
| Open Source | 35 | Contributions to external repos, GSoC, community involvement |
| Self Projects | 30 | Complexity, real-world impact, live demos, GitHub links |
| Production Experience | 25 | Internships, full-time roles, startup early-engineer credit |
| Technical Skills | 10 | Breadth of languages, frameworks, and demonstrated depth |
| Bonus Points | +20 | GSoC (+5), Girl Script SoC (+3), founder/co-founder (+5), LinkedIn (+1), portfolio (+2), blogs (+3) |
| Deductions | −20 | Tutorial-only projects, missing links, broken URLs, generic project names |

Total range: −20 to 120.

---

## How It Works

```
PDF Upload
    │
    ▼
PyMuPDF → Markdown text
    │
    ▼
Claude Haiku (×6 parallel calls)
    ├── basics extraction
    ├── work experience extraction
    ├── education extraction
    ├── skills extraction
    ├── projects extraction
    └── awards extraction
    │
    ▼
JSONResume structured object
    │
    ▼
Claude Haiku — evaluation call
(scored against HackerRank rubric via Jinja prompt templates)
    │
    ▼
EvaluationData: scores + evidence + strengths + improvements
    │
    ▼
Flask API → Frontend (color-coded viewer + score cards)
```

The grading logic comes directly from [interviewstreet/hiring-agent](https://github.com/interviewstreet/hiring-agent), HackerRank's open-source hiring pipeline. This project adds a web UI on top, extends the provider system to support Claude and OpenAI, and fixes template path resolution so it runs from any working directory.

---

## Setup

### 1. Clone and install dependencies

```bash
git clone https://github.com/todddong/hackerrank-resume-ats
cd hackerrank-resume-ats
pip install flask python-dotenv PyMuPDF pymupdf4llm pydantic jinja2 requests anthropic openai google-generativeai ollama
```

### 2. Configure your LLM provider

Create `hiring-agent/.env` — pick one provider:

**Anthropic Claude (recommended — ~$0.001 per resume)**
```env
LLM_PROVIDER=anthropic
DEFAULT_MODEL=claude-haiku-4-5-20251001
ANTHROPIC_API_KEY=sk-ant-...
```

**OpenAI**
```env
LLM_PROVIDER=openai
DEFAULT_MODEL=gpt-4o-mini
OPENAI_API_KEY=sk-proj-...
```

**Google Gemini**
```env
LLM_PROVIDER=gemini
DEFAULT_MODEL=gemini-2.0-flash
GEMINI_API_KEY=AIzaSy...
```

**Ollama (local, free)**
```env
LLM_PROVIDER=ollama
DEFAULT_MODEL=gemma3:4b
```
Requires [Ollama](https://ollama.com) installed and `ollama pull gemma3:4b`.

### 3. Run

```bash
python app.py
```

Open [http://localhost:5050](http://localhost:5050), drag in a PDF, and wait ~30 seconds.

---

## Project Structure

```
.
├── app.py                        # Flask server + /analyze endpoint
├── templates/
│   └── index.html                # Single-page UI (Tailwind CSS, vanilla JS)
└── hiring-agent/                 # HackerRank's evaluation engine
    ├── pdf.py                    # PDF → JSONResume via LLM section extraction
    ├── evaluator.py              # JSONResume → EvaluationData scoring
    ├── models.py                 # Pydantic models + LLM provider classes
    ├── llm_utils.py              # Provider initialization
    ├── prompt.py                 # Model registry + API key loading
    ├── transform.py              # Data normalization utilities
    └── prompts/templates/        # Jinja templates for each resume section
        ├── resume_evaluation_criteria.jinja   # The core scoring rubric
        ├── basics.jinja
        ├── work.jinja
        ├── education.jinja
        ├── skills.jinja
        ├── projects.jinja
        └── awards.jinja
```

---

## LLM Provider Support

| Provider | Models | Notes |
|---|---|---|
| Anthropic | `claude-haiku-4-5-20251001`, `claude-sonnet-4-6` | Recommended — uses prefill trick for reliable JSON |
| OpenAI | `gpt-4o-mini`, `gpt-4o`, `gpt-4.1-mini`, `gpt-4.1` | Uses structured output JSON schema |
| Gemini | `gemini-2.0-flash`, `gemini-2.5-flash`, `gemini-2.5-pro` | Requires paid-tier API key |
| Ollama | `gemma3:4b`, `qwen3:4b`, `mistral:7b`, others | Fully local, no API key needed |

---

## Credits

- Scoring rubric and evaluation pipeline: [interviewstreet/hiring-agent](https://github.com/interviewstreet/hiring-agent) by HackerRank
- PDF extraction: [PyMuPDF](https://pymupdf.readthedocs.io) + [pymupdf4llm](https://github.com/pymupdf/RAG)
