# Docker

## The existing Dockerfile

The repository already ships a working `DockerFile` at the root. It's a solid, minimal setup:

- Base image: `python:3.11-slim`.
- System dependencies for audio: `alsa-utils`, `portaudio19-dev`, `libffi-dev` (for `PyAudio`/`speech_recognition`), plus `gcc` and `build-essential` for compiling any wheels that need it, and `pulseaudio`.
- Standard layer-caching pattern: `requirements.txt` copied and installed before the application code, so a code change doesn't force a full dependency reinstall.
- Runs as a non-root `appuser`, not root, which is good container hygiene.
- No ports exposed, correctly, since this is a voice-only CLI app with no HTTP surface.
- Default command runs the test runner (`WingMan X StudioCode.py`); overridable via `docker run ... python "WingMan X StudioCode.py"` (which is already the default, so overriding is only needed if you point it at a different entrypoint).

Nothing here needs replacing. This document exists to explain what it does and, more importantly, the real limitation that comes with containerizing a microphone-and-speaker application.

## The limitation worth understanding: microphone and speaker access in a container

A container is isolated from the host's audio devices by default. Installing ALSA/PortAudio inside the image gives the Python libraries something to talk to, but it does not by itself give the container access to your actual microphone or speakers. To make voice I/O work when running WingMan X in Docker, you need to explicitly pass audio devices through:

**On Linux**, passing through the ALSA device typically works:

```bash
docker run -it --rm \
  --device /dev/snd \
  -e WINGMAN_GEMINI_API_KEY \
  -e WINGMAN_AIRTABLE_PAT \
  -e WINGMAN_AIRTABLE_BASE_ID \
  wingman-x
```

**On macOS and Windows**, Docker Desktop runs containers inside a Linux VM with no direct path to the host's audio hardware. `--device /dev/snd` will not work the way it does on native Linux. Practical options if you're on macOS/Windows:

- Run WingMan X natively (`python "WingMan X StudioCode.py"` on the host) for actual voice interaction, and reserve the Docker image for environments where audio passthrough is solvable (a Linux server, a Linux dev container) or for testing non-audio code paths.
- If containerized voice I/O is a hard requirement on macOS/Windows, you'd need a host-side audio bridge (e.g. PulseAudio networked to the container) — meaningfully more setup than this project's current scope, and worth confirming is actually necessary before investing in it.

This isn't a flaw in the existing Dockerfile; it's an inherent property of containerizing hardware-adjacent I/O, and worth documenting explicitly so it doesn't look like a broken image when a `docker run` on a laptop produces no captured audio.

## Local orchestration (`docker-compose.yml`)

For the Linux case, a compose file wraps the same device passthrough and env var wiring:

```yaml
version: "3.9"

services:
  wingman-x:
    build: .
    image: wingman-x
    devices:
      - "/dev/snd:/dev/snd"
    environment:
      - WINGMAN_GEMINI_API_KEY=${WINGMAN_GEMINI_API_KEY:-}
      - WINGMAN_AIRTABLE_PAT=${WINGMAN_AIRTABLE_PAT:-}
      - WINGMAN_AIRTABLE_BASE_ID=${WINGMAN_AIRTABLE_BASE_ID:-}
    stdin_open: true
    tty: true
```

`stdin_open` and `tty` are set because this is an interactive CLI application, not a background service; without them, `docker compose up` would run the container without a usable terminal for voice-session interaction.
