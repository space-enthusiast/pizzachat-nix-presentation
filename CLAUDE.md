# Pizzachat Nix Presentation

## Project Overview
Slidev presentation about solving developer environment bottlenecks using Nix + direnv.

## Tech Stack
- **Slidev** — markdown-to-slides presentation framework
- **Nix flake** — reproducible dev environment
- **direnv** — automatic environment loading

## Commands
- `slidev slides.md` — start dev server with live preview
- `slidev build slides.md` — build static SPA for deployment
- `slidev export slides.md` — export to PDF

## Project Structure
- `slides.md` — the presentation content (Slidev format)
- `flake.nix` — Nix flake defining dev dependencies
- `.envrc` — direnv config (just `use flake`)

## Conventions
- Slides are separated by `---`
- Use Slidev's code block line highlighting syntax: `{all|1-3|5|all}`
- Do not add co-author lines to git commits
