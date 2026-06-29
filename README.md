# Church Program Auto-Player — controls (Prev · ⏪ · Play/Pause · ⏩ · Next)

A no-install, full-screen web player that streams chosen sections of a recorded service video — in your order — with the church logo animated between each section, per-section playback speed, **in-section rewind / fast-forward (±10s)**, **buffer-ahead playback** so it doesn't lag mid-clip, and the YouTube interface hidden. Built for **Deeper Life Bible Church** auditorium playback.

> **v3 adds:** ⏪ / ⏩ buttons (and `J` / `L` keys) that jump ±10s **within the current section**, plus buffer-ahead so fast-forwarding lands on already-loaded video (the "you're at 2:00 but it's loaded to 10:00" feel). See [Seeking & buffer-ahead](#seeking--buffer-ahead-no-lag).

**v2 adds a builder page:** instead of fixed GHS/Choir/GS fields, you **add any number of named sections** (Name + Start + End + Speed) with an **Add** button, reorder/remove them, then launch the player.

---

## Why this vs. just playing on YouTube

YouTube is still the host — this app is the **presentation layer** that turns a raw recording into a clean, automated, branded service playback:

- **Only your sections, in your order** — auto-jumps to each part (even out of sequence) and plays start→end. No live scrubbing to timestamps in front of the congregation.
- **No YouTube clutter** — hides the title, channel, controls, "Watch on YouTube", and the end-screen grid of recommended videos. Just clean full-screen video.
- **Church-branded** — animated logo between every section and at the end, instead of a wall of suggested clips.
- **Hands-free & repeatable** — one **Start** click runs the whole program; same result every week.
- **Per-section speed** — speed up a long message (1.25×/1.5×) without touching the songs.
- **Ad-free** with your own non-monetized upload (keep ads off; upload as *Unlisted*) — no surprise ad mid-service.
- **No install, low cost** — runs in Chrome on the AV PC; hosted on GitHub Pages + n8n.

> It rides on YouTube's free, reliable hosting but removes everything that makes it look like YouTube — and automates the service so the operator just presses **Start**.

---

## How it works

```
HOST (each week)                         AUDITORIUM (service)
setup.html (builder)  ──build link──►  index.html (player)
add sections + Launch                  full-screen, auto-advancing, animated logo
```

- **Builder:** https://mivics1.github.io/church-player-controls/setup.html
- **Player:** https://mivics1.github.io/church-player-controls/
- The builder assembles a player link from your inputs; the player reads everything from the URL — nothing is stored server-side.

---

## Using it (weekly, ~2 min)

1. On the auditorium PC in **Chrome**, open the **builder**: `…/church-player-controls/setup.html`.
2. Paste the **video link** (a *recording*, not a live stream). Leave **Logo URL** blank to use the church logo.
3. For each part, type **Name**, **Start (mm:ss)**, **End (mm:ss)**, choose a **Speed**, and click **➕ Add**. Repeat for every section.
4. Reorder with **↑ / ↓**, remove with **✕**.
5. Click **▶ Launch Player** → the player opens. Press **▶ Start Program** to go full-screen and play.

Sections play in order: section → logo (in/out) → next section → … → ends on the logo.

### Operator keys
| Key | Action |
|-----|--------|
| `Space` | Pause / resume |
| `→` / `←` | Next / previous **section** |
| `L` / `J` | Forward / rewind **10s** within the current section |
| `Esc` | Exit full-screen |

On-screen controls (appear on mouse move): **⏮ Prev · ⏪ −10s · ▶/⏸ · ⏩ +10s · ⏭ Next**.

---

## Player URL parameters

The builder produces these automatically, but you can also open the player directly:

```
https://mivics1.github.io/church-player-controls/?video=<URL>&logo=<URL>&logoSec=3&reveal=3.5&sections=<url-encoded JSON>
```

| Param | Meaning |
|-------|---------|
| `video` | YouTube **or** direct/HLS recording URL |
| `sections` | **Primary.** URL-encoded JSON array; each item `{ "name", "start", "end", "rate" }` with **start/end in seconds**, `rate` = 1 / 1.25 / 1.5 |
| `logo` | Logo image/animation URL (optional; defaults to bundled `logo.png`) |
| `logoSec` | Seconds to hold the logo between sections (default 3) |
| `reveal` | Seconds the logo stays after a clip starts, hiding YouTube's start-of-clip UI (default 3.5) |
| `skip` | Seconds each ⏪ / ⏩ (and `J` / `L`) jumps within the current section (default 10) |

Example `sections` (before URL-encoding):
```json
[{"name":"GHS","start":600,"end":660,"rate":1},
 {"name":"Choir Song","start":90,"end":105,"rate":1},
 {"name":"GS Message","start":300,"end":360,"rate":1.25}]
```

> **Backward compatible:** old links using `ghs=<start>-<end>&choir=…&gs=…` (+ `gsRate`) still work — the player falls back to them when `sections` is absent.

---

## Seeking & buffer-ahead (no lag)

- **Rewind / Fast-forward:** **⏪ −10s** and **⏩ +10s** buttons (keys **`J`** / **`L`**) move the playhead **within the current section** — they clamp to that section's start/end, so you can't slip into another part, and the program still auto-advances when you reach a section's end. The **⏮ / ⏭** buttons (and `←` / `→`) still jump between whole sections. Change the jump size with the builder's **Skip step** field or the `skip` URL param.
- **Buffer-ahead:** the player loads the stream well ahead of the playhead so fast-forwarding lands on already-buffered video:
  - **HLS (`.m3u8`)** — buffers minutes ahead (tuned hls.js settings: large `maxBufferLength` / `maxMaxBufferLength`). Strongest effect — this is the real "loaded to 10:00" behavior.
  - **Direct MP4/WebM** — `preload="auto"`; the browser buffers ahead on its own (CDNs that honor range requests help).
  - **YouTube** — YouTube controls its own buffer depth and it **can't be forced** via the IFrame API. The player masks transition buffering with the animated logo and **pre-seeks the next section behind the logo** during the gap, so on a normal connection transitions feel seamless. For full control of how far ahead it loads, use a direct `.mp4`/`.m3u8` source.

---

## The logo

- The player shows **`logo.png`** from this repo by default, fading/zooming **in and out** between sections and holding on it at the end.
- Until a `logo.png` exists, a clean "DEEPER LIFE BIBLE CHURCH" text card shows instead.
- A short animation (`.mp4`/`.webm`) also works — pass it as the `logo` parameter (or as the builder's Logo URL); it loops while shown.

### Replace the logo (drag-and-drop, ~30 sec)
1. Open this repo on GitHub → **Add file ▸ Upload files**.
2. Drag your logo, **rename it `logo.png`**, **Commit**. It appears within ~1 min.

---

## Important constraints

- ⚠️ **Recording only.** You can't seek to fixed times on a *live* stream — use an on-demand recording.
- ⚠️ **Ad-free / no YouTube tells** is best with **your own non-monetized** YouTube upload (set **Unlisted**) or a **direct/HLS** link. YouTube branding can't be 100% removed by ToS, but controls/title/play-pause are hidden in normal playback; a slow connection can still flash a buffering spinner mid-clip.
- ⚠️ Use **Chrome**; YouTube seeking is accurate to ~±1s (direct video is exact).

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| **▶ Launch Player** is disabled | Add at least one section **and** enter a video link |
| A section is skipped / won't add | **End time must be after start** for that section |
| Black screen after Start | Video isn't embeddable → use your own Unlisted upload or a direct `.mp4`/`.m3u8` link |
| No sound | Click **Start** (sound needs the click); raise PC volume |
| Logo shows as a text card | Add your `logo.png` to this repo |
| Ads appear | Video is monetized/third-party → use your own non-monetized upload |

---

## Files

- `setup.html` — the **builder** (add/reorder sections → Launch).
- `index.html` — the **player** (HTML + CSS + JS, no build step).
- `logo.png` — the church logo.

> A v2 n8n form variant exists, but the **builder page is the v2 entry point** (n8n forms can't add dynamic rows).

Built with the [YouTube IFrame Player API](https://developers.google.com/youtube/iframe_api_reference) and [hls.js](https://github.com/video-dev/hls.js) for direct/HLS streams.
