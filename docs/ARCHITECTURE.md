# Architecture

WingMan X is a single-process, voice-first assistant: one Python engine (`wingmanx.py`) that chains speech recognition, an LLM, a CRM integration, and text-to-speech into one loop, with a local JSON file as its only persistence layer.

## Component map

| Concern | Technology | Role |
|---------|-----------|------|
| Speech-to-text | `speech_recognition` + Google Web API | Converts spoken input into text for the engine to process. |
| Reasoning / generation | Google Gemini (`gemini-2.0-flash`) | Produces responses to conversational input. |
| Response caching | `scikit-learn` `TfidfVectorizer` + cosine similarity | Matches new queries against previously-answered ones to avoid redundant Gemini calls. |
| Text-to-speech | `gTTS` | Converts the engine's text response into speech. |
| Audio playback | `pygame.mixer` | Plays the synthesized speech back to the user. |
| CRM sync | Airtable REST API | Adds, updates, and fetches deliverables by voice command. |
| Translation | `googletrans` | Supports the English/Hindi multilingual mode, including Devanagari conversion. |
| Persistence | `Memory.json` (local file) | Chat history, active language, and mode, across sessions. |

## Request loop

```
Microphone input
       │  speech_recognition (Google Web API)
       ▼
   Text query
       │
       ├── TF-IDF cache check ──▶ cached answer found? ──▶ skip Gemini call
       │
       ▼ (cache miss)
   Gemini (gemini-2.0-flash)
       │
       ▼
   Response text
       │
       ├── Airtable CRM call, if the query was a deliverable command
       │
       ▼
   gTTS → pygame.mixer
       │
       ▼
   Spoken response
```

## Why TF-IDF caching sits before the LLM call, not after

Because Gemini calls cost money and latency per request, checking cosine similarity against previously-asked queries first means a repeated or near-duplicate question (e.g. re-asking "what's the status" a few minutes apart) can be answered from cache without a new API round-trip. This is a meaningful design choice for a voice assistant specifically: voice interactions tend to include more repetition and re-confirmation than typed chat, so the caching layer earns its keep more here than it would in a typical text chatbot.

## Why persistence is a flat JSON file, not a database

`Memory.json` holds chat history, language setting, and mode, for a single-user, single-process assistant. There's no multi-user concern and no concurrent-write scenario to guard against, so a database would add operational overhead (schema, migrations, a running service) without buying anything a flat file with periodic trimming doesn't already provide. The trade-off: this design does not scale to multiple simultaneous users or processes sharing state, which is worth knowing before extending WingMan X into anything beyond a single personal assistant.

## Developer vs. user mode

The engine exposes a runtime toggle (voice command: "developer" / "user") that changes what's surfaced to the console: debug logs and command hints in developer mode, a cleaner console in user mode. This is an operational parameter, not a separate build; the underlying engine logic is identical in both modes. See `VOICE_COMMANDS_AND_MODES.md` for the full command list.

## Offline degradation

Because speech recognition, Gemini, and Airtable all require connectivity, the engine is designed to degrade gracefully rather than crash when internet access is unavailable, per the README's stated behavior. Confirm the specific fallback behavior (does it queue commands, print an error, or silently skip the affected feature) against the actual `wingmanx.py` implementation before relying on it as documented in detail.
