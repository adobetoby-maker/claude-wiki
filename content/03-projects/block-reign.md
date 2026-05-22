@AGENTS.md

## VISUAL REVIEW — NON-NEGOTIABLE

Every iteration MUST produce BOTH PNG screenshots AND a scroll video. Static-only review is forbidden — the GLSL shader, Framer Motion scroll beats, the marquee ticker, and all scroll-driven animation can only be evaluated in video.

Two forbidden failure modes: **(a) code-as-proxy scoring** — scoring from what the code intends rather than what is visible; **(b) static-only review** — missing motion regressions because the scroll video was skipped.

### Canonical failure — iter-16

Code lifted the GLSL shader to fullpage fixed position. Score reported +0.25. Toby opened the deploy and saw a regression.

What the screenshots actually showed:
- Compliance section (0.52 alpha): animated node grid competed directly with ghost "COMPLIANT" type, destroying the page's strongest editorial moment
- Mobile: shader hidden but backgrounds still transparent → flat dark blob with no section rhythm
- Hero (0.12 alpha): node network was the background, not a texture; white text on animated noise has less authority than white text on controlled dark navy

Iter-18 repaired this with zone-based alpha (compliance 0.52 → 0.97, hero 0.12 → 0.82, all dense sections 0.95+). Score returned to 6.25 — exactly iter-15. The +0.25 was an illusion scored from intent, not pixels.

iter-16 also failed because no one watched the scroll video, only inspected static PNGs. The video would have shown the shader fighting with the compliance ghost type in motion, not just in a frame.

### Canonical failure — iter-19 (home page: incomplete scroll + single-viewport blindness)

Element collisions near footer zone shipped undetected. The harness scroll stopped short of the true bottom — `document.body.scrollHeight` steps without an explicit scroll-to-bottom. Toby caught it manually. Single-viewport (1440 only) also missed large-screen layout. See `scripts/iterations/ERROR-LOG.md`.

### Required protocol for every iteration

1. **Harness MUST capture all 4 viewports:** 375×812 @2x (mobile), 1440×900 @1x (desktop), 2560×1440 @1x (4K), 2560×1440 @2x (5K). Missing viewport = INCOMPLETE.
2. **Scroll MUST reach true bottom:** `document.documentElement.scrollHeight - window.innerHeight`. End every scroll sequence with an explicit `scrollTo(0, scrollMax)`. **Footer MUST be visible in the final video frame** — if not, the harness is broken, fix it before scoring.
3. **coverage.md MUST be generated** per iteration: viewports captured, scroll max, footer-verified status, frame count. Missing = INCOMPLETE.
4. **Overlap gate:** Visible element collisions at any viewport = FAIL. Do not score as pass.
5. Read all 4 viewport PNGs — `Read('scripts/iterations/iter-NN/blockreign-{mobile,desktop,4k,5k}.png')` — describe what is visible section by section.
6. Extract ffmpeg frames **including final 3 frames** to verify footer: `ffmpeg -i blockreign-scroll.webm -vf fps=2 /tmp/frames/frame%03d.png`. Read ≥6 frames spread across full timeline.
7. Compare PNGs + video against `iter-(NN-1)/` and `iter-01/palantir-desktop.png`, `iter-01/veeva-desktop.png`, `iter-01/servicenow-desktop.png`.
8. Each score dimension requires TWO quoted observations — one from PNG, one from video. Motion presence, Depth + lighting, Hero impact (animated) → video is dominant.
9. Any regression at any viewport → score goes down. No exceptions.
10. All viewports carry equal weight. Large-viewport issues are not optional findings.

Full rule: `~/.claude/rules/visual-review-non-negotiable.md`
Incident log: `scripts/iterations/ERROR-LOG.md`
