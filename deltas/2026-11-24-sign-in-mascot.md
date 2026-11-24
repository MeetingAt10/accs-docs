# Sign-in: animated mascots and landing touches

Status: built and verified 2026-11-25, merged into dev the same day (accs-frontend #115), folded into [pages/01-shell-and-auth.md](../pages/01-shell-and-auth.md) P02.

Touches: [pages/01-shell-and-auth.md](../pages/01-shell-and-auth.md), P02 "Built sign-in and password reset". Before this delta that section still described the older `DoorPlate` panel; `origin/dev` already ships the Figma "User Login" layout (`-login-shell.tsx`, `login.css`, static `lion.svg`). This delta starts from the shipped layout.

## Decisions (user, 2026-11-24)

- Keep the Figma layout (navy brand panel left, form right; navy strip above the form below 960 px). Borrow details from the access-ccs.com landing, no full restyle.
- Both mascots: the lion stays where the static lion is; the cub stands beside it.
- Idle animation plus reactions to the form.

## Changes

1. **Brand panel sky.** The navy gradient becomes the landing hero's night sky (`#040824` to `#0B1A6B` to `#2A4BF0` at the foot), with the landing starfield twinkling at the top. The faint building photo stays. Headline in Poppins 600 (landing display face).
2. **Mascots.** The static `lion.svg` is replaced by the lion sprite from `accs-landing/public/landing/mascot` (same spot, same -10.55° tilt, same size). The cub sprite stands to its left, unrotated, about 55% of the lion's width, from 960 px only (the narrow strip keeps the lion alone). Both are decorative (`aria-hidden`).
3. **Idle.** Each mascot breathes; every 4 to 9 s it plays one blink, look-around or sway, then breathes again. Lion and cub run on separate timers. After 45 s with no pointer or key input both doze; the next input wakes them (surprised for 0.8 s).
4. **Reactions.**

   | Trigger | Lion | Cub |
   |---|---|---|
   | Page load | wave (one cycle) | wave, 0.4 s later |
   | Focus in the email/ID field | point (toward the form) | happy |
   | Focus in a hidden password field | sleepy (eyes shut, not peeking) | sleepy |
   | Password shown with the eye button | wink | wink |
   | Request in flight | thinking | thinking |
   | Wrong details or any error | sad for 2.5 s | sad for 2.5 s |
   | Signed in | cheer | love |
   | Forgot-password view | thinking | thinking |
   | Reset-code view | document | document |

5. **Form touches.** "Sign In" title in Poppins; the submit button takes the landing CTA shape (12 px radius, 600 weight, lift and shadow on hover, press scale).
6. **Assets.** WebP strips converted from the landing PNGs (lion at 2x, cub at 1x; about 1.6 MB and 0.75 MB for all loops and stills). Breathe and wave load first; the other loops load after first paint and a mascot only switches to a loop once its image has loaded, so it never blanks.
7. **Reduced motion.** With `prefers-reduced-motion`, no loop plays (frame 1, the rest pose); reactions still switch the still pose, which is a state change, not motion. Stars do not twinkle.

## Success criteria

- 1440×900: panel shows the sky, stars, the lion at its old position and the cub beside it; form position unchanged. 390×844: navy strip with the lion only, no horizontal scroll.
- Idle: the breathe loop animates; another loop plays within 10 s.
- Reactions, checked in a real browser: focus email/ID → lion `point`; focus password → `sleepy`; show password → `wink`; wrong password → `sad`; correct password → `cheer` and the redirect still happens.
- Reduced motion: no running animation on the mascots or stars.
- `npm run build` (vite build and `tsc -b`) and `npm test` pass; existing sign-in tests unchanged. The frontend has no lint script.

## Verification (2026-11-25)

Production build served by `vite preview` with the dev API proxied, headless Chromium:

- Load: lion and cub `wave`, then `breathe` (background position advancing); `look-around` or `sway` seen within 10 s.
- Focus email/ID `point`/`happy`; password `sleepy`; show `wink`; hide `sleepy`; submit `thinking`; wrong password `sad`, back to idle after 2.5 s; forgot view `thinking`; correct password `cheer`/`love`, then redirect to `/home`.
- 47 s idle `doze`; mouse move `surprised`, then `breathe`.
- 390x844: lion shown, cub hidden, no horizontal scroll. Reduced motion: `animation-name: none` on both sprites and the stars; poses still switch.
- Poppins 600 loaded for the title and headline.

## Revision 2026-11-25: smoother idle, responsive stage

Request: smoother idle animation; check responsiveness.

- **Smoothness.** The sprite loops (8 to 12 fps) stepped visibly. Body motion moved to CSS transforms from the feet (`.mascot` lean/sway/nod/reactions, `.mascot__breath` breathing), which the browser interpolates every display frame; checked: the breath layer's transform changed on 61 of 61 frames in 1 s. Sprites now only swap faces (blinks of 45/85/55 ms, eye glances with a half step), from a new faces sheet plus a 24 fps wave strip built by `scripts/sign-in-mascot.mjs` from the landing rig (no body motion in any frame). The old per-loop strips are gone; sprite weight fell from about 2.4 MB to 1.06 MB.
- **Idle beats** (JS timeline in `-mascot.tsx`): every 2.2 to 5.2 s, a blink (50%, sometimes double), a look around with a lean (32%), or a 4.8 s sway (18%). Lion and cub run independently and breathe out of step. The greeting wave waits for its strip (up to 5 s).
- **Reactions** gained body language: lean toward the form while typing, shrink while the password is hidden, slump on error, a ponder rock while busy, two hops on success, a startle on waking.
- **Responsive.** Swept 320x568 to 2560x1440 (16 sizes, including 844x390 landscape and 960x600). Fixed: at 960 to 1100 px wide the cub covered the facts and fell off the panel's left edge, and at short heights the lion's fur ran under the facts. From 960 px the panel is a flex column and the mascots stand in a stage under the copy sized `min(440.59px, 64cqw, 110cqh)`; the logo scales with height (`clamp(200px, 34dvh, 311px)`); a short stage drops the cub (under 190 px) and then the lion (under 120 px). The cub now also shows in the 600 to 959 px strip. No horizontal scroll and the form fits at every size.
- Verified again on the production build: all reactions, redirect to `/home`, doze after 47 s and wake, reduced motion (no animation, no transform), `npm run build` and `npm test` (197) pass.

## Revision 2026-11-25: mascots peek over the form

Request: reposition the mascots; chosen option "peek over the form" (user, 2026-11-25).

- The form column wraps the form in `.lg-card` (20 px radius, layered shadow, `--accs-surface`). `.lg-perch` above it holds the cub and the lion, right of centre; it overlaps the card by `--sink` (34 px, 38 px from 960 px) and the card paints over it, so about 100 px (heads and shoulders) shows. Lion 132 px wide, 150 px from 960 px; cub at 74% of the lion.
- The brand panel no longer holds mascots: the stage, its container queries and the short-screen logo scaling are gone; the panel is back to the frame's values (311 px logo, 48 px bottom padding); the narrow strip no longer reserves room on the right.
- Hidden password: both duck behind the card (`translateY(34%)`) with eyes shut, the zzz just showing.
- They rise from behind the card on arrival (lion 0.2 s, cub 0.35 s), then wave.
- Verified on the production build: every reaction, redirect to `/home`, reduced motion; 11 sizes from 320x568 to 1920x1080 with no horizontal scroll, the card inside the viewport and both mascots within the card's width.
- Dark mode checked (1440x900, 390x844): the card is opaque (`--accs-surface`) so the lower bodies stay hidden, and its ring now uses `--accs-line` so the edge shows on the dark page. "Forgot Password?" used the hardcoded brand navy (`--lg-navy`) and was unreadable on dark; it now uses `--lg-accent` (navy in light, `--accs-link` in dark). The perch top stays on the page at short heights (960x600, 1280x720, 844x390).
- The lion's `point` pose and the typing lean still aim right, past the card's edge, now that the form is below them; swap `typing` to `happy` if it reads oddly.
