# Voice Commands and Modes

## Command reference

| Action | Say... | Notes |
|--------|--------|-------|
| Switch mode | "mode", or "developer" / "user" directly | Toggles debug logs and command hints on/off. |
| Change language | "language" | Switches between English and Hindi. |
| Add deliverable | "add update: Launch MVP" | Creates a new CRM entry via the Airtable integration. |
| Update last task | "update last: delayed to Friday" | Modifies the most recently added deliverable rather than requiring a lookup by name. |
| Get all updates | "update" or "status" | Fetches current deliverables from Airtable. |
| Show chat history | "show history" | Reads back from `Memory.json`. |
| Search past chat | "search: MVP" | Searches chat history for a keyword/phrase. |
| Export chat | "export chat" | Writes the conversation log out to a file. |
| Exit | "bye" or "exit" | Ends the session. |

## Modes

### User mode (default)

A clean console: spoken responses are the primary interface, with minimal text output cluttering the terminal.

### Developer mode

Surfaces debug logs and command hints directly in the console alongside the voice interaction, useful when diagnosing why a command wasn't recognized or an Airtable call failed, without needing to attach a separate debugger.

Switching modes is a runtime voice command ("mode", "developer", "user"), not a restart-required configuration flag, so you can flip into developer mode mid-session to diagnose an issue and flip back without losing the current conversation state.

## Multilingual support

English and Hindi, with real-time Devanagari conversion for Hindi output. The "language" voice command switches between the two. Hindi text-to-speech uses `gTTS` with `tld='co.in'` specifically, which is what gives the Hindi voice a natural Indian-accent rendering rather than a generic one.

## "add update" vs. "update last": why both exist

`add update` always creates a new deliverable entry; `update last` always targets the most recently created one. Keeping these as two distinct commands, rather than one smart command that guesses intent, avoids the ambiguity of a voice system trying to disambiguate "update the MVP task" when there might be several similarly-named tasks in Airtable. The trade-off is that updating anything other than the most recent task by voice isn't directly supported by the documented command set; that would need to go through the Airtable interface directly, or a future command extension.
