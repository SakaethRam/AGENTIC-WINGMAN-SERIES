# Setup and Configuration

## Clone

```bash
git clone https://github.com/SakaethRam/AGENTIC-WINGMAN.git
cd AGENTIC-WINGMAN
```

## Required credentials

1. **Gemini API key** — from Google AI Studio.
2. **Airtable Personal Access Token** — Airtable → Account → Personal Access Tokens.
3. **Airtable Base ID** — found in your base's URL, in the form `appXXXXXXXXXXXXXX`.

## A correction to the README's environment variable example

The upstream README's setup section has two problems worth fixing rather than copying forward:

```bash
# As written in the README — do not use this as-is:
export OPENAI_API_KEY="sk-..."
export Gemini_API ="<GEMINI_API_KEYS>"
export Airtable Personal Access Token ="<PERSONAL_ACCESS_TOKEN>"
export Airtable Base ID ="<BASE_ID>"
```

1. **`OPENAI_API_KEY` is the wrong provider.** The engine runs on Google Gemini (`gemini-2.0-flash`), not OpenAI. There's no code path in the documented architecture that would use an OpenAI key.
2. **Variable names with spaces are invalid shell syntax.** `export Airtable Personal Access Token ="..."` will fail outright in bash/zsh: environment variable names cannot contain spaces. This block as written will not run.

**Corrected version:**

```bash
export WINGMAN_GEMINI_API_KEY="<your-gemini-api-key>"
export WINGMAN_AIRTABLE_PAT="<your-airtable-personal-access-token>"
export WINGMAN_AIRTABLE_BASE_ID="app<your-base-id>"
```

## A second correction: credentials belong in environment variables, not in `StudioCode.py`

The README's run instructions show keys passed as literal strings directly into the constructor call:

```python
# WingMan X StudioCode.py
from wingmanx import WingManX

WingManX(
    gemini_key="your-gemini-api-key",
    airtable_key="your-airtable-pat",
    base_id="appXXXXXXXXXXXXXX"
).run()
```

If this file is ever committed with real values filled in (an easy mistake for a "just plug in your keys and go" sandbox script), those keys end up in git history permanently, recoverable even after a later commit removes them.

**Recommended pattern instead:**

```python
# WingMan X StudioCode.py
import os
from wingmanx import WingManX

WingManX(
    gemini_key=os.environ["WINGMAN_GEMINI_API_KEY"],
    airtable_key=os.environ["WINGMAN_AIRTABLE_PAT"],
    base_id=os.environ["WINGMAN_AIRTABLE_BASE_ID"],
).run()
```

paired with a `.env` file (added to `.gitignore`, never committed) for local development. The Dockerfile in this repository already expects environment-supplied configuration in spirit; this change makes the Python entrypoint consistent with that.

## Running it

```bash
pip install -r requirements.txt
python "WingMan X StudioCode.py"
```

## Requirements the engine needs at runtime

- **Internet access** — required for speech recognition, TTS, Gemini, and Airtable. There is no fully offline mode.
- **Microphone access** — must be granted to the process/terminal.
- Confirm your system has a working audio output device as well, for `pygame.mixer` playback of the TTS response.

## Full setup checklist

- [ ] Gemini API key obtained from Google AI Studio
- [ ] Airtable Personal Access Token and Base ID obtained
- [ ] Credentials moved to environment variables (see corrections above), not left as literals in `StudioCode.py`
- [ ] `pip install -r requirements.txt`
- [ ] Microphone and audio output confirmed working on the host machine
- [ ] `python "WingMan X StudioCode.py"` runs and responds to a test voice command
