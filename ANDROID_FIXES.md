# DigitAlaya — Android browser fixes

Source of truth (kept identical):
- `/workspace/digital-india/index.html`
- `/workspace/digital-india/BalBharatiPublicSchoolRohini_NityantGupta.html`
- Deploy mirrors: `/workspace/digital-india-deploy/index.html` + `BalBharati…html`

Scope: Android Chrome / Samsung Internet / HuaweiBrowser (and other small-screen Continue-anyway paths). No essay/CONFIG redesign. Desktop + iOS continue paths preserved. Device gate still shows first; Continue anyway still available.

## Bugs found (Android after Continue anyway)

1. **Stuck invisible content (major)**  
   On first visit the gate sets main content to `display:none`. Reveal / counter / explored / pause IntersectionObservers and the scroll layout cache then initialize against zero-size boxes. After Continue anyway, Android Chrome often never re-delivers IO for those targets → `.reveal` stayed at `opacity:0`, counters silent, UPI/mission scroll math wrong, animations stuck paused.

2. **No wake on unlock**  
   `digitalaya-device-continued` did not refresh scroll cache, pause-IO, or reveals. Session revisit was OK (content visible before JS); first Continue click was broken.

3. **`color-mix()` without fallbacks**  
   Older Chromium (some HuaweiBrowser / older Samsung) drops the whole declaration if `color-mix` is unknown → warning boxes / focus rings / ghost button borders could go unreadable or borderless.

4. **`overflow-x:clip` only**  
   Engines without `clip` need `overflow-x:hidden` as a prior fallback (sticky still OK on Blink).

5. **Heavy GPU effects on phones**  
   Infinite button backlight, 48px mesh blur, UPI `drop-shadow`, and `will-change` caused jank / “missing” animations when Chrome throttled compositing.

6. **Hover-only feedback**  
   Spotlight cards and quiz option chrome leaned on `:hover`; coarse Android needed `:active` / broader touch flip detection (`hover:none`, touch points on narrow viewports).

7. **Tabs / pledge inputs**  
   Missing `-webkit-appearance:none` on tabs; pledge `<input>` inherited &lt;16px on some layouts → Android focus zoom.

8. **Tight IO thresholds**  
   Reveal `threshold:0.08` / counters `0.4` / explored `0.35` were brittle after unlock and on short Android viewports with dynamic toolbars.

## Fixes applied

1. **`__digitalayaWake` hub**  
   Registers hooks; runs on Continue, `digitalaya-device-continued`, and session-continued boot. Adds `android-wake`, re-fires `resize`, and re-runs hooks on rAF + 120ms + 500ms.

2. **Scroll engine**  
   Exposes refresh; listens to `visualViewport` resize/scroll; registers wake → `cacheLayout()` + frame.

3. **Reveals**  
   Lower threshold / wider `rootMargin`; `forceNearViewport` + re-observe on wake; `.android-wake` CSS forces opacity/transform so nothing stays frozen half-hidden after continue.

4. **Pause animations**  
   Wake re-observes targets and `refreshAll()` so marquee / UPI dash / mesh / tab pulse resume when in view.

5. **Counters + explored**  
   Softer thresholds; mark counted; wake re-observes.

6. **CSS hardening**  
   - `overflow-x:hidden` then `clip`; `@supports not (overflow-x:clip)` fallback  
   - `min-height:100vh` then `100dvh`  
   - Solid `rgba` before every critical `color-mix`  
   - Narrow/coarse: lighter mesh blur, no button backlight loops, no UPI drop-shadow, drop progress `will-change`  
   - Tabs `-webkit-appearance:none`; pledge input `font-size:16px`; spot `:active`  
   - Short-screen flip min-height; hide tab scrollbar junk; promote nav/UPI stage layers

7. **Flips**  
   Treat `(hover:none)`, touch points on ≤1279px, and `ontouchstart` as touch → “Tap to flip” + click toggle (not hover-only).

8. **Gate**  
   Unchanged policy: phones/tablets/Harmony/KaiOS still gated; Continue anyway still sets `sessionStorage['digitalaya-device-continue']`.

## Tests run (headless Chrome)

| Case | Result |
|------|--------|
| Pixel Android UA 390×844 — gate then Continue | Main visible; 0 in-view reveals at opacity 0; `android-wake` set |
| Menu / theme / stack tabs / flip / quiz / pledge | All operable after continue |
| Samsung Internet UA / HuaweiBrowser UA / KaiOS UA | Gate → continue → main block, no stuck opacity |
| iPhone UA continue | Same unlock path OK |
| Desktop 1440 (real fine pointer) | Unchanged intent; headless lacks `pointer:fine` so gate appears in CI only |

## Residual limits on small Android screens

- Sticky pillar stack / desktop UPI sticky stage are desktop layouts; phone uses stacked / capped diagram (`max-height:30vh`).
- `animation-timeline: view()` still absent on many Android Chrome versions; JS fallbacks cover journey stroke + pillar scale when wide.
- Reduced-motion / battery saver still disables decorative loops (by design).
- Mesh / network motion is lighter, not identical to desktop GPU look.
- Very old WebViews without `:is()` / `gap` / `aspect-ratio` are out of scope.
- Full desktop experience still needs ~1280px+ with fine pointer; Continue anyway remains a best-effort mode.

## Not changed

- Essay copy, CONFIG credits (Nityant Gupta, Class 10, Bal Bharati Public School, Rohini)
- Device-gate whitelist logic for real desktops
- Motion-gate large-screen flow (still skipped when device-continued on narrow)
