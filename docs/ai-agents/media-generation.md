# 🎬 Media Generation via OpenRouter

OpenRouter is the default not just for LLMs but for **generated media** — one API, one key, across images, video, audio/music, and speech. Route to the best model per job without integrating each provider separately.

## Why route through OpenRouter
- **One API + one key** for text, images, video, audio, embeddings, rerankers — swap models by changing a string.
- **Best-of-breed per modality** — top image, video, and audio models behind a single OpenAI-compatible surface.
- **Same provider layer** you already use for the [agent loop](agent-sdk.md) and [orchestration](orchestration.md) — no new SDK.
- **Async video API** with configurable resolution, aspect ratio, duration, and optional reference images (text-to-video and image-to-video).

## Modalities
| Modality | Use |
|---|---|
| 🖼️ Image | thumbnails, assets, avatars, product shots |
| 🎥 Video | text-to-video, image-to-video clips |
| 🎵 Audio / music | soundtracks, jingles |
| 🗣️ Speech | TTS narration, voiceovers |
| 🔤 Text | the LLM tiers ([orchestration](orchestration.md)) |

## Pattern: a thin media SDK
Wrap media calls the same way you wrap LLM calls — one client, model id per modality, provider behind it (an "OpenRouter for media" pattern: a single TS SDK routing to many models across providers):
```ts
// packages/media/src/client.ts  — SRP: routing only
export async function generateImage(prompt: string, opts?: { model?: string }) {
  return mediaClient.create({ modality: "image", model: opts?.model ?? IMAGE_MODEL, prompt });
}
export async function generateVideo(prompt: string, opts?: { durationSec?: number; aspect?: string; ref?: string }) {
  // video is async: kick off, then poll the job to completion
  const job = await mediaClient.createVideo({ model: VIDEO_MODEL, prompt, ...opts });
  return pollUntilDone(job.id);
}
```
- **Store outputs in [R2](../stack/databases.md)** (S3-compatible, no egress) and keep only the key/URL in Postgres.
- **Video is asynchronous** — model it as a [BullMQ job](../stack/backend-bun-hono.md) (enqueue → poll → persist → notify), not a blocking request.
- **Keep keys in [sealed secrets / Vaultwarden](../infrastructure/secrets.md)**.

## Real-world example
A free, browser-based video editor — [KubeezCut / kubeez.com](https://editor.kubeez.com/) — does first-class **image, video, music, and speech generation** through the Kubeez API, then lets you trim, layer, and finish the cut on a WebGPU, local-first timeline. It's a good model for "generate media via a unified API, then edit in the browser." See also the walkthrough: [Generate AI Images, Videos, Music & Speech with JavaScript](https://dev.to/sebyx07/how-to-generate-ai-images-videos-music-speech-programmatically-with-javascript-ggj).

## Rules
- **Route media through one provider layer** (OpenRouter) — don't hand-integrate each model vendor.
- **Async for video/long audio** — job + poll, never block a request.
- **Persist to object storage**, reference by URL; never inline blobs in the DB.
- **Same cost tracking** as LLM calls ([orchestration](orchestration.md#cost--safety)).

Sources: [OpenRouter video generation](https://openrouter.ai/docs/guides/overview/multimodal/video-generation) · [OpenRouter multimodal overview](https://openrouter.ai/docs/guides/overview/multimodal/overview) · [kubeez.com](https://editor.kubeez.com/)
