---
name: elevenlabs-tts
description: Generate speech with ElevenLabs, current model ids (eleven_v3, eleven_multilingual_v2, eleven_flash_v2_5), voice selection, REST and SDK patterns, streaming, and voice settings. Use when the user wants text turned into audio or is coding against the TTS API.
---

# ElevenLabs text to speech

Base URL `https://api.elevenlabs.io`, auth header `xi-api-key` (see the
`elevenlabs-api-key` skill). Through the connected MCP server, just call the
TTS tool; the details below matter for direct API and SDK work.

## Current models (verify at https://elevenlabs.io/docs/overview/models)

| Model id | Use for | Notes |
|---|---|---|
| `eleven_v3` | Most expressive speech, dialogue, audiobooks | 70+ languages; supports audio tags and a Text to Dialogue API; not for real-time |
| `eleven_multilingual_v2` | Polished narration, production quality | 29 languages; best number normalization; higher latency |
| `eleven_flash_v2_5` | Real-time and conversational apps | ~75 ms latency, 32 languages |
| `eleven_flash_v2` | Ultra-low-latency English only | under 75 ms |

`eleven_turbo_v2_5` / `eleven_turbo_v2` still work but Flash is recommended
over Turbo in all cases. Default: `eleven_multilingual_v2` for quality,
`eleven_flash_v2_5` for latency, `eleven_v3` when expressiveness wins.

## Endpoints

- `POST /v1/text-to-speech/{voice_id}`: returns audio bytes.
- `POST /v1/text-to-speech/{voice_id}/stream`: chunked streaming response.
- WebSocket input streaming exists for token-by-token TTS in real-time apps.
- `GET /v1/voices`: list available voices and their `voice_id`s.

Key body fields: `text`, `model_id`, `voice_settings` (`stability`,
`similarity_boost`, `style`, `use_speaker_boost`, `speed`). Output format via
the `output_format` query param (for example `mp3_44100_128`; PCM variants
for telephony).

## SDK patterns

Python (`pip install elevenlabs`):

```python
from elevenlabs.client import ElevenLabs
client = ElevenLabs()  # reads ELEVENLABS_API_KEY
audio = client.text_to_speech.convert(
    voice_id="<voice_id>",
    model_id="eleven_multilingual_v2",
    text="Hello from Caeros.",
    output_format="mp3_44100_128",
)
```

TypeScript: `npm install @elevenlabs/elevenlabs-js`, same shape via
`client.textToSpeech.convert(voiceId, {...})`.

## Voice selection

- Start with the account's default voices (`GET /v1/voices`) or browse the
  Voice Library for thousands of shared voices.
- Instant Voice Cloning needs about a minute of clean audio; Professional
  Voice Cloning needs more data and verification but sounds best.
- Voice Design generates brand-new voices from a text description.
- Keep one `voice_id` per project for consistency; store it in config, not
  prose.

## eleven_v3 specifics

- Inline audio tags steer delivery: `[whispers]`, `[laughs]`, `[sighs]`,
  `[excited]` and similar, placed in the text.
- Output varies between runs; generate a few candidates and pick.
- For multi-speaker scenes use the Text to Dialogue API rather than stitching
  single-voice calls.

## Gotchas

- Every call spends account credits (characters based); long texts cost real
  money, so confirm before bulk narration.
- Chunk very long texts at paragraph boundaries and stitch audio; keep the
  same voice settings across chunks.
- `eleven_v3` in real-time pipelines is a mistake; use Flash there.
