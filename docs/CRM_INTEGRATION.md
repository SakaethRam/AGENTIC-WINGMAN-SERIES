# CRM Integration (Airtable)

WingMan X treats Airtable as its system of record for deliverables/tasks, reachable entirely through voice commands rather than requiring the user to open Airtable directly.

## What's synced

- **Add** — a new deliverable entry, via "add update: <description>".
- **Update** — the most recently added deliverable, via "update last: <change>".
- **Fetch** — current deliverables, via "update" or "status".
- **Auto-refresh** — the README lists periodic refresh of the latest tasks as a feature, meaning the engine polls Airtable on some interval rather than only fetching on an explicit voice command.

## Configuration

Two Airtable credentials are required:

1. **Personal Access Token** — from Airtable → Account → Personal Access Tokens.
2. **Base ID** — found in your base's URL, in the form `appXXXXXXXXXXXXXX`.

Both are passed into the `WingManX` constructor alongside the Gemini key:

```python
WingManX(
    gemini_key="your-gemini-api-key",
    airtable_key="your-airtable-pat",
    base_id="appXXXXXXXXXXXXXX"
).run()
```

See `SETUP_AND_CONFIGURATION.md` for why passing these as literal constructor arguments in a committed file is a practice worth changing before this goes anywhere beyond a personal sandbox.

## Data shape to confirm before integrating further

The README doesn't document the exact Airtable table/field schema WingMan X expects (table name, field names for a deliverable's description/status/due date). Before building anything on top of this integration (a dashboard, a second client), inspect `wingmanx.py`'s Airtable API calls directly to confirm the actual field names it reads and writes, rather than assuming a schema from the voice command wording alone.

## Failure handling

Airtable calls depend on network connectivity, same as Gemini and speech recognition. Per the README's "Offline Graceful Degradation" feature, a failed Airtable call should degrade rather than crash the session, but the specific behavior (retry, skip with a spoken error, queue for later) should be confirmed in source rather than assumed.
