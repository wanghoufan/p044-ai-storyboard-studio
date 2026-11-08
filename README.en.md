# AI Storyboard Studio (AI 图文短剧分镜生成器)

> One idea in, five text shots and five landscape frames out — a storyboard laid out before short-video production starts.

[简体中文](README.md) | English

<p>
  <img src="docs/screenshot-home.png" width="220" alt="Home empty state: short-drama theme, script and style preferences">
  <img src="docs/qa/implementation-storyboard-landscape-1440x1024.png" width="760" alt="Storyboard in progress: creative input on the left, text shots and 16:9 images on the right">
</p>

<sub>UI captured from the running online version. The home image is the narrow-screen empty state; the storyboard image is desktop width.</sub>

## What problem it solves

Before shooting a short video you need to know what each shot shows, what the characters do, what is said, and what the frame should look like. Starting straight from a script often leads to shots that do not cut together.

This tool takes a theme and a script to the model: first it produces 5 structured text shots (shot type, visual description, character action, dialogue or narration, image prompt), then it generates one 16:9 landscape image from each shot's prompt. The result is a storyboard you can take to shooting or video generation.

Live site: <https://ai-storyboard-studio-zeta.vercel.app>

## Core capabilities

- **Streaming text shots**: the server calls Bailian `qwen3.7-plus` and shows the 5 shots one by one as they stream in, without waiting for the whole run.
- **Five landscape images**: it then calls `qwen-image-2.0-pro-2026-06-22` to generate one image per shot, default size `2688*1536` (16:9).
- **Structured shot fields**: each shot carries `shotNumber`, `visualDescription`, `characterAction`, `dialogueOrNarration`, `shotType` and `visualPrompt`.
- **Creative input**: theme (≤ 80 characters), script (≤ 20,000 characters) and style preference (≤ 400 characters), plus preset style tags for quick selection.
- **Bring your own key (BYOK)**: paste your own Beijing-region Bailian API key; it is stored in browser `localStorage` and sent with each request through the `x-dashscope-api-key` header, and can be cleared at any time.
- **Export**: copy all storyboard text; download a single image or all images. Downloads use a server-issued token valid for 24 hours.
- **Rate limiting and stopping**: images are queued one at a time within Bailian's 2 RPM limit; generation can be stopped at any point and completed content is kept.

## Quick start

Online: open <https://ai-storyboard-studio-zeta.vercel.app>, paste your key into the Bailian API key field and save it, fill in the theme and script, then generate. Costs are billed directly to your own Bailian account.

Local development:

```bash
npm install
npm run dev
```

Open the local address printed in the terminal (the Vite dev server also serves `/api/storyboards`, so no separate backend is needed).

Locally you can also fall back to environment variables, useful for debugging without a browser key:

```bash
cp .env.example .env
```

Set `DASHSCOPE_API_KEY` in `.env`. Optional variables:

- `DASHSCOPE_BASE_URL`: defaults to the shared Beijing-region endpoint, `https://dashscope.aliyuncs.com/compatible-mode/v1`.
- `DASHSCOPE_MODEL`: defaults to `qwen3.7-plus`.
- `DASHSCOPE_IMAGE_ENDPOINT`, `DASHSCOPE_IMAGE_MODEL`, `DASHSCOPE_IMAGE_SIZE`: image endpoint, model and size; default `2688*1536`.

Build and test:

```bash
npm run build
npm test
```

`npm test` runs 17 tests covering input validation, streaming NDJSON parsing, image prompts and native API parameters, download signing, tamper checks, copy formatting, the BYOK request-header path (including an assertion that the key is never echoed back) and local key storage.

## Data and privacy

- The API key lives only in the current browser's `localStorage`; the server uses it only within that request, never writes it to disk, never logs it and never echoes it back.
- The site holds no user keys and does not pay on the user's behalf; generation costs go directly to the user's own Bailian account.
- There is no login, quota or cost confirmation, and generated results are not stored on the server; after a refresh or closing the page you need to generate again.
- `.env` is listed in `.gitignore` and should not be committed; deployment environment variables are not shipped with the source or build output.
- Download tokens are signed by the server with HMAC-SHA256 and expire after 24 hours; image files themselves come from temporary Bailian links.

## Known limitations

- The image model is billed per image; one run generates 5 by default. Under Bailian's 2 RPM limit images are queued one at a time, so a full run takes a while.
- Bailian image links expire after roughly 24 hours; download them within the session in which they were generated.
- There is no in-site login, usage quota or cost confirmation; anyone with the link and their own key can use generation, and the cost is theirs.
- The image download endpoint validates download tokens with the server-configured `DASHSCOPE_API_KEY`. For a deployment that relies only on browser BYOK and does not set that variable, whether downloads work is not verified in this repository.
- `worker/`, `scripts/prepare-sites-build.mjs` and `.openai/hosting.json` are leftovers from the old ChatGPT Sites channel and are not part of the default build; verify them separately with `npm run build:sites && npm run test:sites` when needed.
- The UI and copy are Simplified Chinese only.
