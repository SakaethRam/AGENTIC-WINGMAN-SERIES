# Memory and Caching

## `Memory.json`

An auto-generated local file holding:

- Chat history
- Active language setting
- Active mode (developer/user)

Because it's a flat file rather than a database, it persists exactly as long as the file exists on disk: deleting it resets the assistant to a fresh state (no history, presumably default language and mode), which is a useful thing to know both for testing and for a genuine "start over" reset.

### Memory cleanup

Old messages beyond 200 are auto-trimmed. This caps `Memory.json`'s growth and keeps the TF-IDF cache (see below) from being matched against an ever-growing, increasingly stale set of historical queries. A 200-message cap is a reasonable default for a single-user personal assistant; if you extend WingMan X toward heavier daily use, consider whether 200 is enough runway before old context starts dropping off mid-project.

## TF-IDF response caching

Before a new query goes to Gemini, WingMan X checks it against previously-asked queries using a `TfidfVectorizer` and cosine similarity. A sufficiently similar past query returns the cached answer instead of making a new API call.

### Why this matters for a voice assistant specifically

Voice interaction naturally produces more near-duplicate queries than typed chat: a user re-asking "what's the status" a few minutes after the first ask, or rephrasing slightly because the first attempt wasn't recognized correctly by speech-to-text. Caching against semantic similarity (not exact string match) catches these cases and saves the Gemini API call.

### What to watch for

- **Cache staleness** — a cached answer to "what's the status" from ten minutes ago may no longer be accurate if a deliverable was updated in the interim. Confirm in the actual implementation whether cache entries are invalidated when Airtable data changes, or whether status-type queries are excluded from caching specifically for this reason.
- **Similarity threshold** — cosine similarity needs a threshold to decide "close enough to reuse" versus "different enough to ask Gemini again." Too low a threshold risks returning a stale or wrong-context cached answer for what the user intended as a new question; too high a threshold makes the cache rarely fire. This threshold isn't specified in the README and is worth checking directly in `wingmanx.py`.

## Interaction between memory trimming and the cache

Since the TF-IDF cache is built from historical queries, and history is capped at 200 messages, the effective cache "lookback window" is bounded by the same 200-message limit. A query similar to something asked 250 messages ago will not be cache-eligible even though it was asked before, since that entry has already been trimmed from `Memory.json`.
