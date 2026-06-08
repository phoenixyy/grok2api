# Changelog

All notable changes to this fork (`phoenixyy/grok2api`) are documented here.
Upstream changes from `chenyme/grok2api` are not listed; see the upstream repo for those.

Format loosely follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [1.0.1] - 2026-06-09

### Changed

**`POST /v1/images/generations` — new `moderated_count` field**

When one or more images in a batch are blocked by xAI's server-side content
moderation, the response now includes a `moderated_count` field indicating how
many slots were silently dropped.

**Non-streaming response** (`stream: false`, default):

```json
{
  "created": 1749999999,
  "data": [
    { "url": "http://localhost:8000/v1/files/image?id=abc123" }
  ],
  "moderated_count": 1
}
```

`moderated_count` is omitted entirely when no images were moderated (i.e. zero
is never written, absence means zero).

**Streaming response** (`stream: true`):

An extra SSE frame is injected between the final `[DONE]` delta and the
`data: [DONE]` terminator when at least one image was moderated:

```
data: {"moderated_count": 1}

data: [DONE]
```

**Behaviour notes:**
- This field only appears on the WebSocket-backed models (`grok-imagine-image`,
  `grok-imagine-image-pro`). The lite chat-based model (`grok-imagine-image-lite`)
  does not go through the WebSocket path and is unaffected.
- `moderated_count` reflects server-side hard blocks (`moderated: true` in the
  upstream WS frame). Images marked `r_rated: true` are **not** counted here —
  those are returned normally when `features.enable_nsfw` is enabled.
- If all requested images are moderated, `data` will be an empty array and
  `moderated_count` will equal the originally requested `n`.

---

## [1.0.0] - 2026-06-08

### Added

- Initial fork from `chenyme/grok2api` at commit `5805cbb`
- CI: upgraded GitHub Actions to Node.js 24-compatible versions
  (`actions/checkout@v6.0.3`, `docker/setup-qemu-action@v4.1.0`,
  `docker/setup-buildx-action@v4.1.0`, `docker/login-action@v4.2.0`,
  `docker/metadata-action@v6.1.0`, `docker/build-push-action@v7.2.0`)
- Docker image published to `ghcr.io/phoenixyy/grok2api`
