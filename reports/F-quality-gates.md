# F — Quality Gates: What Separates a Winning Site From a Pretty Demo

Research date: 2026-09-15. Environment assumed: Windows 11 + WSL2, headless Chrome at
`/mnt/c/Program Files/Google/Chrome/Application/chrome.exe`, no Linux Node, Windows Node v24 /
npm 11 at `/mnt/c/Program Files/nodejs/`, python3 in WSL. **All Chrome/CDP claims below and the
Python recipe in §2 were live-tested against real Chrome 152 headless=new, launched from WSL2
bash, on this exact machine** — not just documentation summaries. Where something is a live,
verified finding rather than a documentation quote, it's marked **[verified live]**.

---

## 0. TL;DR — the tension this whole doc is about

Awwwards judges **Design 40% / Usability 30% / Creativity 20% / Content 10%** — there is no
explicit "performance" line item. A WebGL-heavy site can and regularly does win Site of the Day
with an LCP north of 4 seconds, because the jury looks at it on a fast desktop over a good
connection, not a throttled mid-range Android on 4G. **Winning an award and shipping a page that
converts/ranks are two different gates.** Google's ranking systems and real mobile users do not
grade on a curve: Core Web Vitals are measured in the field at the 75th percentile, mobile is
harder to pass than desktop, and a beautiful site that stutters on interaction loses users
regardless of trophies. This doc gives you both gates — the aesthetic/awwwards-style bar and the
hard, machine-checkable one — and the tooling to verify the second one from this exact machine.

---

## 1. Core Web Vitals as of 2026

### 1.1 The current metric set and exact thresholds

Three Core Web Vitals, all evaluated at the **75th percentile of page loads**, segmented mobile
vs. desktop (same thresholds for both — mobile is just harder to hit in practice):

| Metric | Good | Needs Improvement | Poor | What it measures |
|---|---|---|---|---|
| **LCP** (Largest Contentful Paint) | ≤ 2.5 s | 2.5 s – 4.0 s | > 4.0 s | Loading — time to render the largest visible content element |
| **INP** (Interaction to Next Paint) | ≤ 200 ms | 200 ms – 500 ms | > 500 ms | Responsiveness — worst-case (near-worst-case) latency of user interactions across the whole page visit |
| **CLS** (Cumulative Layout Shift) | ≤ 0.1 | 0.1 – 0.25 | > 0.25 | Visual stability — sum of unexpected layout-shift scores in the worst 5s session window |

Diagnostic-only metrics (not ranking Core Web Vitals, but feed the other two and are what you
actually optimize):

| Metric | Good | Poor | Role |
|---|---|---|---|
| **TTFB** (Time to First Byte) | ≤ 0.8 s | > 1.8 s | First component of LCP; server/CDN/redirect latency |
| **FCP** (First Contentful Paint) | ≤ 1.8 s | > 3.0 s | First pixel painted; precedes LCP |
| **TBT** (Total Blocking Time, lab only) | ≤ 200 ms | > 600 ms | Lab proxy for INP (see §1.3) |

### 1.2 What replaced FID, and why

**First Input Delay (FID)** — good ≤ 100 ms — was replaced by **INP** as the official
responsiveness Core Web Vital on **March 12, 2024**. FID only measured the *input delay* of the
very first interaction on a page (time from user input to the browser starting to run the event
handler). It said nothing about how long the handler itself ran, or how long it took to paint the
result, and it ignored every interaction after the first. INP fixes both gaps: it observes **every
click, tap, and key press** on the page for the full visit and reports (roughly) the worst one,
covering the complete lifecycle of an interaction. Field data showed the gap was real: on mobile,
~93% of origins had "good" FID, but only ~65% had "good" INP — FID was systematically hiding
responsiveness problems that INP now surfaces. FID is fully deprecated; there's no reason to track
it in 2026.

### 1.3 How INP is measured, precisely

An interaction's latency has three phases, and INP is the sum of all three for the worst
interaction:

1. **Input delay** — from the physical input to the first event handler starting. Almost always
   caused by a **long task** (any main-thread work ≥ 50 ms) already running when the input arrives.
2. **Processing time** — the event handlers themselves running (synchronous JS).
3. **Presentation delay** — from the end of handler execution to the browser painting the next
   frame (style/layout/paint/composite work, including any work queued by `requestAnimationFrame`
   callbacks that ran before the frame was presented).

Measurement rule: only **click, tap, and physical/on-screen key press** interactions are observed
(hover, scroll, and drag alone don't count). On pages with many interactions, the single highest
value is discarded for every 50 interactions observed (statistical outlier handling), then the
**75th percentile across all page loads** is reported as the origin/page's INP.

**Common causes of bad INP on animation-heavy / WebGL sites** — this is the part that actually
bites award-style sites:

- **Long tasks blocking input delay**: heavy scene-graph construction, texture decode, or physics
  steps running synchronously on the main thread when the user clicks.
- **Non-composited animation properties**: animating `width`, `height`, `top`, `left`,
  `box-shadow`, `filter` forces layout/paint on the main thread every frame, competing directly
  with input handling. Animating `transform` and `opacity` instead lets the compositor thread do
  the work, leaving the main thread free to respond to input.
- **Scroll-driven JS (parallax, scroll-hijacking)**: a `scroll` listener that does synchronous
  layout reads (`getBoundingClientRect`, `offsetTop`, etc.) interleaved with writes causes **forced
  synchronous layout / layout thrashing** — the classic read-write-read-write pattern that costs
  10-100ms+ per frame on complex DOM.
- **Third-party scripts** (chat widgets, analytics, ad tags) parsing/executing on the main thread
  at an inopportune moment — invisible to the site's own profiling until you check a full trace.
- **Un-chunked JS**: heavy click handlers (opening a menu that re-renders a large component tree,
  recalculating a WebGL scene) not broken into yield points via `setTimeout(fn, 0)`,
  `requestIdleCallback`, or the newer `scheduler.yield()`.
- **Shader compilation / texture upload on interaction**: compiling shaders or uploading large
  textures synchronously in response to a click (e.g., "enter experience" button) blocks the main
  thread for the exact interaction being measured.

### 1.4 Field vs. lab — and why they disagree on award sites

- **Field (RUM / CrUX)**: real users, real devices, real networks, 75th percentile over a trailing
  28-day window. This is what Google uses for the Core Web Vitals ranking signal and what
  PageSpeed Insights reports as "Discover what your real users are experiencing." Requires enough
  traffic to have data; brand-new landing pages often show "not enough data."
- **Lab (Lighthouse, WebPageTest, your own CDP trace)**: one simulated run, fixed device/network
  throttling profile, fully reproducible, available the second the page exists — but it **cannot
  measure INP at all**, because INP requires a real interaction and a lab run is a single
  automated page load with no user. Lighthouse substitutes **Total Blocking Time (TBT)** as its
  interactivity proxy metric in the lab score.
- Lighthouse's Performance category weights (Lighthouse 10+, still current):
  **TBT 30%, LCP 25%, CLS 25%, FCP 10%, Speed Index 10%.** Note TBT (a lab proxy) is the single
  biggest slice — so a Lighthouse 95 does not guarantee good field INP, it just makes it likely.

### 1.5 Realistic budget for a WebGL landing page

Pragmatic numbers used by teams that ship award-caliber 3D sites without failing CWV:

- **Decouple first paint from WebGL init.** LCP's element should be DOM/CSS content (headline,
  hero image/video poster) that paints before the WebGL canvas has even started downloading its
  bundle. Never make LCP wait on Three.js/WebGL bootstrap.
- **Reserve canvas layout space up front** (explicit `width`/`height` or `aspect-ratio`) so canvas
  mount never causes a layout shift — protects CLS for free.
- Budget **shader compilation and texture decode off the input-critical path**: use
  `KHR_parallel_shader_compile`, compile/link across multiple idle callbacks, and don't gate a
  click handler on synchronous GPU work.
- Cap `devicePixelRatio` used for the WebGL canvas on mobile (e.g., clamp to 1.5–2× rather than a
  full 3× retina) — this is a fill-rate/INP lever, not just a battery one: full-res 3D rendering on
  every interaction frame directly competes with input handling on the main thread.
- Treat any JS executed before the page is interactive as TBT budget: keep it low enough that
  Lighthouse TBT stays under ~200 ms on a mid-tier mobile CPU throttle (4× slowdown), which is a
  reasonable proxy for real INP headroom.
- **Lazy-load the heavy 3D bundle** behind `requestIdleCallback`/dynamic `import()` after LCP, and
  show a lightweight (CSS-only) loading treatment if the experience needs guaranteed 3D-on-load.

### 1.6 Does Awwwards actually score performance? How much?

**Fetched directly from awwwards.com/about-evaluation/:** the published jury criteria are **Design
40%, Usability 30%, Creativity 20%, Content 10%** — performance/speed is not a named criterion.
Each submission goes to a jury of **at least 18 members**; the **3 scores furthest from the
average are automatically dropped** before the final score is computed. **Honorable Mention** = jury
score ≥ 6.5. **Site of the Day** = the highest-scored site(s) of the day (recent SOTD winners have
landed roughly 7.45–8.65/10). A site has a 3-month eligibility window from approval to win SOTD.
There is a separate **Developer Award**, restricted to SOTD winners, which does look harder at
code quality/technical execution and requires a score above 7 under its own, more technically
literate guidelines — this is the closest Awwwards gets to grading performance directly, and it's
optional and separate from the main jury score. **Practical conclusion**: a slow, heavy, gorgeous
WebGL site can absolutely win Site of the Day; it will not win the Developer Award, and it will
lose real users and organic search ranking if you ship it to production unchanged. Design for the
jury and re-tune for the field before launch — they are different targets.

---

## 2. Lighthouse / measurement tooling you can run here

### 2.1 What's actually available on this machine

| Tool | Needs Node? | Works here? | Notes |
|---|---|---|---|
| Headless Chrome CLI flags | No | **Yes** | `chrome.exe` at the given path; invoke from WSL bash with **Windows-style paths** |
| Chrome DevTools Protocol via raw WebSocket | No (stdlib only) | **Yes** | See §2.4 — hand-rolled client, no `pip install` needed |
| `websocket-client` / `requests` pip packages | No (Python) | `requests` present; `websocket-client` **not installed** in this shell — don't depend on it, use the stdlib recipe |
| Lighthouse CLI (`lighthouse` npm pkg) | **Yes** | Usable via Windows Node (`npx.cmd`), but it's an extra ~300MB install and one more moving part | Prefer CDP-direct for repeatable local captures; use Lighthouse for the actual scored report |
| `chrome-devtools-mcp` | **Yes** (`npx -y chrome-devtools-mcp@latest`) | Usable via Windows Node | MCP server, not a CLI script — wire into an MCP-aware client, not a plain Python pipeline |
| `playwright-mcp` / Playwright | **Yes** | Usable via Windows Node, but Playwright wants its own bundled browser download; pointing it at the system Chrome is extra config | Skip unless you need multi-browser (Firefox/WebKit) automation |
| `unlighthouse` | **Yes** (Node 22+, `npx unlighthouse --site <url>`) | Usable via Windows Node | Site-wide Lighthouse crawler (sitemap/robots/link discovery, parallel workers, dashboard) — great for auditing every page at once, overkill for one landing page |
| PageSpeed Insights API | No | **Only for a public URL** — will not accept `localhost` or `file://`. Returns both Lighthouse lab data and (subject to enough traffic) CrUX field data. Free tier works without a key; a key is recommended for repeated automated calls. | Use it post-deploy, not pre-deploy |
| WebPageTest | No (public instance) / Docker for self-host | Public instance needs a reachable URL, same constraint as PSI; self-hosting via Docker is possible but heavy for this use case | Gives filmstrip + video + multi-location/device — nice second opinion after deploy |
| axe-core (accessibility) | No | **Yes**, via CDN `<script>` tag injected into the page, or injected via CDP | See §3 |
| Pa11y | **Yes** | Usable via Windows Node (`npx.cmd pa11y <url>`), needs internet to fetch the package | Skip if axe-via-CDP already covers it |
| Nu HTML Checker (validator.w3.org) | No | **Yes**, plain HTTP POST from `requests`/`curl` | `curl -s -H "Content-Type: text/html; charset=utf-8" --data-binary @page.html "https://validator.w3.org/nu/?out=json"` |

**Bottom line for this machine**: everything that must run against a **local file** (pre-deploy) —
screenshots, DOM dumps, PDFs, animation capture, a11y scan, HTML validation — is achievable with
**zero Node dependency**, using headless Chrome CLI flags plus the stdlib CDP client below.
Everything that needs a **public URL** (PageSpeed Insights, WebPageTest, Google Rich Results Test)
is a post-deploy step, not a pre-flight one.

### 2.2 Headless Chrome CLI flags — what they do and where they break

Verified **[verified live]** against Chrome 152 headless=new on this machine unless noted.

| Flag | Effect | Gotchas |
|---|---|---|
| `--headless=new` | The current headless implementation — same binary/renderer/feature set as regular Chrome since Chrome 112 (no more "headless has a different rendering engine" problem from the old `--headless` mode). | Use this, not bare `--headless` (the legacy mode is deprecated and has real rendering differences, especially fonts). |
| `--screenshot[=path]` | One-shot viewport screenshot to PNG. **[verified live]** works with a UNC output path (`\\wsl.localhost\...`) as well as a normal `C:\...` path. | Captures **viewport only**, not full page, at the default/`--window-size` dimensions. For full-page or multi-point capture you need CDP (§2.4), not this flag. |
| `--window-size=W,H` | Sets the headless viewport size for `--screenshot`/`--print-to-pdf`. | Comma-separated per current docs (`412,892`); some older examples show `x` (`1280x1024`) — comma form is the current documented syntax. |
| `--print-to-pdf[=path]` | Renders the page to a PDF. **[verified live]**, incl. to a UNC path. | Add `--no-pdf-header-footer` to drop the default date/URL header-footer. Uses print CSS (`@media print`), not screen CSS — verify your print stylesheet separately (see checklist). |
| `--dump-dom` | Prints the serialized DOM (post-JS) to stdout. **[verified live]**. | Useful to sanity-check that client-side rendering actually produced content (SEO/SSR check) without needing CDP. |
| `--virtual-time-budget=ms` | Fast-forwards timers (`setTimeout`/`setInterval`)/rAF so the page believes `ms` milliseconds have passed, then the CLI action (screenshot/PDF/dump-dom) fires. | **Do not use this for animation capture** — it collapses wall-clock time, so a "2-second animation" is not something you can film with it; it's for letting delayed content finish appearing before a single static capture, not for sampling motion. For real animation capture, drive Chrome over CDP in real time instead (§2.4). |
| `--timeout=ms` | Hard cap on how long the CLI action waits before giving up. | Combine with `--virtual-time-budget` for deterministic single-shot captures of pages with deferred content. |
| `--hide-scrollbars` | Removes scrollbar chrome from screenshots. | Cosmetic only; doesn't affect layout. |
| `--force-device-scale-factor=1` | Pins DPR so screenshots are pixel-predictable across machines. | Omit if you specifically want to test hi-DPI rendering. |
| `--remote-debugging-port=PORT` | Opens the CDP HTTP+WebSocket endpoint on `localhost:PORT`. This is the door into everything in §2.4. | **Must pair with a dedicated `--user-data-dir`** — see next row. |
| `--user-data-dir=PATH` | Chrome profile directory. | **[verified live] Required for automation.** Without a fresh, dedicated profile dir, a second Chrome invocation just attaches to (or refuses to start alongside) any already-running Chrome on the machine — `chrome.exe --version` alone printed "Opening in existing browser session" on this machine because a real Chrome window was open. Always pass a throwaway `--user-data-dir` per automation run. |
| `--allow-chrome-scheme-url` | Lets `--print-to-pdf`/`--dump-dom`/`--screenshot` target `chrome://` URLs (Chrome 123+). | Irrelevant for normal page QA; useful for `chrome://gpu` sanity checks (see §5 WebGL note). |

**Gotcha found only by testing [verified live]**: when `--user-data-dir` resolves to a **UNC path**
(true whenever your working files live under WSL's own filesystem rather than `/mnt/c/...`, e.g.
`/tmp/...` or `~/...`), Chrome prints a long wall of stderr noise on every run —
`LockFileEx: Incorrect function (0x1)`, `Failed to reset the quota database`,
`Failed to grant sandbox access to cache directory`, `Unable to open the password store database`.
These come from subsystems (crash reporter, Quota, UKM, Password Manager, network sandbox) that
don't fully work over a network-style path and are **harmless** — navigation, screenshots, PDF, and
DOM dump all still succeed with exit code 0. Don't let this noise convince you the run failed;
check the exit code and the actual output file/bytes instead. If the noise bothers you (e.g., in
CI logs), put `--user-data-dir` under a real Windows drive path (`/mnt/c/Users/<you>/AppData/Local/Temp/...`)
instead of a WSL-native path — the *page/profile* location matters for this noise, not the file
being screenshotted.

### 2.3 Chrome DevTools Protocol — the domains you need

CDP is a JSON-RPC-ish protocol over WebSocket. Two connection levels:

- **Browser-level** endpoint (from `GET http://localhost:PORT/json/version` →
  `webSocketDebuggerUrl`) — for creating/closing tabs via the `Target` domain.
- **Page-level** endpoint (from `GET http://localhost:PORT/json/list`, the `webSocketDebuggerUrl`
  of an entry with `"type":"page"`) — connect directly to *this* socket to drive that one tab
  without needing `sessionId` plumbing at all. **This is what the recipe below uses** — simplest
  possible setup for single-page automation.

Domains that matter for QA automation:

| Domain | Key methods/events | Use |
|---|---|---|
| `Page` | `navigate`, `captureScreenshot`, `printToPDF`, `getLayoutMetrics`; events `loadEventFired`, `domContentEventFired`, `lifecycleEvent` | Navigation, screenshots, PDF, full-page dimensions |
| `Runtime` | `evaluate` (with `awaitPromise`, `returnByValue`) | Run arbitrary JS — this is how you wait on `document.fonts.ready`, scroll the page, read `PerformanceObserver` output, dispatch synthetic checks |
| `Network` | `enable`; events `requestWillBeSent`, `loadingFinished`, `loadingFailed` | Build your own "network idle" wait (no built-in CDP command for this — you track in-flight request IDs yourself) |
| `Emulation` | `setDeviceMetricsOverride` | Force a specific viewport/DPR/mobile emulation without relying on `--window-size` |
| `Tracing` | `start`, `end`; events `dataCollected`, `tracingComplete` | Record a full performance trace (loadable in `chrome://tracing` or DevTools Performance panel) — this is how you'd capture a real trace for INP/long-task analysis on a local file |
| `Target` | `attachToTarget`, `createTarget` | Only needed if you want multiple tabs/frames from one browser-level connection |
| `Input` | `dispatchMouseEvent`, `dispatchKeyEvent` | Synthesize a click/keypress to sample interaction latency in a lab script (imperfect INP proxy — see §2.5 caveat) |

`Page.captureScreenshot` full-page technique (no official "full page" flag before Chrome ~104):
call `Page.getLayoutMetrics`, read `cssContentSize.{width,height}`, then call
`captureScreenshot` again with `captureBeyondViewport: true` and a `clip` rectangle covering the
full content size. **[verified live]** — correctly captured a 1418×3150 px page from a 900px-tall
viewport in one shot.

### 2.4 Does `--headless=new` support what we need in 2026?

Yes, with one nuance worth stating precisely because it's usually reported wrong:

- **WebGL/WebGL2 works out of the box in `--headless=new` on this machine — no `--use-gl`,
  `--use-angle`, `--enable-unsafe-swiftshader`, or `--enable-gpu` flags needed.**
  **[verified live]**: `webgl2` context creation succeeded headlessly and reported
  `ANGLE (AMD, AMD Radeon(TM) Graphics ..., Direct3D11 vs_5_0 ps_5_0, D3D11)` as the renderer —
  i.e., **real hardware-accelerated GPU rendering**, not a software fallback. This is specific to
  running on real Windows hardware with a real GPU driver; the old "headless has no WebGL" advice
  is about headless Linux in containers/CI with no GPU device passed through, where you *do* still
  need `--use-angle=swiftshader`/`--enable-unsafe-swiftshader` to get a software GL fallback. On
  this WSL2-drives-real-Windows-Chrome setup, you get the real GPU for free.
- **CSS/WebGL/rAF animations run in real wall-clock time** under `--headless=new` — they are not
  frozen or virtualized unless you explicitly use `--virtual-time-budget`. **[verified live]** by
  reading `getComputedStyle(...).transform` across six screenshots taken 0.25s apart: the rotation
  matrix advanced correctly each time, and every screenshot's MD5 hash differed. (First attempt at
  this test used a rotating red square over a red-to-blue gradient background — all 6 frames came
  back byte-identical because a solid color rotating over a near-identical-color background is
  visually static, not because capture was broken. If your frame strip looks suspiciously
  identical, check contrast between the moving element and what's behind it before assuming the
  script is broken.)
- Fonts, `document.fonts.ready`, and standard DOM/CSSOM APIs all work exactly as in headed Chrome.
- What you still can't easily get: a literal user gesture for genuine INP measurement (see next
  point), and anything requiring actual OS-level compositing (e.g., screen recording of overlapping
  native windows) — not relevant for page QA.

### 2.5 The Python recipe (stdlib-only, tested end to end on this machine)

This is a single self-contained script. **No `pip install` required** — it hand-rolls the RFC 6455
WebSocket handshake/framing (since `websocket-client` is not installed in this environment and
there's no reason to require it). It was run to completion against a real local test page on this
exact machine, producing: a full-page PNG, 4 scroll-position PNGs, a 12-frame real-time animation
strip, a Tracing JSON, and a best-effort lab Web-Vitals sample. Save as `capture.py`:

```python
#!/usr/bin/env python3
"""
capture.py — self-contained WSL2 -> Windows headless Chrome CDP driver.
Pure stdlib. No pip installs. Tested against real Chrome 152 headless=new
launched from WSL2 bash, controlling the Windows chrome.exe binary.

Usage:
  python3 capture.py /path/to/local/file.html ./out_dir
"""
import base64
import hashlib
import json
import os
import socket
import struct
import subprocess
import sys
import time
import urllib.request

CHROME_EXE = r"/mnt/c/Program Files/Google/Chrome/Application/chrome.exe"
GUID = "258EAFA5-E914-47DA-95CA-C5AB0DC85B11"


# --------------------------------------------------------------------------
# Path helpers: this is the #1 WSL2-specific gotcha. chrome.exe is a WINDOWS
# process, so every path you hand it (profile dir, --print-to-pdf target,
# file:// URL) must be a Windows path, not a /mnt/c/... WSL path.
# --------------------------------------------------------------------------
def to_windows_path(wsl_path):
    """/mnt/c/... -> C:\\... , and pure-WSL paths (e.g. /tmp/...) -> a UNC
    \\\\wsl.localhost\\<Distro>\\... path. Both are valid Windows paths."""
    out = subprocess.check_output(
        ["wslpath", "-w", os.path.abspath(wsl_path)], text=True
    ).strip()
    return out


def to_file_url(wsl_path):
    """Build a file:// URL that Windows Chrome can open, from a WSL path.
    Verified against BOTH cases on a real machine:
      /mnt/c/Users/x/page.html -> file:///C:/Users/x/page.html
      /tmp/x/page.html         -> file://///wsl.localhost/Ubuntu/tmp/x/page.html
    The formula is the same in both cases: take the Windows path, flip
    backslashes to forward slashes, and prepend 'file:///' verbatim. For a
    UNC path the leading '\\\\' becomes '//' and concatenates with the three
    slashes of 'file:///' to give five slashes -- that is correct and is
    exactly what Chrome expects for a UNC file URL."""
    win = to_windows_path(wsl_path)
    return "file:///" + win.replace("\\", "/")


# --------------------------------------------------------------------------
# Minimal WebSocket client (RFC 6455) -- no websocket-client dependency.
# --------------------------------------------------------------------------
class CDP:
    def __init__(self, ws_url, timeout=30):
        self.timeout = timeout
        self._id = 0
        self._buf = b""
        u = ws_url.split("://", 1)[1]
        host_port, path = u.split("/", 1)
        path = "/" + path
        host, _, port = host_port.partition(":")
        port = int(port) if port else 80
        self.sock = socket.create_connection((host, port), timeout=timeout)
        self.sock.settimeout(timeout)
        key = base64.b64encode(os.urandom(16)).decode()
        req = (
            f"GET {path} HTTP/1.1\r\nHost: {host}:{port}\r\n"
            f"Upgrade: websocket\r\nConnection: Upgrade\r\n"
            f"Sec-WebSocket-Key: {key}\r\nSec-WebSocket-Version: 13\r\n\r\n"
        )
        self.sock.sendall(req.encode())
        resp = b""
        while b"\r\n\r\n" not in resp:
            chunk = self.sock.recv(4096)
            if not chunk:
                raise ConnectionError("WS handshake failed: connection closed")
            resp += chunk
        header, _, rest = resp.partition(b"\r\n\r\n")
        if b" 101 " not in header.split(b"\r\n", 1)[0]:
            raise ConnectionError(f"WS handshake rejected: {header[:200]!r}")
        expect = base64.b64encode(hashlib.sha1((key + GUID).encode()).digest())
        if expect not in header:
            raise ConnectionError("Sec-WebSocket-Accept mismatch")
        self._buf = rest

    def _send_frame(self, data, opcode=0x1):
        fin_op = 0x80 | opcode
        length = len(data)
        if length < 126:
            header = struct.pack("!BB", fin_op, 0x80 | length)
        elif length < (1 << 16):
            header = struct.pack("!BBH", fin_op, 0x80 | 126, length)
        else:
            header = struct.pack("!BBQ", fin_op, 0x80 | 127, length)
        mask_key = os.urandom(4)
        masked = bytes(b ^ mask_key[i % 4] for i, b in enumerate(data))
        self.sock.sendall(header + mask_key + masked)

    def _recv_exact(self, n):
        while len(self._buf) < n:
            chunk = self.sock.recv(65536)
            if not chunk:
                raise ConnectionError("socket closed mid-frame")
            self._buf += chunk
        out, self._buf = self._buf[:n], self._buf[n:]
        return out

    def _recv_frame(self):
        while True:
            b0, b1 = self._recv_exact(2)
            opcode = b0 & 0x0F
            masked = b1 & 0x80
            length = b1 & 0x7F
            if length == 126:
                length = struct.unpack("!H", self._recv_exact(2))[0]
            elif length == 127:
                length = struct.unpack("!Q", self._recv_exact(8))[0]
            if masked:
                mkey = self._recv_exact(4)
                payload = self._recv_exact(length)
                payload = bytes(b ^ mkey[i % 4] for i, b in enumerate(payload))
            else:
                payload = self._recv_exact(length)
            if opcode == 0x9:      # ping
                self._send_frame(payload, opcode=0xA)
                continue
            if opcode == 0xA:      # pong
                continue
            if opcode == 0x8:      # close
                raise ConnectionError("WebSocket closed by peer")
            return payload

    def send(self, method, params=None, session_id=None):
        self._id += 1
        msg = {"id": self._id, "method": method, "params": params or {}}
        if session_id:
            msg["sessionId"] = session_id
        self._send_frame(json.dumps(msg).encode("utf-8"))
        return self._id

    def recv(self, timeout=None):
        self.sock.settimeout(timeout if timeout is not None else self.timeout)
        return json.loads(self._recv_frame().decode("utf-8"))

    def call(self, method, params=None, session_id=None, timeout=None):
        want_id = self.send(method, params, session_id)
        t_end = time.time() + (timeout or self.timeout)
        while True:
            m = self.recv(timeout=max(0.05, t_end - time.time()))
            if m.get("id") == want_id and m.get("sessionId") == session_id:
                if "error" in m:
                    raise RuntimeError(f"{method} failed: {m['error']}")
                return m.get("result", {})

    def wait_for(self, event_method, predicate=None, timeout=30):
        t_end = time.time() + timeout
        while True:
            m = self.recv(timeout=max(0.05, t_end - time.time()))
            if m.get("method") == event_method:
                if predicate is None or predicate(m.get("params", {})):
                    return m.get("params", {})

    def close(self):
        try:
            self._send_frame(b"", opcode=0x8)
        except OSError:
            pass
        self.sock.close()


# --------------------------------------------------------------------------
# Chrome process lifecycle
# --------------------------------------------------------------------------
def launch_chrome(port, profile_dir, headless=True, window_size="1440,900",
                   extra_flags=None):
    os.makedirs(profile_dir, exist_ok=True)
    win_profile = to_windows_path(profile_dir)
    flags = [
        CHROME_EXE,
        f"--remote-debugging-port={port}",
        f"--user-data-dir={win_profile}",
        "--no-first-run",
        "--no-default-browser-check",
        f"--window-size={window_size}",
        "--hide-scrollbars",
        "--force-device-scale-factor=1",
    ]
    if headless:
        flags.append("--headless=new")
    flags += extra_flags or []
    flags.append("about:blank")
    # Discard stdout/stderr: headless Chrome on a WSL UNC profile path prints
    # a wall of harmless "LockFileEx / quota database / sandbox access"
    # warnings from subsystems (crash reporter, password store, UKM) that
    # don't work over a network-path profile dir. Exit code and the actual
    # output file are what to trust, not stderr noise.
    proc = subprocess.Popen(flags, stdout=subprocess.DEVNULL,
                             stderr=subprocess.DEVNULL)
    _wait_for_cdp(port, timeout=20)
    return proc


def _wait_for_cdp(port, timeout=20):
    t_end = time.time() + timeout
    last_err = None
    while time.time() < t_end:
        try:
            with urllib.request.urlopen(
                f"http://localhost:{port}/json/version", timeout=1
            ) as r:
                return json.loads(r.read())
        except Exception as e:  # noqa: BLE001
            last_err = e
            time.sleep(0.2)
    raise RuntimeError(f"Chrome did not open CDP port {port}: {last_err}")


def get_page_target(port):
    with urllib.request.urlopen(f"http://localhost:{port}/json/list") as r:
        targets = json.loads(r.read())
    return next(t for t in targets if t["type"] == "page")


# --------------------------------------------------------------------------
# High-level capture helpers
# --------------------------------------------------------------------------
def navigate_and_wait(cdp, url, timeout=30):
    cdp.call("Page.enable")
    cdp.call("Runtime.enable")
    cdp.call("Network.enable")
    cdp.send("Page.navigate", {"url": url})
    cdp.wait_for("Page.loadEventFired", timeout=timeout)


def wait_fonts_ready(cdp, timeout=10):
    cdp.call(
        "Runtime.evaluate",
        {"expression": "document.fonts.ready.then(()=>true)",
         "awaitPromise": True, "returnByValue": True},
        timeout=timeout,
    )


def wait_network_idle(cdp, idle_ms=500, timeout=15):
    """Track in-flight requests via the Network domain until none have been
    outstanding for idle_ms. Network.enable must already be on."""
    inflight = set()
    t_start = time.time()
    t_end = t_start + timeout
    last_activity = time.time()
    while True:
        remaining_to_idle = (last_activity + idle_ms / 1000.0) - time.time()
        if not inflight and remaining_to_idle <= 0:
            return
        if time.time() >= t_end:
            return  # give up waiting; caller can still proceed
        try:
            m = cdp.recv(timeout=max(0.05, min(remaining_to_idle, 0.5)))
        except (socket.timeout, TimeoutError):
            continue
        method = m.get("method", "")
        p = m.get("params", {})
        if method == "Network.requestWillBeSent":
            inflight.add(p["requestId"])
            last_activity = time.time()
        elif method in ("Network.loadingFinished", "Network.loadingFailed"):
            inflight.discard(p["requestId"])
            last_activity = time.time()


def full_page_screenshot(cdp, out_path, max_height=16000):
    lm = cdp.call("Page.getLayoutMetrics")
    size = lm["cssContentSize"]
    shot = cdp.call("Page.captureScreenshot", {
        "format": "png",
        "captureBeyondViewport": True,
        "clip": {"x": 0, "y": 0, "width": size["width"],
                 "height": min(size["height"], max_height), "scale": 1},
    })
    with open(out_path, "wb") as f:
        f.write(base64.b64decode(shot["data"]))
    return size


def scroll_and_capture(cdp, positions_px, out_dir, prefix="scroll"):
    """positions_px: list of Y offsets. Writes {prefix}_{i}_{y}.png for each."""
    paths = []
    for i, y in enumerate(positions_px):
        cdp.call("Runtime.evaluate", {"expression": f"window.scrollTo(0,{y})"})
        # let layout/paint settle for one frame before shooting
        cdp.call("Runtime.evaluate", {
            "expression": "new Promise(r=>requestAnimationFrame(()=>requestAnimationFrame(r)))",
            "awaitPromise": True,
        })
        shot = cdp.call("Page.captureScreenshot", {"format": "png"})
        path = os.path.join(out_dir, f"{prefix}_{i}_{y}.png")
        with open(path, "wb") as f:
            f.write(base64.b64decode(shot["data"]))
        paths.append(path)
    return paths


def capture_frame_strip(cdp, duration_s, fps, out_dir, clip=None, prefix="frame"):
    """Captures a real-time (wall-clock) animation frame strip.
    CSS/WebGL/rAF animations keep running in real time under headless=new,
    so this is genuinely sampling the animation, not a frozen virtual clock."""
    interval = 1.0 / fps
    n = max(1, int(duration_s * fps))
    paths = []
    t0 = time.time()
    for i in range(n):
        target_t = t0 + i * interval
        sleep_for = target_t - time.time()
        if sleep_for > 0:
            time.sleep(sleep_for)
        params = {"format": "png"}
        if clip:
            params["clip"] = clip
        shot = cdp.call("Page.captureScreenshot", params)
        path = os.path.join(out_dir, f"{prefix}_{i:03d}.png")
        with open(path, "wb") as f:
            f.write(base64.b64decode(shot["data"]))
        paths.append(path)
    elapsed = time.time() - t0
    return paths, elapsed


def read_web_vitals(cdp, sample_ms=1000):
    """Inject a short-lived PerformanceObserver and read back whatever LCP /
    CLS / longtask entries have fired within sample_ms. Good enough for a lab
    script; real INP needs a genuine user session (see note in the report)."""
    expr = f"""
    new Promise((resolve) => {{
      const out = {{ lcp: null, cls: 0, longtasks: [] }};
      try {{
        new PerformanceObserver((list) => {{
          const es = list.getEntries();
          if (es.length) out.lcp = es[es.length-1].startTime;
        }}).observe({{type: 'largest-contentful-paint', buffered: true}});
      }} catch (e) {{}}
      try {{
        new PerformanceObserver((list) => {{
          for (const e of list.getEntries()) if (!e.hadRecentInput) out.cls += e.value;
        }}).observe({{type: 'layout-shift', buffered: true}});
      }} catch (e) {{}}
      try {{
        new PerformanceObserver((list) => {{
          for (const e of list.getEntries()) out.longtasks.push(e.duration);
        }}).observe({{type: 'longtask', buffered: true}});
      }} catch (e) {{}}
      setTimeout(() => resolve(JSON.stringify(out)), {sample_ms});
    }})
    """
    r = cdp.call("Runtime.evaluate", {"expression": expr, "awaitPromise": True,
                                       "returnByValue": True},
                 timeout=(sample_ms / 1000.0) + 10)
    return json.loads(r["result"]["value"])


def record_trace(cdp, seconds, out_path,
                  categories=("devtools.timeline", "v8", "disabled-by-default-devtools.screenshot")):
    cdp.call("Tracing.start", {
        "categories": ",".join(categories),
        "transferMode": "ReportEvents",
    })
    events = []

    def _drain_until_complete(deadline):
        while True:
            m = cdp.recv(timeout=max(0.1, deadline - time.time()))
            if m.get("method") == "Tracing.dataCollected":
                events.extend(m["params"].get("value", []))
            elif m.get("method") == "Tracing.tracingComplete":
                return

    t_end = time.time() + seconds
    while time.time() < t_end:
        try:
            m = cdp.recv(timeout=max(0.05, t_end - time.time()))
            if m.get("method") == "Tracing.dataCollected":
                events.extend(m["params"].get("value", []))
        except (socket.timeout, TimeoutError):
            pass
    cdp.send("Tracing.end")
    _drain_until_complete(time.time() + 15)
    with open(out_path, "w") as f:
        json.dump({"traceEvents": events}, f)
    return len(events)


# --------------------------------------------------------------------------
# Demo: the exact recipe the brief asked for.
# --------------------------------------------------------------------------
def main():
    if len(sys.argv) < 3:
        print(f"usage: {sys.argv[0]} <local.html> <out_dir>", file=sys.stderr)
        sys.exit(1)
    html_path, out_dir = sys.argv[1], sys.argv[2]
    os.makedirs(out_dir, exist_ok=True)
    port = 9500
    profile = os.path.join(out_dir, "_chrome-profile")

    proc = launch_chrome(port, profile)
    try:
        page = get_page_target(port)
        cdp = CDP(page["webSocketDebuggerUrl"])
        try:
            url = to_file_url(html_path)
            print("Navigating:", url)
            navigate_and_wait(cdp, url)
            wait_fonts_ready(cdp)
            wait_network_idle(cdp, idle_ms=500, timeout=10)
            print("Page settled (loaded + fonts + network idle).")

            vitals = read_web_vitals(cdp, sample_ms=800)
            print("Lab vitals sample:", vitals)

            size = full_page_screenshot(cdp, os.path.join(out_dir, "full_page.png"))
            print("Full-page screenshot:", size)

            positions = [0, 600, 1200, 1800]
            shots = scroll_and_capture(cdp, positions, out_dir)
            print("Scroll screenshots:", shots)

            frames, elapsed = capture_frame_strip(
                cdp, duration_s=2.0, fps=6, out_dir=out_dir,
                clip={"x": 80, "y": 500, "width": 120, "height": 120, "scale": 1},
            )
            print(f"Animation frame strip: {len(frames)} frames in {elapsed:.2f}s")

            record_trace(cdp, seconds=1.5, out_path=os.path.join(out_dir, "trace.json"))
            print("Trace saved.")
        finally:
            cdp.close()
    finally:
        proc.terminate()
        try:
            proc.wait(timeout=5)
        except subprocess.TimeoutExpired:
            proc.kill()


if __name__ == "__main__":
    main()
```

**Actual output from a live run on this machine** (`richer_test.html`, a page with a hero block, a
CSS-rotating div, a 2500px tall gradient section, a 400ms-delayed image, and a 700ms-delayed layout
shift):

```
Navigating: file://///wsl.localhost/Ubuntu/tmp/.../richer_test.html
Page settled (loaded + fonts + network idle).
Lab vitals sample: {'lcp': 56, 'cls': 0, 'longtasks': []}
Full-page screenshot: {'x': 0, 'y': 0, 'width': 1418, 'height': 3150}
Scroll screenshots: ['./run1/scroll_0_0.png', './run1/scroll_1_600.png', './run1/scroll_2_1200.png', './run1/scroll_3_1800.png']
Animation frame strip: 12 frames in 1.90s
Trace saved.
EXIT: 0
```

12 files + a trace.json landed on disk, full-page dimensions matched the page's real content
height exactly, and the frame strip's wall-clock duration (1.90s for a requested 2.0s at 6fps)
confirms real-time capture, not virtual time.

**Known limitation of `read_web_vitals()`**: it's a single automated page load with no real user,
so its "LCP"/"CLS" numbers are a rough same-session approximation, not what CrUX/RUM would report
— useful as a fast pre-flight smoke check ("did CLS obviously blow up after my change"), not a
substitute for field data. It **cannot** produce a real INP number at all, because INP requires an
actual user interaction; `Input.dispatchMouseEvent` can synthesize a click and you can measure the
resulting `event` performance-entry duration as a crude proxy, but it isn't equivalent to a real
touchscreen tap's input-delay characteristics.

### 2.6 What each tool is *for*, concretely

- **Local pre-flight (this machine, no URL needed)**: the Python recipe above, plus CLI
  `--dump-dom` (did SSR/CSR actually produce content), `--print-to-pdf` (print stylesheet check),
  axe-via-CDP (§3), Nu HTML Checker via `curl`.
- **Once deployed to a public URL**: PageSpeed Insights API/UI for the official Lighthouse score
  plus (traffic permitting) real CrUX field data; WebPageTest for filmstrip/video and multi-
  location/device testing; `unlighthouse` if you want every page on the site scanned at once
  instead of hand-picking URLs.
- **Interactive/agentic debugging**: `chrome-devtools-mcp` if your tooling can speak MCP — it
  wraps the same CDP primitives used above (trace recording, screenshots, network inspection)
  behind tool calls, useful when an AI agent needs to *drive* Chrome conversationally rather than
  run a fixed script.

---

## 3. Accessibility for motion-heavy sites

### 3.1 WCAG 2.2 — what's new, and the exact AA bar

WCAG 2.2 added 9 success criteria over 2.1 (confirmed against the W3C's own "What's New in WCAG
2.2" page). Levels, exactly:

| SC | Name | Level | What it requires |
|---|---|---|---|
| 2.4.11 | Focus Not Obscured (Minimum) | **AA** | The focused element must not be *entirely* hidden by other author content (sticky headers/footers, cookie banners) |
| 2.5.7 | Dragging Movements | **AA** | Anything operable by a drag gesture must have a single-pointer (click/tap) alternative that doesn't require dragging |
| 2.5.8 | Target Size (Minimum) | **AA** | Pointer targets ≥ **24×24 CSS px**, unless: spacing (unhit target's 24px circle doesn't overlap an adjacent target's), equivalent alternative exists, target is inline within a text run, size is user-agent-controlled, or the size is legally/essentially required |
| 3.3.8 | Accessible Authentication (Minimum) | **AA** | No cognitive-function test (remembering a password, solving a puzzle) required for login unless an alternative exists (password managers/paste must be allowed) |
| 3.2.6 | Consistent Help | A | Help mechanisms (contact, chat, FAQ) appear in the same relative order on every page they exist on |
| 3.3.7 | Redundant Entry | A | Don't make users re-enter information they already gave earlier in the same process |
| 2.4.12 | Focus Not Obscured (Enhanced) | AAA | Stricter version — *no part* of the focus indicator may be hidden |
| 2.4.13 | Focus Appearance | AAA | Focus indicator ≥ 3:1 contrast against adjacent colors and a minimum area/thickness |
| 3.3.9 | Accessible Authentication (Enhanced) | AAA | No cognitive-function test at all, full stop |

**The two juries/users actually notice on a motion-heavy landing page are 2.4.11 (Focus Not
Obscured) and 2.5.8 (Target Size)** — both routinely broken by custom cursors, full-bleed hero
canvases, and "minimal" nav dots/hamburgers sized at 16-20px.

Pre-existing (WCAG 2.0/2.1, still fully in force, and the ones automated tools actually catch most
reliably):

- **1.4.3 Contrast (Minimum), AA**: normal text ≥ **4.5:1**; **large text ≥ 3:1** (large = ≥18pt/24px
  regular weight, or ≥14pt/18.66px bold).
- **1.4.11 Non-text Contrast, AA**: UI components (buttons, form borders, focus rings) and
  meaningful graphical objects ≥ **3:1** against adjacent colors.
- **2.4.7 Focus Visible, AA**: keyboard focus must be visibly indicated — `outline: none` with
  nothing substituted is a straight fail, and a very common one on "clean" award-site designs.
- **1.4.6 Contrast (Enhanced), AAA** (aspirational, not required): 7:1 / 4.5:1 — nice for hero
  headline text if you can get it for free.

**Text over images/video**: there's no separate WCAG number for this — the same 4.5:1/3:1 ratios
apply, but they must hold against the **busiest, lightest part of the actual rendered background**,
not just the image's average color. Automated contrast checkers (axe, Lighthouse) generally
**can't verify this** because they read DOM/CSS color values, not rendered pixels under a photo —
this is one of the few checks that needs an actual human eyeballing real screenshots (see §2's
frame-strip/scroll-capture output for exactly that). Practical fixes: a gradient scrim
(`linear-gradient(transparent, rgba(0,0,0,.6))`) behind caption zones, a solid/semi-opaque bar
behind text, or `text-shadow`/stroke as a *supplement* (not a replacement — shadows don't reliably
raise measured contrast, only perceived legibility).

### 3.2 `prefers-reduced-motion` — what must actually stop

Two values: `no-preference` (default) and `reduce`. Widely supported since ~2020. What "reduce"
means in practice for an award-style landing page — **all of these must stop or be replaced**, not
just toned down:

- **Parallax scrolling** (elements moving at a different rate than scroll position) → pin elements
  or use a subtle opacity/cross-fade instead.
- **Autoplaying background video/animation loops** → freeze on a static frame, or require explicit
  play.
- **Scroll-hijacking** (JS intercepting native scroll to drive a custom animated transition between
  sections) → fall back to native scroll entirely; this is as much a motion-sickness issue as an
  aesthetic one (vestibular disorders + hijacked scroll acceleration is a well-documented trigger).
- **Large transform/scale/rotation entrance animations** on scroll-into-view → either remove or
  replace with a simple opacity fade of ≤ a few percent movement.
- **Auto-advancing carousels/sliders** → stop auto-advance; keep manual controls.

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.001ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.001ms !important;
    scroll-behavior: auto !important;
  }
  .parallax-layer { transform: none !important; }
  video[autoplay] { display: none; } /* or swap to a static poster element */
}
```

(Near-zero duration rather than `0` avoids some browsers' historical quirks with `0`-duration
transitions not firing their `transitionend`/completion logic that other code may depend on.)

Related, newer, and worth including even though support is still patchy in 2026 — treat as
progressive enhancement:

- `prefers-reduced-transparency: reduce` — drop `backdrop-filter: blur()` glass effects to a solid
  background for users who've asked for it (also helps low-end GPUs).
- `prefers-contrast: more` — offer a higher-contrast palette swap for users who've asked for it at
  the OS level.
- `prefers-reduced-data: reduce` — Chrome/Android only as of 2026; skip autoplay hero video, serve
  a lower-resolution asset.

### 3.3 Keyboard, focus, structure

- **Skip link**: a visually-hidden-until-focused `<a href="#main">Skip to content</a>` as the
  literal first focusable element in the DOM — essential on any site with a large decorative nav or
  intro animation before the main content.
- **Landmarks**: one `<main>`, semantic `<header>`/`<nav>`/`<footer>`, exactly one `<h1>`, and a
  logical (not just visual) heading order — screen reader users navigate by landmark/heading jump
  far more than linearly.
- **Keyboard traps in custom scroll**: scroll-hijacking implementations frequently break native Page
  Down/Space/arrow-key scrolling and can strand focus inside a "section" the JS thinks it owns.
  Test explicitly: unplug the mouse, Tab through the entire page, Shift+Tab back, and try Page
  Down/End/Home — the DOM focus order should match the visual order, and nothing should intercept
  standard scroll keys without an escape.
- **`aria-hidden` on decorative canvas**: a WebGL/canvas element that's purely decorative
  (background particles, ambient 3D scene with no independently meaningful content) should get
  `aria-hidden="true"` and, if it's ever focusable by default, `tabindex="-1"` — otherwise screen
  readers announce an empty, meaningless "canvas" element to every user.
- **Alt text**: every meaningful `<img>` needs real alt text; purely decorative images get
  `alt=""` (empty, not omitted) so they're skipped rather than read as their filename.

### 3.4 The split-text / per-character-span screen-reader trick

Motion-heavy sites routinely split headline text into per-character or per-word `<span>`s for
stagger animations (GSAP SplitText and equivalents). Unpatched, this is an accessibility disaster:
screen readers either read every fragment separately with unnatural pauses, or in the worst case
spell out single letters. Fix:

```html
<h1 aria-label="Full readable sentence goes here">
  <span aria-hidden="true">
    <span class="char">F</span><span class="char">u</span><span class="char">l</span>…
  </span>
</h1>
```

Put the real, complete text in `aria-label` on the outer heading element, and mark the entire
animated/split inner structure `aria-hidden="true"`. Screen readers then announce the clean
`aria-label` and completely ignore the fragmented spans used for the visual animation. Verify with
an actual screen reader (VoiceOver on macOS/iOS or NVDA on Windows) — this is one of the few things
axe-core cannot catch automatically, because the markup is technically valid; it just doesn't
*sound* right.

### 3.5 Tools you can run here (no Node)

**axe-core via CDN, injected into the live page** — run this in the DevTools console, or inject via
CDP `Runtime.evaluate` after your existing capture script's `navigate_and_wait()`:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/axe-core/4.10.2/axe.min.js"></script>
<script>
  axe.run().then(results => {
    console.log(`${results.violations.length} violations`);
    results.violations.forEach(v =>
      console.log(v.id, v.impact, v.help, v.nodes.length, "node(s)")
    );
  });
</script>
```

From the Python recipe, inject it programmatically without any local axe install:

```python
axe_src = urllib.request.urlopen(
    "https://cdnjs.cloudflare.com/ajax/libs/axe-core/4.10.2/axe.min.js"
).read().decode("utf-8")
cdp.call("Runtime.evaluate", {"expression": axe_src})               # defines window.axe
r = cdp.call("Runtime.evaluate", {
    "expression": "axe.run().then(r => JSON.stringify(r.violations))",
    "awaitPromise": True, "returnByValue": True,
})
violations = json.loads(r["result"]["value"])
```

Each violation object carries `id`, `impact` (`minor`/`moderate`/`serious`/`critical`), `help`,
`helpUrl`, and a `nodes` array with the actual failing selectors — enough to triage without a UI.

**Nu HTML Checker (validator.w3.org)** — plain HTTP, no Node:

```bash
curl -s -H "Content-Type: text/html; charset=utf-8" \
  --data-binary @page.html "https://validator.w3.org/nu/?out=json" | python3 -m json.tool
```

**Pa11y** needs Node but Windows Node is available on this machine: `"/mnt/c/Program Files/nodejs/npx.cmd" pa11y <url>` works if you want its specific ruleset/report format; not necessary if axe-via-CDP already covers your checks.

---

## 4. SEO / meta / social

### 4.1 Title and description

- **Title**: 50–60 characters (~580px rendered width is the practical Google truncation point),
  unique per page, primary keyword near the front, brand name as a suffix (`Page Title — Brand`).
- **Meta description**: 150–160 characters, includes a concrete reason to click (not keyword
  stuffing — Google frequently rewrites descriptions it judges low-quality, so write for humans).

### 4.2 Open Graph + Twitter/X cards

Required OG properties per the ogp.me spec: `og:title`, `og:type`, `og:image`, `og:url`.
Strongly recommended: `og:description`, `og:site_name`, `og:locale`, `og:image:width`,
`og:image:height`, `og:image:alt`.

| Platform | Card type | Image size | Notes |
|---|---|---|---|
| Facebook/LinkedIn/Slack/Discord/Telegram (OG consumers) | — | **1200×630** (1.91:1), min 200×200, < 8MB, JPG/PNG | This is the de-facto universal size; smaller images get upscaled/cropped unpredictably |
| Twitter/X | `summary_large_image` | **1200×630** (or 2:1), min 300×157 | Set `twitter:card=summary_large_image` explicitly or platforms fall back to a small thumbnail; X's own crawler behavior has been inconsistent through 2026, but every other OG consumer still reads these tags, so keep them regardless |

```html
<meta property="og:title" content="…">
<meta property="og:type" content="website">
<meta property="og:url" content="https://example.com/">
<meta property="og:description" content="…">
<meta property="og:site_name" content="…">
<meta property="og:locale" content="en_US">
<meta property="og:image" content="https://example.com/og-image.jpg">
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
<meta property="og:image:alt" content="…">
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="…">
<meta name="twitter:description" content="…">
<meta name="twitter:image" content="https://example.com/og-image.jpg">
```

### 4.3 Favicon / app icon set for 2026

A complete, future-proof set:

| File | Size | Purpose |
|---|---|---|
| `favicon.svg` | any (vector) | Modern browsers, sharp at any zoom/DPI |
| `favicon.ico` | 32×32 (multi-res ideally) | Legacy fallback, browser tab |
| `apple-touch-icon.png` | **180×180** | iOS home-screen icon — no transparency (iOS adds a white background and rounds the corners itself) |
| `icon-192.png` | 192×192 | Android/PWA manifest icon |
| `icon-512.png` | 512×512 | Android/PWA manifest icon, used for splash screens |
| `icon-maskable.png` | 512×512, ~40% safe-zone padding | Android adaptive icon shape (circle/squircle/etc. mask applied by the OS) |
| `site.webmanifest` | — | References the above + `theme_color`/`background_color`/`name` |

Google's own favicon guidance (developers.google.com): supports BMP/GIF/ICO/PNG/JPEG/PPM/TIFF,
must be a 1:1 square, minimum 8×8px but recommend **> 48px** for it to look good across surfaces,
declared once per hostname (not per subdirectory), URL kept stable.

```html
<link rel="icon" href="/favicon.svg" type="image/svg+xml">
<link rel="icon" href="/favicon.ico" sizes="32x32">
<link rel="apple-touch-icon" href="/apple-touch-icon.png">
<link rel="manifest" href="/site.webmanifest">
```

### 4.4 `theme-color`, canonical, structured data

```html
<meta name="theme-color" content="#0b0b0f" media="(prefers-color-scheme: dark)">
<meta name="theme-color" content="#ffffff" media="(prefers-color-scheme: light)">
<link rel="canonical" href="https://example.com/current-page/">
```

- **Canonical**: always an **absolute** URL, self-referencing on every indexable page (even the
  "canonical" one points to itself). The most common real-world bug: a relative canonical URL, or
  one that silently points to a staging/www-vs-non-www variant.
- **JSON-LD** — put it in `<head>` or end of `<body>`, validate with Google's Rich Results Test:

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "…",
  "url": "https://example.com",
  "logo": "https://example.com/logo.png",
  "sameAs": [
    "https://www.instagram.com/…",
    "https://www.linkedin.com/company/…"
  ]
}
</script>
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "WebSite",
  "name": "…",
  "url": "https://example.com"
}
</script>
```

Only add `Product`/`aggregateRating` structured data if the properties are genuinely populated
(real price/availability/reviews) — fabricated or stale rich-result data is a Google Search Console
manual-action risk, not just bad practice.

### 4.5 Sitemap, robots, hreflang (RU/UK/EN)

```
# robots.txt
User-agent: *
Allow: /
Sitemap: https://example.com/sitemap.xml
```

`sitemap.xml` lists canonical URLs with `<lastmod>`; keep it in sync with what's actually
indexable (no noindex'd or redirected URLs in it).

For a site with Russian/Ukrainian/English versions, hreflang must be **reciprocal** — every
language variant lists *all* variants including itself, plus an `x-default` fallback:

```html
<link rel="alternate" hreflang="ru" href="https://example.com/ru/">
<link rel="alternate" hreflang="uk" href="https://example.com/uk/">
<link rel="alternate" hreflang="en" href="https://example.com/en/">
<link rel="alternate" hreflang="x-default" href="https://example.com/en/">
```

Use ISO 639-1 language codes (`ru`, `uk`, `en`); add a region subtag only if you actually have
region-specific content (`en-US` vs `en-GB`), not by default.

### 4.6 Copy-paste `<head>` block

```html
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">

<title>Page Title — Brand</title>
<meta name="description" content="150-160 character description with a real reason to click.">
<link rel="canonical" href="https://example.com/">

<meta name="theme-color" content="#0b0b0f" media="(prefers-color-scheme: dark)">
<meta name="theme-color" content="#ffffff" media="(prefers-color-scheme: light)">

<link rel="icon" href="/favicon.svg" type="image/svg+xml">
<link rel="icon" href="/favicon.ico" sizes="32x32">
<link rel="apple-touch-icon" href="/apple-touch-icon.png">
<link rel="manifest" href="/site.webmanifest">

<link rel="alternate" hreflang="ru" href="https://example.com/ru/">
<link rel="alternate" hreflang="uk" href="https://example.com/uk/">
<link rel="alternate" hreflang="en" href="https://example.com/en/">
<link rel="alternate" hreflang="x-default" href="https://example.com/en/">

<meta property="og:title" content="Page Title">
<meta property="og:type" content="website">
<meta property="og:url" content="https://example.com/">
<meta property="og:description" content="Same or tighter version of the meta description.">
<meta property="og:site_name" content="Brand">
<meta property="og:locale" content="en_US">
<meta property="og:image" content="https://example.com/og-image.jpg">
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
<meta property="og:image:alt" content="Describe the image.">

<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="Page Title">
<meta name="twitter:description" content="Same as og:description.">
<meta name="twitter:image" content="https://example.com/og-image.jpg">

<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "Brand",
  "url": "https://example.com",
  "logo": "https://example.com/logo.png",
  "sameAs": ["https://www.instagram.com/brand", "https://www.linkedin.com/company/brand"]
}
</script>
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "WebSite",
  "name": "Brand",
  "url": "https://example.com"
}
</script>
</head>
```

---

## 5. Cross-browser / device reality

Safari (desktop and iOS) is where award-style sites actually break in production — it's the
engine furthest from Chromium's behavior and the one designers/devs test least, because most dev
machines are Windows/Linux with Chrome.

| Issue | Detail | Fix |
|---|---|---|
| `backdrop-filter` | Supported in Safari desktop/iOS since Safari 9, but **keep `-webkit-backdrop-filter` alongside the unprefixed property** — older Safari (still in the field via old iOS devices that can't update) needs the prefix, and WebKit's own bug tracker lists active issues combining `backdrop-filter` with `mix-blend-mode` (e.g., `backdrop-filter: brightness` breaking under `mix-blend-mode: exclusion`) and with scrolling/compositing containers. | `-webkit-backdrop-filter: blur(20px); backdrop-filter: blur(20px);` — and test glass-morphism UI specifically while scrolling on a real iPhone, not just a static screenshot |
| `100vh` on iOS Safari | The address bar expands/retracts as you scroll, and plain `vh` units are pinned to the *large* viewport (bar hidden) — a `100vh` hero can be taller than the visible area when the bar is showing, clipping content/CTAs. | `height: 100vh; height: 100dvh;` (dvh as progressive enhancement over vh, not a replacement — keep the vh fallback for older browsers). Caveat: `dvh` itself changes value continuously while the address bar animates during scroll, which can cause visible resize jank on an element sized with it — don't use `dvh` for anything that must stay perfectly static during a scroll gesture. |
| `mix-blend-mode` | Multiple open WebKit bugs: hairline gaps with `isolation: isolate` + `plus-lighter`, failures inside composited/accelerated scrollers, incorrect results vs. Chrome/Firefox on some blend modes, black screen for fullscreen video combined with a blend mode. | Treat any blend-mode effect as "verify on real Safari," not "verify in a Chromium screenshot tool" |
| WebGL differences | Safari's WebGL runs through its own ANGLE/Metal backend; shader precision and extension availability differ from Chrome's ANGLE/D3D or native GL paths; iOS is more aggressive about reclaiming/losing WebGL contexts under memory pressure. | Always implement `webglcontextlost`/`webglcontextrestored` handlers; don't assume every extension you tested in Chrome exists in Safari |
| Video autoplay (iOS) | Requires **`muted` + `playsinline`** on the `<video>` element; without `playsinline` it forces fullscreen playback on iPhone; playback pauses automatically when scrolled out of view and won't resume without `muted`/no audio track. | `<video autoplay muted playsinline loop>` |
| `scroll-behavior: smooth` | CSS-native smooth scroll has been supported in Safari since 15.4, but custom easing (the kind award sites use) still typically needs a JS smoother (Lenis, GSAP ScrollSmoother) for consistent cross-browser feel — and **must be disabled under `prefers-reduced-motion`**. | Gate any custom scroll smoothing library behind the media query |
| `-webkit-` prefixes still worth keeping in 2026 | `-webkit-backdrop-filter`, `-webkit-appearance` (form control resets), `-webkit-line-clamp` (multi-line truncation — pair with unprefixed `line-clamp` where supported), `-webkit-font-smoothing` (cosmetic antialiasing), `-webkit-tap-highlight-color` (kill the gray tap flash on mobile links/buttons) | Keep these in a base reset, they cost nothing on browsers that don't need them |
| Touch vs. hover | Hover-only affordances (custom cursors, hover-reveal captions, hover-triggered menus) don't fire — or "stick" after a tap — on touch devices with no guard. | `@media (hover: hover) and (pointer: fine) { /* hover-only enhancement */ }`, and make sure every hover-gated action has a tap-equivalent |
| High-refresh displays (120Hz+ ProMotion, 144Hz monitors) | Animations driven by `requestAnimationFrame` correctly scale to the display's real refresh rate — but any hand-tuned animation using a fixed frame-count or `setTimeout` interval assuming 60fps will run visibly too fast on 120Hz+. | Always animate by **elapsed delta-time** (ms since last frame), never by frame count, in custom canvas/WebGL render loops |
| Reduced data | `prefers-reduced-data: reduce` exists but is Chrome/Android-only as of 2026 — real coverage is thin. | Treat it as a bonus, not a primary strategy; the `Save-Data` request header (server-side) has broader real-world reach if you control the backend |

---

## 6. Submission-grade checklist

Ordered, checkable, brutally concrete. Run top to bottom before calling a landing page finished.

### Content
- [ ] 1. Every headline/CTA is real copy, not lorem ipsum or placeholder brackets
- [ ] 2. No "Client Name" / "TODO" / `#` placeholder links anywhere, including footer legal links
- [ ] 3. Every image has final art, not a stock placeholder unless stock is the actual final choice
- [ ] 4. Contact info / email / social links go to real, live destinations (click every one)
- [ ] 5. Legal pages exist and are linked if the site collects any data (privacy policy, cookie notice)
- [ ] 6. Copy has been proofread by a second person or spell-checker in every language shipped

### Typography
- [ ] 7. All custom fonts have `font-display: swap` (or `optional`) so text isn't invisible during load
- [ ] 8. A system-font fallback stack is specified for every `font-family` (no bare custom font name)
- [ ] 9. Line length for body text is roughly 45–75 characters per line at default viewport
- [ ] 10. No orphaned single words on their own line in headlines (manual `<br>`/`text-wrap: balance` pass)
- [ ] 11. Font weights actually loaded match font weights actually used (no browser-faux-bolding a weight you never imported)
- [ ] 12. Cyrillic (RU/UK) glyphs render correctly in every custom font used for RU/UK copy — verify, don't assume a Western font has full Cyrillic coverage

### Layout / responsive
- [ ] 13. Tested at 320px, 375px, 768px, 1024px, 1440px, 1920px, and one ultra-wide (2560px+) width
- [ ] 14. No horizontal scrollbar at any width (the one exception: an intentionally `overflow-x:auto` gallery/table, in its own container)
- [ ] 15. Touch targets ≥ 24×24 CSS px (WCAG 2.5.8), with real spacing between adjacent tappable elements
- [ ] 16. Hero section is legible and complete without scrolling on a 375×667 (iPhone SE-class) viewport
- [ ] 17. `100dvh`/`100vh` hero doesn't clip content when the iOS address bar is showing (test on a real device or accurate simulator, not just DevTools device toolbar)
- [ ] 18. Images use `width`/`height` or `aspect-ratio` so layout doesn't shift while they load
- [ ] 19. Nothing depends on a fixed pixel viewport — test browser zoom to 200% (WCAG 1.4.4 Resize Text)

### Color / contrast
- [ ] 20. Body text ≥ 4.5:1 contrast against its background (WCAG 1.4.3)
- [ ] 21. Large text (≥24px regular / ≥18.66px bold) ≥ 3:1 contrast
- [ ] 22. UI components (buttons, input borders, icons that convey meaning) ≥ 3:1 non-text contrast (WCAG 1.4.11)
- [ ] 23. Text over photos/video checked against the *actual busiest region* of the image, on a real rendered screenshot, not just the DOM color value
- [ ] 24. Dark-mode variant (if offered) independently passes the same contrast checks — colors don't automatically stay compliant when inverted
- [ ] 25. Color is never the only signal for state (error/success/required-field) — pair with an icon or text label
- [ ] 26. Link text is distinguishable from body text by more than color alone (underline, weight)

### Motion
- [ ] 27. `prefers-reduced-motion: reduce` is implemented and actually tested (OS-level toggle, not assumed)
- [ ] 28. Parallax stops or reduces to near-zero under reduced motion
- [ ] 29. Autoplaying background video/animation freezes to a static frame under reduced motion
- [ ] 30. Scroll-hijacking (if present) falls back to native scroll under reduced motion
- [ ] 31. No flashing content exceeds 3 flashes per second anywhere (seizure risk, WCAG 2.3.1)
- [ ] 32. Auto-advancing carousels/sliders have a pause control and stop under reduced motion
- [ ] 33. Entrance/scroll-triggered animations don't re-trigger annoyingly on every re-scroll into view (check scroll up and back down)
- [ ] 34. Cursor-follower / custom cursor effects have a non-hover, touch-safe fallback

### 3D / performance
- [ ] 35. LCP element paints without waiting on WebGL/3D bundle initialization
- [ ] 36. Lighthouse (or PSI post-deploy) LCP is under ~2.5s on Mobile/Slow 4G throttle, or you have a documented reason it isn't
- [ ] 37. Lighthouse TBT is under ~200ms on mobile CPU throttle (proxy for real-world INP headroom)
- [ ] 38. CLS stays under 0.1 through the full page lifecycle, including late-loading images/ads/fonts
- [ ] 39. Canvas/WebGL container has explicit dimensions before the canvas mounts (no layout shift on 3D init)
- [ ] 40. `devicePixelRatio` used for 3D rendering is capped on mobile (not blindly rendering at full 3× retina)
- [ ] 41. Shader compilation/texture upload is not happening synchronously inside a click handler
- [ ] 42. WebGL context-loss (`webglcontextlost`/`webglcontextrestored`) is handled, not just assumed away
- [ ] 43. Total page weight and third-party script count have been reviewed — every analytics/chat/font/script tag is intentional, not copy-pasted cruft
- [ ] 44. Animate `transform`/`opacity`, not `top`/`left`/`width`/`height`/`box-shadow`, for anything running every frame

### Accessibility
- [ ] 45. Full keyboard pass: Tab through the entire page, Shift+Tab back, nothing is unreachable or trapped
- [ ] 46. Visible focus indicator on every interactive element (no blanket `outline: none`)
- [ ] 47. Skip-to-content link present and functional as the first Tab stop
- [ ] 48. One `<h1>`, logical heading order, real landmarks (`header`/`nav`/`main`/`footer`)
- [ ] 49. Every meaningful image has real `alt` text; decorative images have `alt=""`
- [ ] 50. Decorative canvas/WebGL has `aria-hidden="true"`
- [ ] 51. Split-text/per-character animated headlines have a clean `aria-label` on the container and `aria-hidden="true"` on the fragmented spans
- [ ] 52. Forms have associated `<label>`s (not placeholder-as-label), and error messages are programmatically associated with their field
- [ ] 53. axe-core run produces zero critical/serious violations
- [ ] 54. Tested with an actual screen reader (VoiceOver or NVDA) on the hero and primary CTA flow, not just automated tooling
- [ ] 55. Page is usable and makes sense with CSS fully disabled (source-order sanity check)
- [ ] 56. `lang` attribute set correctly on `<html>` (and on any inline foreign-language spans, e.g. English brand name inside RU copy)

### SEO / meta
- [ ] 57. Unique, correctly-lengthed `<title>` and meta description on every page
- [ ] 58. Self-referencing absolute canonical URL on every indexable page
- [ ] 59. OG + Twitter card tags present with a real 1200×630 image (test the actual rendered preview, not just the tags — see polish section)
- [ ] 60. Full favicon/app-icon set present (svg, ico, apple-touch-icon 180×180, 192/512 PNG, maskable, manifest)
- [ ] 61. `theme-color` set for both light and dark
- [ ] 62. JSON-LD Organization + WebSite present and validated in Google's Rich Results Test
- [ ] 63. `robots.txt` and `sitemap.xml` exist, are linked to each other, and contain only real indexable URLs
- [ ] 64. hreflang tags present and fully reciprocal across every language version, with `x-default`
- [ ] 65. No accidental `noindex`/staging robots meta tag left in from a dev environment

### Cross-browser
- [ ] 66. Verified on a real iPhone in Safari, not just a device-emulation panel — hero height, video autoplay, backdrop-filter
- [ ] 67. Verified on Android Chrome — different address-bar behavior and font rendering than iOS
- [ ] 68. `-webkit-` prefixes present for `backdrop-filter`, `appearance`, `line-clamp` where used
- [ ] 69. `mix-blend-mode` effects specifically re-checked on real Safari while scrolling
- [ ] 70. Touch-only devices don't show "stuck" hover states after a tap
- [ ] 71. Tested with a slow/throttled connection at least once end-to-end (not just on localhost/fiber)
- [ ] 72. Tested on a high-refresh-rate display if any hand-timed (non-rAF-delta) animation exists in the codebase
- [ ] 73. Firefox pass — the browser most likely to reveal a Chromium-only CSS/JS assumption after Safari

### Polish details
- [ ] 74. Custom favicon shows correctly in a real browser tab (not the default globe/blank icon)
- [ ] 75. Custom 404 page exists and matches the site's design, with a way back to the homepage
- [ ] 76. Text selection color (`::selection`) is styled intentionally, not left as default blue-on-white against a dark design
- [ ] 77. Scrollbar styling (if customized) degrades gracefully on browsers that don't support the custom scrollbar CSS
- [ ] 78. Focus ring color/style is intentional, not the raw browser default clashing with the design
- [ ] 79. Cursor is set intentionally on every interactive element (`pointer` on clickables, custom cursor states match actual affordance)
- [ ] 80. OG image preview actually checked in a real share-preview tool (e.g. paste the URL into a Slack/Telegram message to yourself, or a dedicated OG preview checker) — tags being *present* isn't the same as the preview looking right
- [ ] 81. Print stylesheet reviewed (`--print-to-pdf` the real page) — at minimum, navigation/decorative canvas/video don't dump garbage onto a printed page
- [ ] 82. Favicon/app-icons checked on an actual iOS/Android home-screen add, not just the manifest file existing
- [ ] 83. Page title in the browser tab is the actual page title, not "localhost" or a CMS default leftover
- [ ] 84. Console is clean of errors/warnings on load and through the primary interaction flow
- [ ] 85. Right-click / long-press context menu wasn't accidentally disabled site-wide (a real usability complaint on some award-style sites, and an accessibility problem)
- [ ] 86. Any custom scrollbar/cursor/selection styling still works correctly after browser zoom to 150–200%

---

## Sources consulted

Direct fetches (primary/authoritative): web.dev (`/articles/inp`, `/articles/lcp`, `/articles/cls`),
developer.chrome.com (`/docs/automation-and-testing/headless-cli`, `/docs/lighthouse/performance/performance-scoring`),
chromedevtools.github.io/devtools-protocol, w3.org (WCAG 2.2 quickref and "New in WCAG 2.2"),
developer.mozilla.org (`prefers-reduced-motion`, viewport units `dvh`/`svh`/`lvh`, `@media (hover)`,
`rel=canonical`), awwwards.com/about-evaluation, ogp.me, developers.google.com (favicon guidance,
structured data intro, PageSpeed Insights API get-started), caniuse.com (`backdrop-filter`),
webkit.org (iOS video autoplay policy blog post), bugs.webkit.org (`mix-blend-mode` bug list),
github.com/ChromeDevTools/chrome-devtools-mcp, unlighthouse.dev.

Plus **live experimentation** against real Chrome 152 headless=new on this machine (Windows 11 +
WSL2, `/mnt/c/Program Files/Google/Chrome/Application/chrome.exe`) — every claim marked
**[verified live]** in §2 was produced by actually running the code in this document, not inferred
from documentation.

Note on search budget: this session's shared WebSearch allowance (200 calls) was already exhausted
by other activity before this task reached its own search quota; 6 WebSearch queries completed
before the cap hit. Compensated with ~20 direct WebFetch calls to primary sources plus hands-on
verification, per the sourcing above.
