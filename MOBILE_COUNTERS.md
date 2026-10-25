# DigitAlaya — mobile counters, marquee, UPI bars

Source of truth (kept identical):
- `/workspace/digital-india/index.html`
- `/workspace/digital-india/BalBharatiPublicSchoolRohini_NityantGupta.html`
- Deploy mirrors: `/workspace/digital-india-deploy/` (same filenames)

## Problems fixed

1. **Language marquee frozen on phones**
   - Sticky `:hover` pause on touch → wrapped pause rule in `@media (hover:hover) and (pointer:fine)`.
   - Keyframes use `translate3d(-50%,0,0)` for Android GPU compositing.
   - Coarse/narrow lite CSS does **not** disable `.bharat__track`.
   - `PAUSE_ANIMS` wake: clear `is-paused`, force `animationPlayState=running` when `.bharat` in view; on first `continued` unlock, restart animation (`animation:none` → reflow → restore).

2. **Number figures static / stuck at final HTML / stay at 0 after Continue**
   - Converted stack metrics, DigiLocker/ABDM figures, ONDC numeric tiles, and all `#impact` bento figures to `[data-count]` starting at `0`.
   - Supports `data-prefix`, `data-suffix`, `data-locale`, `data-decimals`.
   - COUNTERS wake **force-runs** not-yet-counted elements that intersect via `getBoundingClientRect` (not only reobserve).
   - Delayed passes at 0 / 100 / 400 / 1000 ms after continue/allowed.
   - `prefers-reduced-motion`: snap to final once visible (no tween).

3. **UPI `.bar--now i` stuck full / never grows**
   - CSS still starts `.bar>i` at `width:0`; `.bar--then>i` stays 8%.
   - `__digitalayaAnimateUpiBar`: reset to 0, reflow, then set `data-w`%.
   - Called from counters wake, reveal wake, and IO when bar enters view.

4. **UPI how-it-works demo scrolls away on phones**
   - Narrow screens previously forced `.upi-stage{position:relative}` so the Payer→Payee animation left the viewport before steps 2–4 activated.
   - Fix: keep animation-first order; on `max-width:899px` use `position:sticky` under the fixed/sticky nav (`top: calc(var(--nav-h) + .5rem)`, `z-index:45`), compact SVG height, caption stays with the stage; step cards scroll below with bottom margin on the last step so later steps can still become active.
   - Desktop (≥900px) two-column sticky layout unchanged.
   - Narrow scroll probe uses ~62% viewport (below sticky stage) so steps 2–4 still advance; last step has extra scroll runway.
   - Tab switches wake counters so DigiLocker/ABDM metrics count up when their panel is shown.

## Counters added / converted (start text `0`)

| Location | count | prefix | suffix | notes |
|----------|-------|--------|--------|-------|
| Hero (existing) | 24162, 850, 93 | — | Cr / Cr+ | unchanged start-at-0 |
| Stack UPI | 24162, 314, 703, 13000 | ~₹ / ~ | × | locale on 24162 & 13000 |
| DigiLocker | 70.69, 850, 144 | > | Cr | decimals on 70.69 |
| ONDC | 5, 1000 | ~ | lakh+ / — | skipped vague `Open` |
| ABDM | 93, 104, 48 | > | Cr | |
| Impact bento | 144, 59.09, 1.11, 2.15, 6.5, 18.4, 9.20, 12 | > / ~ / >₹ / ~₹ | Cr / Bn / Lakh / Lakh Cr | decimals where needed |

Essay paragraph prose in milestones / pillars / sources was **not** changed.

## Verify (manual / smoke)

After Continue anyway on Android UA: marquee track `getAnimations()[0].playState === "running"` when `.bharat` in view; hero/stack counters tween from 0; scrolling to UPI compare grows the tricolor bar from 0→100%.
