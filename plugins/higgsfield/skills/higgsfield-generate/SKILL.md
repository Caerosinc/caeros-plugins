---
name: higgsfield-generate
description: Generate images and videos through the Higgsfield MCP server or Cloud API, including model and preset selection, Soul characters, and image-to-video workflows. Use when the user wants to create or animate visual media with Higgsfield.
---

# Higgsfield generation

The connected MCP server (`https://mcp.higgsfield.ai/mcp`) exposes generation
tools directly; ask it to list tools if unsure which are available in the
current session. All generation is asynchronous: submit, then poll status
until the job completes. Images usually finish in seconds; videos take longer.

## Capabilities and limits

- 30+ models behind one connector. The server can auto-select a model from
  your prompt, or you can name one explicitly.
- Images up to 4K resolution; videos up to 15 seconds.
- Every generation costs credits from the user's Higgsfield plan, priced by
  model and resolution. Warn before large batches or 4K/video runs.

## Model selection (verify with the server's model list when precision matters)

| Need | Reach for |
|---|---|
| Photorealistic people, consistent characters | Soul (supports character training) |
| Cinematic video, camera-controlled shots | Cinema Studio |
| General text-to-image | Flux, Seedream |
| Text/image-to-video | Kling, Minimax Hailuo, Veo |

Auto-select is a good default; pin a model when the user names one or when
consistency across a series matters.

## Common workflows

- **Text to image**: one prompt, optionally resolution/aspect. Prompt like a
  photographer: subject, lens or framing, lighting, mood, style references.
- **Image to video**: pass a source image plus a motion prompt. Describe the
  camera move explicitly (dolly-in, orbit, crash zoom, handheld) and keep
  motion to one or two beats for a 5 to 15 second clip.
- **Consistent characters (Soul)**: create/train a character once, then
  reference it across generations for the same face and identity. List
  existing characters before training a new one.
- **Image utilities**: background removal, upscaling, and image expansion are
  available server-side; prefer these over regenerating from scratch.
- **Marketing videos**: the server can build product marketing videos from a
  product URL; give brand tone and target duration.

Always poll generation status rather than assuming completion, and surface
the returned asset URLs to the user promptly.

## Direct Cloud API (when the user is coding an integration)

Higgsfield also has a developer platform at https://cloud.higgsfield.ai with
an official Python SDK, `higgsfield-client`
(https://github.com/higgsfield-ai/higgsfield-client):

- Auth via API credentials from Higgsfield Cloud set as environment variables
  (see the `higgsfield-api-key` skill).
- Sync and async clients; `submit` returns a job you poll, `subscribe` blocks
  until done.
- Job statuses: `Queued`, `InProgress`, `Completed`, `Failed`, `NSFW`,
  `Cancelled`. Handle `NSFW` as a content rejection, not an error to retry.
- Optional webhook callbacks on completion, useful for long video jobs.

Docs index: https://docs.higgsfield.ai (fetch when exact request shapes or
current model ids matter; the catalog changes often).
