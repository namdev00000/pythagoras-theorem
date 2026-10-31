# Pythagoras' Theorem: an interactive page

A single scrolling web page that teaches Pythagoras' theorem (a² + b² = c²). It opens on a bold formula over crushed paper, then moves through the idea, a visual proof, the history, a calculator, six real-life examples and a closing note by Namdev.

Built by Namdev with Claude Sonnet 5.5.

Published as a private Claude artifact. Source files live next to this README:

```
pythagoras.html     the whole page: markup, CSS and JavaScript
site/bg.jpg         black crushed paper (hero background, dark theme)
site/bg-light.jpg   white crushed paper (hero background, white theme)
README.md           this file
```

---

## 1. How we iterate

We worked in small rounds. Each round was one request from Namdev, one change from Claude, and one republish to the same link, so the URL never changed.

| Round | Request | What changed |
|---|---|---|
| 1 | Build the page: grid-pattern hero with the theorem in bold, then introduction, history, calculator, at least five real-life examples with visuals, and a closing note by Namdev | Whole page built from scratch (version 1) |
| 2 | Use the black crushed-paper image on the front, remove the blue dark theme, make dark pure black, add icons (white in dark, dark in white) | Neutral light and pure-black dark tokens, image hero, icon set (version 2) |
| 3 | Use the white crushed paper for the white theme, and scatter π, +, −, ÷, %, = around the title and formula | Theme-aware hero image, about 22 floating symbols (version 3) |
| 4 | White paper only on the front page, like the black theme | Removed the page-wide paper background (version 4) |
| 5 | Always open in the white theme | Default theme set to light on load (version 5) |

Working rules we followed:

- **One change per round.** Each request was applied on its own and nothing else was touched. When Namdev said "don't make any changes", only the one requested behaviour changed.
- **Same file, same link.** Every round edits `pythagoras.html` and republishes to the same artifact URL. Small edits are sent as exact find-and-replace pairs.
- **Images are prepared first.** The uploaded paper images are resized to at most 1920 px and saved as JPEGs of about 130 KB each before they go into the page.
- **Checks are light.** After each change the page script is syntax-checked with `node --check`. No browser preview or screenshot was taken in this session, so visual details (symbol placement on small phones, for example) have not been confirmed by eye.

## 2. How we communicate

- **Namdev describes what he wants to see**, in plain language and often with an image attached. Examples: "white crushed paper only for the front page", "add more scattered mathematical icons".
- **Claude replies briefly after each round**: what changed and anything worth knowing, without pasting code or the link again.
- **Claude says what it did not verify.** The page has not been looked at in a rendered browser by Claude, so each reply says so and asks Namdev to report anything that looks off.
- **Choices Claude made are named.** For example, the closing note is written in Namdev's voice by Claude, and Namdev can edit the wording. The blue and red used for legs a and b were kept even after the dark theme became black.
- **Fixes are requested the same way.** If something looks wrong, describe it or attach a screenshot, and it is fixed in the next round.

## 3. How we build the page

1. **Plan the look first.** A small set of colour, type and layout tokens is written at the top of the stylesheet. Everything else reads from them.
2. **Gather content before styling it.** The history entries and the maths come first, then the layout wraps them. History facts used: Plimpton 322 and YBC 7289 (Babylon), Baudhayana's Sulba Sutra (India), Pythagoras of Samos, Euclid's Elements I.47, the Chinese gougu rule and Liu Hui, Bhaskara II, Garfield's 1876 proof, and Loomis's collection of 370 proofs.
3. **Build the diagrams as live SVG.** Each diagram is redrawn from numbers whenever a slider or input changes, so the picture and the arithmetic cannot disagree.
4. **Check the logic and the script.** The calculator formulas are plain functions of their inputs. The script is syntax-checked before every publish.
5. **Publish, then refine in rounds** as in section 1.

---

## 4. Technical details

### Stack
- One self-contained HTML file. No framework, no build step, no bundler.
- Vanilla JavaScript in one immediately-invoked function.
- All diagrams are inline SVG generated from template strings.
- Fonts from Google Fonts: **Bricolage Grotesque** (display), **IBM Plex Sans** (body), **IBM Plex Mono** (numbers and labels), each with a system fallback stack.
- Hosted as a Claude artifact. The page may load scripts and fonts only from an allowlist, and it cannot download files or open print dialogs, so none of those are used. The only external resource is the Google Fonts stylesheet.

### Page structure
| Section | id | Content |
|---|---|---|
| Hero | `top` | Title, formula, grid and paper background, floating symbols |
| The idea | `intro` | Legs and hypotenuse, squares drawn on each side with a, b sliders (1 to 10) |
| How it's built | `proof` | Two equal squares of side a + b, each holding four copies of the triangle; sliders for a, b (1 to 9) |
| History | `history` | Seven expandable stops built from `<details>` |
| Calculator | `calculator` | Find c, a or b; scale drawing; triple chips; right-triangle checker |
| Real life | `real-life` | Six interactive examples |
| Note | `note` | Short closing note signed Namdev |

A sticky top bar holds the section links, the theme button and a reading-progress line.

### Theming
- All colours are CSS custom properties defined on `:root`. The dark values are redefined in two places: under `prefers-color-scheme: dark` (guarded with `:not([data-theme="light"])`) and under `[data-theme="dark"]`.
- **Light:** neutral greys and white. **Dark:** pure black `#000`, `#0b0b0b` panels, white text.
- On load the script sets `data-theme="light"` if the viewer has not chosen a theme, so the page opens white by default. The moon/sun button toggles `data-theme` and redraws every SVG so its colours follow.
- The hero has its own token overrides so its text, grid and paper image always match its theme: `bg-light.jpg` in the white theme, `bg.jpg` in the dark theme. The paper image is used only in the hero, not behind the rest of the page.

### Hero
- Layered CSS backgrounds: a fine 32 px grid, a heavier 160 px grid, then the paper image (`--hero-img`). Layers shift a few pixels with pointer movement through the `--mx` and `--my` variables set by JavaScript.
- A radial vignette fades the edges into the theme colour.
- Text sits on translucent theme-coloured boxes so it stays readable over the paper texture.
- About 22 symbols (π + − ÷ × % = √ ∠ θ ∞ ∑ △ ²) are absolutely positioned spans. Each has its own size, rotation, opacity, colour, parallax strength and drift timing set through inline custom properties. They sit behind the text boxes and ignore pointer events.

### Icons
- A hidden SVG sprite holds 25 `<symbol>` icons drawn with `stroke: currentColor`. They therefore turn white in the dark theme and dark in the white theme without separate files.
- Icons appear in the nav, section labels, history stops, real-life cards, the theme button and the note signature.

### The maths in the page
| Feature | Formula used |
|---|---|
| Find hypotenuse | c = √(a² + b²), computed with `Math.hypot` |
| Find a leg | leg = √(c² − other²); rejected unless c is greater than the known leg |
| Right-triangle check | Sort three sides; compare s₁² + s₂² to s₃² with a relative tolerance of 1e-9; reports right, acute or obtuse, and rejects sets that fail the triangle inequality |
| Squares on sides | Square on the hypotenuse is built from the vector (b, −a) perpendicular to the hypotenuse |
| Proof diagram | Tilted square corners at (a,0), (a+b,a), (b,a+b), (0,b) inside a square of side a + b |
| Ladder | h = √(5² − d²); angle = atan2(h, d); "near safe 75°" shown for 70° to 78° |
| Screen size | width = diagonal × w ÷ √(w² + h²) for 16:9, 4:3, 21:9, 19.5:9 |
| Field shortcut | Saved distance = (a + b) − √(a² + b²) |
| 3-4-5 corner | Angle from the law of cosines, cos θ = (3² + 4² − d²) ÷ (2·3·4); "square" when within 0.5° of 90° |
| Ramp | run = rise × ratio; surface = √(rise² + run²) for slopes 1:20, 1:12, 1:8 |
| Distance between points | √(dx² + dy²) on a 12 × 8 grid with draggable points snapped to whole units |

### Rendering approach
- Each interactive block has one `draw` function that rebuilds its SVG from current input values and updates the readout text. Inputs call it on every `input` event.
- Sliders use native `<input type="range">`; the draggable points use pointer events with `setPointerCapture` and `touch-action: none` so dragging works on touch screens.
- Number outputs use `font-variant-numeric: tabular-nums` so digits line up.

### Responsive layout and accessibility
- A 16 px minimum side gutter that also respects phone safe-area insets, a 1040 px content column, and grids that collapse to one column at 820 px.
- Diagrams are SVG with `viewBox`, so they scale to the container. Wide step lines scroll inside their own box instead of widening the page.
- Visible keyboard focus outlines, `aria-label` on SVG figures and the theme button, `aria-live` on calculator results, and icon-only decoration marked `aria-hidden`.
- `prefers-reduced-motion` turns off the symbol drift, the scroll cue bounce and the section slide-in.
- Sections slide up slightly when scrolled into view, but they are never hidden before that, so the page is complete without scrolling.

### Known limits
- The floating symbols can partly sit behind the title and formula on very narrow phones; that is intentional, but it has not been checked on a real phone.
- The 3-4-5 slider and ladder angle ranges are teaching examples, not safety or building-code advice.
- The page was syntax-checked but not run through a visual preview in this session.
- The artifact is private until its owner shares it from the Share menu.
