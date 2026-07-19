---
name: elevenlabs-agents
description: Build real-time conversational voice agents on the ElevenLabs Agents platform, agent configuration, WebSocket and WebRTC connections, SDKs, telephony, and pricing. Use when the user wants a voice or chat agent powered by ElevenLabs.
---

# ElevenLabs Agents

The Agents platform (docs: https://elevenlabs.io/docs/eleven-agents/overview)
is a managed stack for real-time voice and chat agents: you configure an
agent (LLM, voice, prompt, tools, knowledge base) and connect clients over
WebSocket or WebRTC, or route phone calls to it.

## Creating an agent

- Dashboard: create and iterate visually, including a workflow builder.
- CLI: `npm install -g @elevenlabs/cli`, then scaffold from templates
  (`complete`, `minimal`, `voice-only`, `text-only`, `customer-service`,
  `assistant`). Recommended for versioned, reproducible agent config.
- Core config: system prompt, LLM choice, voice (`voice_id`, use a Flash
  model for latency), first message, tools (server tools call your HTTP
  endpoints; client tools run in the user's app), knowledge base documents
  for RAG, and evaluation criteria.

Each agent has an `agent_id`; that is the only handle clients need.

## Connecting clients

- WebSocket:
  `wss://api.elevenlabs.io/v1/convai/conversation?agent_id={agent_id}`.
  Public agents connect with just the id. Private agents require a signed
  URL that your backend fetches from the ElevenLabs API with the API key;
  signed URLs expire after 15 minutes (the conversation may run longer, it
  just has to start within the window).
- WebRTC: best echo cancellation and noise handling in browsers, no
  plugins. In current SDKs the connection type is inferred: voice
  conversations use WebRTC, text-only use WebSocket; you can still force
  `connectionType`.
- Restrict public agents with domain allowlists so only approved origins
  can connect.

## SDKs

`@elevenlabs/client` (JS), `@elevenlabs/react` (re-exports the client
package, so install only the React one for React apps), Python, and iOS.
Typical React flow: `useConversation()` then
`conversation.startSession({ agentId })`. Listen for the
`agent_response_complete` event to reliably detect end of turn after all
tool calls and audio playback finish.

## Telephony and channels

Phone via Twilio-style carrier integration, plus WhatsApp and embedded web
chat widgets. For phone audio, pick PCM/mulaw-friendly output formats and a
low-latency voice model.

## Pricing shape (verify at https://elevenlabs.io/pricing/agents)

- Billed by call minutes with per-plan concurrency limits (Free 15 min and
  4 concurrent; paid tiers scale both).
- Overage around $0.08 per extra minute; burst mode allows up to 3x
  concurrency at double the per-minute rate.
- LLM and telephony costs are billed separately from platform minutes.
- No cap on number of agents; only minutes and concurrency are limited.

## Gotchas

- Never ship the workspace API key to browsers or mobile apps; only the
  signed-URL fetch belongs on your server.
- Test tool latency: slow server tools stall the conversation, so keep tool
  endpoints fast and idempotent.
- Text-only agents are much cheaper than voice; prototype logic in text
  before turning on audio.
