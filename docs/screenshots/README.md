# Screenshots

This directory holds the screenshots referenced from the top-level
README. To capture / update:

## What to capture

| File | What | Where to capture |
|---|---|---|
| `self-view.png` | Self-View top section showing **Today · 3 things** + "By the numbers" black band | `localhost:7600/` |
| `the-week.png` | The Week with at least one bound event (black border) and one floating event (red border) | `localhost:7600/calendar` |
| `briefing-panel.png` | Slide-out Briefing panel for a future event with rich participants (Top of mind / Where you stand / Last we talked / Worth bringing up) | Click `[▸ Brief]` on a future event card |
| `recap-panel.png` | Same panel for a past event (Recap mode) | Click `[▸ Recap]` on a past event card |
| `inbox-edit.png` | Inbox row in edit mode showing the form (key/when/who/what + focus-block toggle) | `localhost:7600/inbox` → click `✎ Edit` |
| `persona-card.png` | Persona Card with Next-meeting hint, + Add fact / ✨ Enrich row, and facts grouped by prefix | `localhost:7600/people/p_maya-chen` |
| `the-network.png` | Force-directed relationship graph with `me` at the centre and at least one red high-load node | `localhost:7600/network` |

## Capture settings

- Browser: **Arc** or **Chrome incognito** at 1440 × 900 (matches the
  README's display crop).
- DPI: **2x retina** (default on Macs). Don't rescale for github
  rendering — it does the right thing automatically.
- Format: **PNG** (lossless; the Bauhaus typography stays crisp).
- Crop tightly to the surface — don't include the top nav unless the
  screenshot specifically shows nav state.

## Updating

```bash
# After you take new shots
cp ~/Downloads/self-view.png docs/screenshots/self-view.png
git add docs/screenshots/*.png
git commit -m "docs: refresh screenshots — <brief description of what changed>"
git push
```

The README references each by relative path (e.g. `docs/screenshots/self-view.png`),
so file names matter — keep them stable.
