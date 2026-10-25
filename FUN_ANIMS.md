# DigitAlaya — fun entrance animations

Source of truth (kept identical):
- `/workspace/digital-india/index.html`
- `/workspace/digital-india/BalBharatiPublicSchoolRohini_NityantGupta.html`
- Deploy mirrors: `/workspace/digital-india-deploy/` (same filenames)

## Classes

| Class | Effect |
|-------|--------|
| `.reveal` | Base: fade + slight rise/scale → final |
| `.reveal--pop` | Scale ~0.88 → 1 |
| `.reveal--rise` | TranslateY ~28px → 0 |
| `.reveal--tilt` | Slight rotate + scale (desktop); phones drop rotate |
| `.reveal--left` / `--right` | Slide from side |
| `.reveal--soft` | Gentler rise, longer duration |
| `.reveal--fade` | Opacity only (sticky UPI stage, hero art) |
| `.reveal-stagger` | Children enter with uneven delays (~95–140ms organic) |

Grid children also get alternating transforms via `:nth-child(6n+…)` so a bento/pillar/services row does not feel samey.

Cards / spots / stats / milestones / metrics / flips / svc tiles get a soft saffron→green **border flash** (`cardEnterFlash`) when `.is-in` lands.

Tab panels use `panelPop`; metrics inside an active panel use staggered `metricPop`.

## Phone / Android wake

1. Device gate still first; motion gate after **Continue anyway**.
2. `__digitalayaWake` → `wakeReveals` → `forceNearViewport` marks near items with **stagger** (CSS transitions play — no instant snap).
3. IO: thresholds `[0, 0.12, 0.22, 0.35]`, `rootMargin: 0px 0px -8% 0px`.
4. Re-observe after unlock for anything still pending.
5. Safety: after ~2.4s, only **near-viewport** leftover hidden nodes get `.is-in` + `.reveal-force` (below-fold scroll reveals stay intact).
6. Sticky pillars / UPI stage: **opacity-only** (transform would break `position:sticky`).

## Reduced motion

`prefers-reduced-motion: reduce` (and existing `reduceMotion` JS): all reveals / panel pops / card flashes snap to final with no transition.

## What was wired

Hero (eyebrow, title, lead, CTAs, stats, art), mission, journey header + track, milestones (existing stagger), pillars, UPI header + fade stage + step cards, stack header/tabs/panels, UPI compare bars + card, impact header + bento, services track + nav, toolkit flips + quiz shell, vision header + cards + timeline items, pledge header + form, sources list, footer blocks.

## Kept intact

Counters (~4–7s calm), UPI bar JS grow, sticky UPI demo, marquee fixes, device + motion gates, accessibility focus, no new stats, no per-letter splits, no heavy blur on many items.

## Risks

- More simultaneous transitions on low-end Androids → mitigated with opacity-only sticky bits, no rotate on coarse pointers, will-change cleared after `.is-in`, 2.4s force safety.
- Nested `.reveal-stagger` (hero content → hero stats) relies on recursive `markIn`.
- Horizontal `.services` stagger reveals off-screen cards when the track enters — intentional.
