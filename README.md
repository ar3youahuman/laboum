# laboum player

A tiny, controllable video-embed shim hosted on GitHub Pages.

**Why it exists:** sandboxed iframes (Claude artifacts, `sandbox=""` frames, etc.)
often can't load a streaming host directly. But they *can* load a normal HTTPS page.
So this page sits in the middle: the sandbox embeds **this**, and **this** embeds the
movie — and you can still drive it from the sandbox.

Live: https://ar3youahuman.github.io/laboum/

## Three ways to control it

### 1. URL query params (simplest — works under the tightest sandbox)
Just point the iframe `src` at the page with params:

```
https://ar3youahuman.github.io/laboum/?p=ok&v=7241510750968
https://ar3youahuman.github.io/laboum/?p=youtube&v=dQw4w9WgXcQ
https://ar3youahuman.github.io/laboum/?p=vidsrc-movie&v=27205          # TMDB/IMDB id
https://ar3youahuman.github.io/laboum/?p=vidsrc-tv&v=1399&s=1&e=1
https://ar3youahuman.github.io/laboum/?src=https://ok.ru/videoembed/7241510750968
https://ar3youahuman.github.io/laboum/?p=ok&v=7241510750968&ui=0       # hide controls
```

| param | meaning |
|-------|---------|
| `p` / `provider` | `ok`, `youtube`, `vidsrc-movie`, `vidsrc-tv`, `url` |
| `v` / `id` | video id (or TMDB/IMDB id, or full link — ids are auto-extracted) |
| `s`, `e` | season / episode (for `vidsrc-tv`) |
| `src` / `url` | a full embed URL, used as-is |
| `ui=0` | hide the control bar |

### 2. postMessage (swap movies live, no reload)
From the parent (the sandbox) once the frame is loaded:

```js
frame.contentWindow.postMessage(
  { type: "laboum", cmd: "load", provider: "ok", id: "7241510750968" }, "*");

frame.contentWindow.postMessage(
  { type: "laboum", cmd: "load", url: "https://www.youtube.com/embed/dQw4w9WgXcQ" }, "*");

frame.contentWindow.postMessage({ type: "laboum", cmd: "fullscreen" }, "*");
frame.contentWindow.postMessage({ type: "laboum", cmd: "ui", show: false }, "*");
```

The page posts back `{type:"laboum", event:"ready"}` on load and
`{type:"laboum", event:"loaded", src}` whenever a video starts.

### 3. The built-in control bar
Pick a source, paste an id/URL, hit Play — works even when the page itself is
embedded inside the sandbox.

## Embedding inside a sandbox iframe

```html
<iframe
  src="https://ar3youahuman.github.io/laboum/?p=ok&v=7241510750968"
  sandbox="allow-scripts allow-same-origin allow-presentation allow-forms allow-popups"
  allow="autoplay; fullscreen; encrypted-media; picture-in-picture"
  allowfullscreen
  style="width:100%;height:480px;border:0">
</iframe>
```

> Note: `allow-same-origin` + `allow-scripts` together let the inner page run
> normally. Drop `allow-same-origin` for a stricter sandbox — the query-param
> mode still works, but `postMessage` from the parent may be limited.
