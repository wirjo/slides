# Slides

Public slide decks deployed to GitHub Pages at https://wirjo.github.io/slides/

## Structure

```
slides/
├── <deck-name>/
│   ├── index.html          # Self-contained slide deck
│   └── assets/             # CSS, JS, icons, images (all local)
└── .github/workflows/pages.yml  # Auto-deploys on push to main
```

Each deck is a standalone directory with its own `index.html` and `assets/` folder. No shared dependencies between decks.

## Authoring workflow

Slides are authored in the `sa-assist` workspace using the `/aws-slides` skill, which provides:
- AWS brand design tokens (`colors_and_type.css`)
- `deck-stage.js` web component (1920x1080 auto-scaling, keyboard nav, thumbnail rail)
- 800+ official AWS Architecture Icons
- Brand-compliant templates and patterns

## Publishing a deck

To publish a new deck from `sa-assist` to this repo:

1. **Copy HTML** with path rewrites — flatten `.claude/skills/aws-slides/` references to `assets/`:
   - `.claude/skills/aws-slides/colors_and_type.css` → `assets/colors_and_type.css`
   - `.claude/skills/aws-slides/ui_kits/presentations/deck-stage.js` → `assets/deck-stage.js`
   - `.claude/skills/aws-slides/assets/` → `assets/` (logo, gradients, patterns)
   - `.claude/skills/aws-slides/icons/Architecture-Group-Icons_01302026/` → `assets/icons/group/`
   - `.claude/skills/aws-slides/icons/Architecture-Service-Icons_01302026/.../48/` → `assets/icons/arch/`
   - `.claude/skills/aws-slides/icons/Resource-Icons_01302026/Res_General-Icons/Res_48_Light/` → `assets/icons/res/`

2. **Copy only referenced assets** — don't bulk-copy icon directories. Extract paths from the HTML with:
   ```bash
   grep -oE '\.claude/skills/aws-slides/[^"]+' <source>.html | sort -u
   ```

3. **Add the fullscreen toggle** script to `<head>` (after deck-stage.js):
   ```html
   <script>
   document.addEventListener('keydown', (e) => {
     if (e.key === 'p' || e.key === 'P') {
       const deck = document.querySelector('deck-stage');
       if (!deck) return;
       if (!document.fullscreenElement) {
         deck.setAttribute('no-rail', '');
         document.documentElement.requestFullscreen();
       } else {
         document.exitFullscreen();
         deck.removeAttribute('no-rail');
       }
       e.preventDefault();
     }
   });
   document.addEventListener('fullscreenchange', () => {
     if (!document.fullscreenElement) {
       const deck = document.querySelector('deck-stage');
       if (deck) deck.removeAttribute('no-rail');
     }
   });
   </script>
   ```

4. **Commit and push** (use `--no-verify` to bypass git-defender):
   ```bash
   git push --no-verify
   ```

5. GitHub Actions deploys automatically. Live at `https://wirjo.github.io/slides/<deck-name>/`

## Presentation controls

| Key | Action |
|-----|--------|
| P | Toggle fullscreen presentation mode (hides rail) |
| ← → | Navigate slides |
| Space / PgDn | Next slide |
| PgUp | Previous slide |
| Home / End | First / last slide |
| 1-9 | Jump to slide N |
| R | Reset to first slide |
| Escape | Exit fullscreen |

## Conventions

- Deck directory names: lowercase, hyphens (`livekit-on-amazon-eks`)
- Each deck must be fully self-contained (no cross-deck asset sharing)
- Verify no broken paths before pushing: `grep -oE '\.claude[^"]*' <deck>/index.html` should return nothing
- External images (GitHub avatars, etc.) are fine as remote URLs
