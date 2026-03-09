# RetroHoops -- UX/Accessibility Review Memory

## Project Overview
- Single-file HTML5 Canvas basketball game (`/Users/carlray/VS_Code/RetroHoops/index.html`)
- Vanilla JS + Canvas 2D API, no frameworks, no build step
- Served via `npx serve -p 4000`
- Font: Press Start 2P (Google Fonts)

## Target Visual Direction
- Warm NES-era basketball aesthetic (Tecmo Bowl, Double Dribble era)
- Reference palette: `#3D2E1A`, `#7A4C17`, warm golds `#F8E0A7`
- Restrained minimalism, earthy warm tones, generous negative space
- NOT cold blue/purple arcade neon

## Review Findings (2026-03-08)
- Court wood tones (`#8B5E1A`, `#7A5016`, `#6B4512`) already align with warm palette
- Ball colors (`#e85d04`, `#f4a261`) and rim (`#e05000`) are correct
- CRITICAL: Background gradient is cold navy (`#040410` to `#16163e`) -- must shift to warm browns
- HUD uses neon shadowBlur glows -- NES aesthetic demands flat pixel text, no glow
- Crowd base shades are cold blue-purple -- need warm dark browns
- CSS bezel uses blue arcade glow -- needs warm amber
- Backboard is neutral grey -- should be warm cream
- Game-over overlay uses cold dark blue backdrop and mint green text

## Accessibility Gaps (Canvas-inherent)
- No aria-label or fallback text on canvas element
- No keyboard input support (mouse/touch only)
- HUD labels at ~40% white opacity may fail 4.5:1 contrast for small text
- Drag-only input with no single-point alternative

## See Also
- [retrohoops-palette.md](retrohoops-palette.md) for full color mapping
