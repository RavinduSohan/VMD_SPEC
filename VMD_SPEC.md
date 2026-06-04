# VMD Format Specification

**Format:** Video Markdown (VMD)
**File extension:** `.vmd`
**Author:** Sohan Dananjaya
**Version:** 1.0
**Date:** 2026

---

## Abstract

VMD (Video Markdown) is a plain-text file format for describing animated visual content. A `.vmd` file is a YAML document that specifies a scene — elements, their visual properties, and how they transform over time. A VMD renderer parses the file and produces a frame sequence, which is encoded into a video file.

VMD is designed so that anyone who can write YAML can produce animated technical content without learning a programming language, a timeline editor, or a visual design tool. The format is text-based, version-control-native, and renderer-independent.

---

## 1. File Structure

A `.vmd` file is a YAML document with the following top-level fields:

```yaml
concept: "string"           # Required. Title of the animation.
total_duration: number      # Required. Sum of all phase durations in seconds.
background: "#hex"          # Required. Canvas background color.
emoji: "unicode"            # Optional. Emoji representing the concept.

subtitles:                  # Required. List with at least one empty string.
  - ""

phases:                     # Required. Named phases containing elements.
  phase_name:
    duration: number        # Duration of this phase in seconds.
    description: "string"   # Optional. One-sentence summary of the phase.
    elements:
      element_name:
        type: primitive | morph
        shape: <shape_expression>
```

**Constraints:**
- `total_duration` must equal the sum of all phase durations.
- `subtitles` must be present and contain at least one entry.
- `shape:` is always a single line. The parser fails immediately on a line break inside a shape expression.
- Element names must be valid YAML keys.

---

## 2. Canvas

```
x-axis:   -8.0  to  +8.0   (16 logical units wide)
y-axis:   -6.0  to  +6.0   (12 logical units tall)
origin:   (0, 0) — center of canvas
y-direction: positive Y is UP
```

All coordinates, sizes, and radii are in logical units. The renderer maps logical units to pixels based on the output resolution. A logical unit at 1920×1080 output is 120 pixels.

---

## 3. Element Types

### 3.1 primitive

A static element or an element with a single animation chain. Does not use the `~>` morph operator.

```yaml
element_name:
  type: primitive
  shape: circle[r:0.8 fill:#4fc3f7/0.8 at:(0,0)] -> fadeIn(0.6s, 0.3s)
```

### 3.2 morph

An element that transforms through multiple states using the `~>` operator.

```yaml
element_name:
  type: morph
  shape: circle[r:0.8 fill:#4fc3f7/0.4 at:(0,0)] -> fadeIn(0.6s, 0.3s) ~> square[w:1.6 fill:#6bffb8/0.9 at:(0,0)] (1.2s, ease.inOut, 2s)
```

---

## 4. Shape Expression Syntax

A shape expression is a single line of text following `shape:`. It consists of a shape literal optionally followed by animation commands and/or morph steps.

```
shape: <primitive>[<properties>] -> <command>(<args>) ~> <primitive>[<properties>] (<duration>, <easing>, <delay>)
```

### 4.1 Property syntax

Properties are written as space-separated key:value pairs inside square brackets:

```
circle[r:0.8 fill:#4fc3f7/0.6 stroke:#4fc3f7/0.3 strokeWidth:1.5 at:(0,0)]
```

- Color values: `#RRGGBB` with optional opacity suffix `/0.0–1.0`
- Position: `at:(x,y)` where x and y are logical units
- Sizes: plain numbers in logical units
- Angles: number followed by `deg`
- Times: number followed by `s`

### 4.2 Command chaining

Animation commands chain with `->`:

```
shape[...] -> fadeIn(0.6s, 0.3s) -> moveTo((4,0), 2s, ease.inOut, 0.8s)
```

### 4.3 Morph steps

Morph steps chain with `~>`. Each step specifies duration, easing, and delay:

```
shape[...] ~> shape[...] (duration, easing, delay)
```

The delay is the absolute start time within the phase — not relative to the previous step.

---

## 5. Primitives

### circle
```
circle[r:<num> fill:#hex/opacity stroke:#hex/opacity strokeWidth:<num> at:(x,y)]
```
| Property | Type | Description |
|---|---|---|
| `r` | number | Radius in logical units |
| `fill` | #hex/opacity | Fill color |
| `stroke` | #hex/opacity | Stroke color |
| `strokeWidth` | number | Stroke thickness |
| `at` | (x,y) | Center position |

### rectangle
```
rectangle[w:<num> h:<num> fill:#hex/opacity stroke:#hex/opacity strokeWidth:<num> at:(x,y)]
```

### square
```
square[w:<num> fill:#hex/opacity stroke:#hex/opacity strokeWidth:<num> at:(x,y)]
```

### ellipse
```
ellipse[w:<num> h:<num> fill:#hex/opacity stroke:#hex/opacity strokeWidth:<num> at:(x,y)]
```

### line
```
line[from:(x1,y1) to:(x2,y2) stroke:#hex/opacity strokeWidth:<num>]
```
Lines draw themselves by morphing from zero length to full length:
```
line[from:(-4,0) to:(-4,0) stroke:#4fc3f7/0.5 strokeWidth:1.5] ~> line[from:(-4,0) to:(4,0) stroke:#4fc3f7/0.5 strokeWidth:1.5] (1.8s, ease.inOut, 3s)
```

### triangle
```
triangle[size:<num> fill:#hex/opacity stroke:#hex/opacity strokeWidth:<num> at:(x,y) direction:up]
```
`direction`: `up` (default) · `down` · `left` · `right`. Also accepts `rotation:<N>deg`.

### star
```
star[points:<num> outerRadius:<num> innerRadius:<num> fill:#hex/opacity stroke:#hex/opacity strokeWidth:<num> at:(x,y)]
```

### regularPolygon
```
regularPolygon[sides:<num> r:<num> fill:#hex/opacity stroke:#hex/opacity strokeWidth:<num> at:(x,y)]
```

### text
```
text[text:"string" fontSize:<num> fill:#hex/opacity at:(x,y)]
```
| Property | Type | Description |
|---|---|---|
| `text` | string | Content. Escape sequences: `\n` `\t` `\"` `\\` `\uXXXX` |
| `fontSize` | number | Size in pixels |
| `fill` | #hex/opacity | Solid text |
| `outline` | #hex/opacity | Hollow/stroke-only text. Do not combine with `fill`. |
| `strokeWidth` | number | Stroke thickness for outline text |
| `font` | string | Font family (default: Arial) |
| `fontWeight` | normal \| bold | Snaps at 50% midpoint during morph |
| `fontStyle` | normal \| italic | Snaps at 50% midpoint during morph |
| `letterSpacing` | number | Ratio relative to fontSize. Interpolates in morph. |
| `textAlign` | left \| center \| right | Alignment anchor at `at:(x,y)` (default: center) |
| `at` | (x,y) | Position |

**Text width limits:**
```
fontSize:64  →   8 chars max      fontSize:36  →  18 chars max
fontSize:52  →  10 chars max      fontSize:30  →  22 chars max
fontSize:46  →  12 chars max      fontSize:22  →  30 chars max
fontSize:40  →  14 chars max      fontSize:16  →  44 chars max
```

### functionGraph
```
functionGraph[fn:"sin(x)" xStart:<num> xEnd:<num> segments:<num> stroke:#hex/opacity strokeWidth:<num> at:(x,y)]
```
Supported functions: `sin` `cos` `tan` `exp` `log` `sqrt` `pow`

### parametric
```
parametric[paramX:"cos(t)" paramY:"sin(t)" tStart:<num> tEnd:<num> segments:<num> stroke:#hex/opacity strokeWidth:<num> at:(x,y)]
```

### group
```
group{at:(x,y)} [ shape, shape, icon ]
```
Alternative syntax: `group[at:(x,y)] { shape, shape, icon }`

Shapes inside the group are comma-separated. The entire `shape:` line including group contents must remain a single line. The group moves as one unit.

### image
```
image[src:"images/path/file.jpg" w:<num> h:<num> fit:cover radius:<num> at:(x,y)]
```
| Property | Type | Description |
|---|---|---|
| `src` | string | Repo-relative path. Must start with `images/`. |
| `w` / `h` | number | Dimensions in logical units. Omit one to preserve aspect ratio. |
| `fit` | cover \| contain | Fill mode |
| `radius` | number | Corner radius in logical units |
| `opacity` | 0–1 | Overall opacity multiplier |
| `grayscale` | 0–1 | Desaturation amount. Interpolates in morph. |
| `blur` | number | Blur radius. Interpolates in morph. |
| `tint` | #hex/opacity | Color overlay. Interpolates in morph. |
| `crop` | (x,y,w,h) | Crop region. Interpolates in morph — use for wipe reveals. |

Image `src` crossfades at the 50% midpoint of a morph. Position, size, and state properties (grayscale, blur, tint, crop) interpolate smoothly.

### icon
```
icon[icon:"prefix:name" size:<num> color:#hex/opacity at:(x,y)]
```
| Property | Type | Description |
|---|---|---|
| `icon` | "prefix:name" | Required. Exact format. |
| `size` | number | Logical units. 1 ≈ 50px. |
| `color` | #hex/opacity | Icon color |
| `opacity` | 0–1 | Multiplies color opacity |
| `at` | (x,y) | Position |
| `rotation` | Ndeg | Static orientation at spawn |
| `flip` | horizontal \| vertical \| both | Mirror |

**Icon sets:**
- `mdi` — 7,638 icons. Expressive, filled. Characters, objects, environments.
- `lucide` — 1,771 icons. Clean outline. System components, data flows.
- `logos` — 2,091 icons. Brand identities. Use only when the brand is the subject.

Icon-to-icon morphs are crossfades. Position, size, color, and opacity interpolate. SVG geometry does not path-interpolate between different icon names.

---

## 6. Animation Commands

All animation commands apply to all primitives and icons. Commands chain with `->`.

| Command | Parameters | Description |
|---|---|---|
| `fadeIn` | `(duration, delay)` | Fade from transparent to target opacity |
| `fadeOut` | `(duration, delay)` | Fade to transparent |
| `moveTo` | `((x,y), duration, easing, delay)` | Move to canvas position |
| `rotate` | `(angle, duration, easing, delay)` | Rotate. Use `360deg` for full spin. |
| `scale` | `(factor, duration, easing, delay)` | Scale. 1.0 = unchanged. 1.4 = 40% larger. |

### Text-only commands

| Command | Parameters | Description |
|---|---|---|
| `typewriter` | `(duration, delay)` | Characters appear sequentially with soft 0.12s ramp each |
| `typewriter` | `(duration, delay, cursor)` | Same, with blinking caret that fades after completion |
| `wordReveal` | `(stagger, delay)` | Words drift upward into position. Stagger is per-word interval. |
| `focusPull` | `(duration, delay)` | Starts blurred + letterSpacing wide, collapses to sharp. Minimum duration 0.8s. |
| `focusPull` | `(duration)` before `~>` | Blurs through a morph transition and lands sharp at destination. |

---

## 7. Morphing

The `~>` operator transforms one element state into another. Syntax:

```
shape[...] ~> shape[...] (duration, easing, delay)
```

Multiple morph steps chain directly:

```
shape[...] ~> shape[...] (dur, ease, delay) ~> shape[...] (dur, ease, delay)
```

**Interpolated properties** (smooth):
`at:(x,y)` · `r` · `w` · `h` · `icon size` · opacity · `strokeWidth` · stroke color · fill color · icon color · `fontSize` · `letterSpacing` · image `grayscale` · `blur` · `tint` · `crop`

**Snapping properties** (snap at 50% midpoint):
`icon name` · `image src` · `fontWeight` · `fontStyle`

---

## 8. Easing

| Value | Description |
|---|---|
| `ease.out` | Fast start, decelerates to stop |
| `ease.in` | Slow start, accelerates to finish |
| `ease.inOut` | Smooth both ends — natural travel |
| `linear` | Constant speed |
| `ease.spring` (alias: `spring`) | Overshoots then snaps back |
| `ease.elastic` (alias: `elastic`) | Exaggerated overshoot with wobble |
| `bounce` | Rebound easing |

---

## 9. Color System

### Color syntax
`#RRGGBB` with optional opacity suffix: `#4fc3f7/0.6`

### State colors
State colors carry fixed semantic meaning. They must not be reassigned.

| Color | Name | Meaning |
|---|---|---|
| `#4fc3f7` | cyan | alive, active, working, healthy, data in motion |
| `#ffd54f` | amber | cinematic moment only — one per scene |
| `#ffffff` | white | labels, neutral structural elements |
| `#ff6b9d` | pink | broken, failed, overloaded, blocked, under attack |
| `#6bffb8` | mint | resolved, correct, complete, the answer found |
| `#c77aff` | violet | deep system, unknown, beneath the surface |

### Opacity roles
```
background / ambient     fill /0.06–0.20   stroke /0.15–0.35
resting / seed state     fill /0.25–0.40   stroke /0.45–0.65
active / interactable    fill /0.40–0.65   stroke /0.65–0.85
triggered / fired        fill /0.65–0.85   stroke /0.85–1.00
cinematic / hero         fill /0.85–1.00   stroke /1.00
text inside filled shape fill /0.85 minimum
label whispers           fill /0.28–0.35
scene tag                fill /0.28–0.35
```

---

## 10. Scene Layout Conventions

These are conventions of the VMD format, not hard parser constraints. Renderers should follow them. Scene authors should follow them.

### Spawn gating
Elements that should appear later in a scene must have their first keyframe hidden — either at opacity `0` or positioned off-canvas. Do not rely on a delayed `fadeIn` alone when the first keyframe would otherwise be visible at frame zero.

```yaml
# Correct — first keyframe at opacity 0
shape: text[text:"RESULT" fontSize:52 fill:#ffd54f/0 at:(0,0)] ~> text[text:"RESULT" fontSize:52 fill:#ffd54f/0.9 at:(0,0)] (0.8s, ease.out, 12s)
```

### Causality timing
Element A's animation completes at time T. Element B responds at T + 0.3s. Causally coupled events (elements that represent the same event) morph simultaneously.

### Three-layer structure
Every scene distributes elements across three layers:
- **Background** — world identity, opacity `/0.06–0.20`, slow or no motion
- **Actor** — lead characters, full active opacity, maximum two lead actors
- **Detail** — small ambient elements, independent motion at edges

### The cinematic moment
One per scene. Always one. The lead word or value in amber (`#ffd54f`) at large fontSize (52–64). When it appears, all other active elements dim to opacity 0.2–0.3. It holds for 2–3 seconds then shrinks. This is followed by up to three label whispers at `fontSize:13 fill:#ffffff/0.3` positioned at `y:-3.2`.

### Scene tag
Every scene begins with a scene tag — a small text element identifying the scene number and name:
```yaml
tag:
  type: primitive
  shape: text[text:"① SCENE NAME" fontSize:15 fill:#ffffff/0.3 at:(-5.2,4.85)] -> fadeIn(0.6s, 0.2s)
```

---

## 11. Hard Rules

These are parser or renderer constraints. Violating them produces either a parse error or silent incorrect output.

| Rule | Consequence of violation |
|---|---|
| `shape:` is always one line | Parser fails immediately |
| `subtitles:` must be present with `- ""` | Parse error |
| `image src` must start with `images/` | Renders blank silently |
| `icon` name must be exact `prefix:name` | Renders placeholder silently |
| `triangle` uses `direction` or `rotation` | Default direction is `up` |
| Text escape sequences: `\\` `\"` `\'` `\n` `\t` `\r` `\uXXXX` | Literal text is not escape-aware otherwise |
| Text inside a filled shape is a separate element at the same center | Required for readable overlap |
| Amber (`#ffd54f`) appears nowhere before the cinematic moment | Breaks color semantics |
| Maximum two lead actors per scene | Convention, not enforced by parser |

---

## 12. Version History

| Version | Date | Notes |
|---|---|---|
| 1.0 | 2026 | Initial public specification. Format formerly named `.sherlock`, renamed to `.vmd`. |

---

## Author

Video Markdown (VMD) — Created by **Sohan Dananjaya**

This specification is the canonical reference for the `.vmd` file format. Implementations that parse or render `.vmd` files should conform to this document.
