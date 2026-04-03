# RetroHoops Warm NES Palette Mapping

## Reference Colors (from cosmos.so direction)
| Name | Hex | Usage |
|---|---|---|
| Deep brown | `#1A1008` | Body bg, outer bezel |
| Dark brown | `#2A1E10` | Background gradient mid, crowd base |
| Rich brown | `#3D2E1A` | Background gradient bottom, bezel inner |
| Medium brown | `#7A4C17` | Backboard border, labels |
| Warm gold | `#F8E0A7` | HUD text, trajectory dots, spotlight |
| Muted gold | `#D4A84B` | Best score, secondary text, non-swish particles |
| Warm red | `#C04020` | Game-over heading, urgent timer |

## Already Correct (keep as-is)
| Element | Hex |
|---|---|
| Rim orange | `#e05000` |
| Ball body | `#e85d04` |
| Ball highlight | `#f4a261` |
| Ball seams | `#3d0c00` |
| Court primary | `#8B5E1A` |
| Court alt strip | `#7A5016` |
| Court seams | `#6B4512` |

## Elements That Must Change (cold -> warm)
| Element | Current (cold) | Proposed (warm) |
|---|---|---|
| Background top | `#040410` | `#1A1008` |
| Background mid | `#09092b` | `#2A1E10` |
| Background bottom | `#16163e` | `#3D2E1A` |
| Crowd base shades | `#0c0c36` etc (blue) | `#2A1A0C` etc (brown) |
| Crowd blue accent | `rgba(0,80,200,0.55)` | `rgba(180,140,40,0.55)` |
| Backboard face | `#d8d8d8` (grey) | `#E8DCC8` (cream) |
| Backboard border | `#999999` (grey) | `#7A4C17` (brown) |
| HUD score fill | `#ffd700` + glow | `#F8E0A7`, no glow |
| HUD timer fill | `#ffffff` + blue glow | `#F8E0A7`, no glow |
| HUD best fill | `#7df9c0` + green glow | `#D4A84B`, no glow |
| HUD labels | `rgba(255,255,255,0.4)` | `rgba(248,224,167,0.45)` |
| CSS bezel glow | blue `rgba(100,120,255,0.3)` | warm `rgba(200,150,60,0.2)` |
| Game-over backdrop | `rgba(2,2,16,0.9)` | `rgba(20,14,6,0.92)` |
| Game-over heading | `#ff2222` + red glow | `#C04020`, no glow |
| Game-over best | `#7df9c0` + green glow | `#D4A84B`, no glow |
| Star particles (normal) | `#aaddff` (ice blue) | `#D4A84B` (muted gold) |
| Pop text default shadow | `#aaaaff` (lavender) | `#C8A060` (warm tan) |
| Trajectory dots | `#ffffffcc` | `rgba(248,224,167,0.7)` |
| Body background | `#000` | `#0D0A06` |
