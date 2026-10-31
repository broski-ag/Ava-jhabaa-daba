# Portfolio — how it works & where to tweak

```
npm install      # once
npm run dev      # develop
npm run build    # production build -> dist/   (deploy the dist folder)
npm run typecheck && npm run lint
```

## The scroll experience (`src/hooks/useScrollStage.ts`, `src/index.css`)
Hero is `fixed`. The portfolio panel is a normal block that the page's own scroll lifts over it —
nothing moves it by a pixel value, so it behaves the same on every screen. After the panel there is a
closing "runway" (`.contact-runway`, one screen + 7rem) that lets it lift away and uncover the fixed
contact section behind it. Once the panel fully covers the hero the hero is switched off and the contact
section switched on (`data-covered` on `.stage`).
* Parallax strength → `PARALLAX` at the top of `useScrollStage.ts` (`vh: 0.12` = 12 % of screen height).
* Parallax follow-speed → `FOLLOW_SECONDS` (lower = tighter to the scroll).
* On very short screens the contact block scales down to fit (`ContactSection.tsx`).

## Which hero composition a screen gets (`src/utils/layout.ts`)
Decided by aspect ratio, not just width: portrait (9:16 and taller, or ≥960 px wide and taller than 0.72)
→ the portrait composition; everything else → the landscape one, which morphs towards its "tablet" state
on narrow portrait-ish windows (`useHeroMorph.ts`). **`index.html` repeats this rule in its preload /
`<picture>` media queries and `index.css` in one `@media` — change all three together.**

## Videos (`src/utils/playbackManager.ts`, `src/components/VideoTile.tsx`)
Every tile always shows its thumbnail. Only the few tiles nearest the middle of the screen play
(`playBudget()`: 6 desktop / 4 tablet / 3 phone, fewer on low-end or data-saver devices); the next few
pre-buffer; everything else holds no decoder and no data. Raise the numbers in `playBudget()` if you
prefer more simultaneous motion.
* Add / reorder videos and thumbnails → `src/data/content.ts` (`thumb` = number of the file in `public/thumbnails`).
* **Biggest remaining speed-up:** run `scripts/make-previews.sh <folder-of-videos>` (needs ffmpeg), upload the
  small `previews/` clips and the `faststart/` versions next to your videos, then add
  `preview: '<url>'` to each item in `content.ts`. Grid tiles then play a ~150 KB clip; the full video is
  only used for fullscreen and the showreel.
* Videos are served from jsDelivr's GitHub mirror, which is built for code, not video, and is the slowest
  link left. Cloudflare R2 / Bunny / any object storage with a CDN will start them noticeably faster.

## Assets
`public/pc` (landscape hero), `public/mobile` (portrait hero), `public/hero/*.html` (the two hero text
designs, fetched only for the composition in use), `public/thumbnails`. `public/_redirects` is kept here
so Netlify's fallback rule survives rebuilds.
