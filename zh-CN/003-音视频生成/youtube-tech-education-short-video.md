# YouTube Shorts Production Workflow

> **Purpose:** A portable, self-contained workflow for an AI agent to create a polished educational YouTube Short, iterate with a user, add Qwen3-TTS narration, and prepare a YouTube Studio draft.
>
> **Default deliverable:** English; under two minutes; 1080×1920 vertical; 30fps; H.264 video and AAC audio.
>
> **Principle:** Treat this document as a reusable production specification, not as a description of a particular user, machine, repository, previous project, URL, or existing file.

---

## 0. Configure the run before creating files

At the start of a new Short, establish these values. Do not assume any already exist.

| Variable | Meaning | Example |
|---|---|---|
| `<SHORTS_ROOT>` | Parent workspace for all Shorts | `/worktrees/youtube-shorts` |
| `<TOPIC_ID>` | Next unused, zero-padded sequence number | `002` |
| `<TOPIC_SLUG>` | Concise, lowercase, hyphenated topic name | `topic-foo` |
| `<PROJECT_ROOT>` | Topic container directory; contains only the current `script.md` at its top level plus one `workspace/` directory | `<SHORTS_ROOT>/<TOPIC_ID>-<TOPIC_SLUG>` |
| `<WORKSPACE>` | Everything other than scripts for this topic: renderer code, dependencies, assets, audio, previews, renders, manifests, notebooks, caches, and final output | `<PROJECT_ROOT>/workspace` |
| `<CHANNEL>` | Target YouTube channel | Ask when unknown |
| `<FILE_BROWSER_URL>` | Optional link base for a user-visible file browser | Deployment-specific |

If `<SHORTS_ROOT>` is not supplied or cannot be discovered, ask the user before creating files. Inspect the root before choosing `<TOPIC_ID>` so no existing project is overwritten.

---

## 1. Non-negotiable production rules

1. **One topic, one self-contained project.**
   - Create a fresh `<PROJECT_ROOT>` for each Short.
   - Install renderer dependencies, virtual environments, assets, caches, audio, preview files, and final output inside that project only.
   - Do not import from, build through, or share `node_modules`, Python environments, renderer caches, source code, or generated assets with another topic.
   - A previous project may be studied as inspiration, but the new project must be independently runnable.

2. **Script first; video second; narration third.**
   - Do not begin visual production until the user has reviewed the script.
   - Render and review a silent visual version before generating final narration.

3. **English is the final script language.**
   - A user may edit text in any language or add inline instructions such as `(remove this)` or `(rewrite this more directly)`.
   - Interpret the intent, apply the edit, and preserve a fluent English script unless the user explicitly requests a different output language.

4. **Be concise and compelling.**
   - Keep the Short below two minutes unless the user requests otherwise.
   - **Start with the subject itself.** The first frame and first spoken sentence should present the core fact, mechanism, data, command, or question directly.
   - Do not add a separate curiosity hook, teaser, setup, or promise when the technical content can begin immediately.
   - Eliminate slow introductions, repeated conclusions, and filler.
   - Prefer precise, descriptive titles over curiosity, urgency, or opinion-driven packaging.
   - Optimize for clicks while keeping the central factual promise defensible; do not fabricate evidence, results, or capabilities.

5. **Draft, never publish.**
   - The final YouTube Studio action in this workflow is saving a correctly configured **Draft**.
   - Do not make the video Public, Private, or Unlisted, and do not complete publication steps, unless the user explicitly changes this boundary.

6. **Respect human attention and reading limits.**
   - Design for what a person can actually notice, read, and understand while a scene is visible.
   - A four-second beat can carry one short idea—not a heading, paragraph, labels, and animation.
   - **NO METAPHORS (MANDATORY):** Never use analogies, metaphors, or childish comparisons (e.g., restaurants, leaky buckets, waiting rooms, highways). Explain the actual technical mechanism directly, literally, and professionally. Your audience consists of engineers.
   - **LITERAL STATE REPRESENTATION**: Instead of analogies, explain processes using literal representations of system assets—such as sequence flowcharts, database row states, configuration payloads, or strict API protocols.
   - Treat the practical reason or use case as primary content, not as a footnote. Give it title-level hierarchy when it is the main point.
   - If motion already explains the mechanism, do not repeat it with explanatory text. Keep only the minimum words needed to establish purpose or consequence.
   - Keep readable text static. Never move URLs, code, headers, or sentences that viewers must decode.

7. **DO NOT ACCUMULATE VERSIONS (STRICT NO-VERSIONING RULE).**
   - **CRITICAL:** NEVER create `v1`, `v2`, `v3`, or similarly versioned files for large media (like MP4s, WAVs, or project directories). 
   - Users DO NOT like large files taking up disk space. Always OVERWRITE the existing file.
   - Keep one current `<PROJECT_ROOT>/script.md` and edit it in place. Use source control when editorial history is needed.
   - `<WORKSPACE>` is not an archive.
   - Keep only the current implementation, current review preview, necessary assets, and final deliverables in `<WORKSPACE>`.
   - When a new preview replaces an old one, immediately OVERWRITE or DELETE the old preview, obsolete stills, temporary checks, and abandoned render variants.

---

## 2. Phase A — Receive the idea and write the first script

### 2.1 Infer the brief from context

A Short often begins after a useful discussion of a concept, rather than from a fully specified production brief. Use the current conversation as the source of truth for the topic, the important claims, the intended result, and the appropriate depth.

When the user asks to turn a discussed knowledge point into a Short, begin drafting immediately. Do **not** ask routine follow-up questions about audience, language, duration, visual style, evidence, or renderer. Make pragmatic defaults:

- English output;
- under two minutes;
- an immediate start with the core technical content;
- the warm editorial field-note design system in this document;
- the renderer best suited to the concept.

Ask only when execution is genuinely blocked—for example, the conversation does not identify a topic at all, a required source/asset is inaccessible, or two incompatible interpretations would produce fundamentally different factual content.

### 2.2 Create the project and script

Create the current script at the topic-container level:

```text
<PROJECT_ROOT>/script.md
```

The topic-container level is reserved for scripts only. Do not create `<WORKSPACE>` yet; it begins only after script approval and visual production starts.

Use this format:

```markdown
# <Short Title> — Script

> **Status:** Working English script; revise until approved.
> **Tone:** Direct, concise, lightly playful

---

## 1. <Section title>

**On screen**
- Title: `...`
- Visual: `...`

**Voice-over**
> ...

---

## 2. <Section title>

**On screen**
- ...

**Voice-over**
> ...

---

## 3. <Section title>

**On screen**
- ...

**Voice-over**
> ...
```

The script must include:

- numbered sections;
- concise on-screen direction;
- exact voice-over wording;
- an immediate opening that begins the explanation itself;
- enough explanation to make the mechanism, data, or comparison genuinely understandable;
- a clean ending after the last necessary fact, operation, or visual resolves—no conclusion, recap, sign-off, or dedicated ending section. Leave a short visual tail so the final motion can resolve and the viewer can read the completed state; do not cut off the last word or snap immediately to black;
- enough visual detail for a renderer to implement the Short without guessing.

### 2.3 Hand off for script review

After the script is written, reply with:

1. the absolute script path; and
2. a file-browser preview link if the environment provides a file browser.

Do not invent a link format. Build it from the runtime’s documented file-browser URL and path rules, or omit the link if none is available.

---

## 3. Phase B — Review and revision loop

The user may edit `script.md` directly or give instructions in chat.

For each review round:

1. Read the newest script and identify all replacements, deletions, annotations, and structural changes.
2. Interpret instructions in any language.
3. Rewrite the result as natural English while retaining the required **On screen** and **Voice-over** structure.
4. Patch `<PROJECT_ROOT>/script.md` in place. Do not create numbered script copies.
5. Briefly state what changed and share the script path and file-browser link when available.

Continue until the user clearly approves video production. Do not treat silence as approval.

---

## 4. Phase C — Select the right renderer

Choose the renderer based on what must be taught, not on habit. If uncertain, recommend one with a one-sentence reason and allow the user to choose differently.

| Renderer | Choose it for | Strength |
|---|---|---|
| **Remotion** | Engineering explainers, product/UI walkthroughs, architecture, code/configuration, charts, reusable templates, and media composition | React/TypeScript components, deterministic timelines, maintainability, and browser-oriented debugging |
| **Manim** | Equations, proofs, coordinate systems, geometry, vectors, algorithms, mathematical transformations, and rigorous scientific mechanisms | Semantic mathematical APIs for objects, axes, formulas, transformations, and independently renderable scenes |
| **HyperFrames** | HTML/CSS/JavaScript motion design, expressive graphic packaging, animation-heavy cards, and frame-precise composition | HTML composition model with seek-safe, frame-oriented animation; can combine with GSAP for explicit timelines and keyframes |

### Required skill-loading discipline

Before implementation, load the relevant available skills rather than relying on memory:

- For **Manim**, load `manim-video`. Follow its plan → code → low-quality render → review → final-render workflow.
- For **HyperFrames**, load `hyperframes` first. Follow its routing/install instructions and load the related core, animation, keyframes, creative, and CLI skills that it requires.
- For **Remotion**, load any available Remotion or technical-explainer skill. Use local composition registration, TypeScript checks, frame/studio preview, deterministic rendering, and media probing.

If a named skill is unavailable, use the renderer’s official documentation and retain the same verification discipline.

---

## 5. Phase D — Design system: warm editorial field-note style

This style specification is self-contained. Apply it by default unless the user supplies a different brand or art direction.

### Visual identity

Create a warm, polished, editorial technical video—not a default blue/purple AI template, not an unstyled slide deck, and not a crowded dashboard.

### Terminology Abstraction (MANDATORY)

- **NEVER use internal business terminology** (e.g., "Sandbox", "Gateway", "Step CA") in the script or visual assets.
- ALWAYS abstract concepts to universally understood architectural terms (e.g., "Dynamic Node", "API Proxy", "Internal CA"). Educational content must be broadly applicable, not tied to a specific company's internal infrastructure naming.

### Scientific & Mathematical Explanations (MANDATORY)

- **Beyond the Conclusion (The "Why" and "How")**: Never just state a scientific conclusion or a final formula (e.g., "$D=20N$"). You MUST explain the deeper physical, mathematical, or historical origin of the numbers. Distinguish between hardware physics (e.g., "$C=6ND$ due to forward/backward passes") and empirical learning laws. Reveal engineering trade-offs, historical optimizer bugs (e.g., Epoch AI fixing DeepMind's L-BFGS), and mathematical "aha!" moments. The audience wants profound, hard-won insights, not just a textbook summary.
- **Strict Epistemology**: Do not mischaracterize the nature of a concept. If a concept is an "empirical law" observed from data (like Scaling Laws), state it explicitly. Do not lazily frame it as an "engineering perspective" or a "theorem".
- **Academic Citations (MANDATORY)**: For all science and research-based videos, EVERY time a specific paper is mentioned (e.g., Kaplan, Chinchilla), you MUST render a formal, academic-style citation at the bottom of the screen (e.g., `Reference: "Title of Paper" (Author et al., Year)`). Treat the screen like a proper research presentation.
- **Epistemic Modesty (Data vs. Absolutes)**: When discussing empirical data, real-world scatter plots, or benchmarks, avoid absolute language (e.g., "clusters exactly inside", "converges perfectly"). Real-world data is messy; use grounded framing like "clusters around the optimal band".
- **Exhaustive Variable Labeling**: EVERY formula shown on screen MUST be accompanied by a clear, explicit legend defining EVERY variable right below it (e.g., `$L$ = Loss, $N$ = Parameters`). Do not assume the audience remembers from a previous scene.
- **Audio/Visual Asymmetry (Formula Translation)**: When a complex formula like $C \approx 6ND$ is on screen, the voiceover should NEVER read the literal math variables ("C equals six N D"). The voiceover must act as a human translator (e.g., "Compute FLOPs equals six times parameters times tokens") so the audience can map the spoken concepts to the visual symbols.
- **Auditory Cognitive Load (Number Rounding)**: Do not force the TTS to read out long decimals or hyper-specific academic numbers (e.g., "0.2849"). This destroys pacing. Abstract them in the voiceover into dramatic narrative concepts (e.g., "a tiny two-digit rounding error"), while leaving the exact precision for the visual screen.
- **Visual Curve Pairings**: Formulas must always be paired with a visual curve (or multiple curves) demonstrating the relationship between the key variables.
- **Preserve Iconic Formula Structures**: DO NOT mathematically simplify famous academic formulas if it destroys their recognizable shape (e.g., do not simplify Kaplan's iconic `(C / N)^\alpha` into `A / N^\alpha`). The audience expects the famous structure.
- **Eradicate Ambiguous Subscripts**: While preserving the formula's structure, you MUST rename confusing subscript variables (like `N_c` or `\alpha_N`) to unambiguous single-letter constants (like `C` or `\alpha`). In a fast-paced video format, adjacent letters are easily mistaken for multiplication (e.g., `alpha * N`).
- **SVG Coordinate Reality Checks**: When hand-coding SVG curves for "Performance vs Scale":
  - Remember that SVG Y-coordinates go DOWN. To show performance increasing, your path must go UP towards `Y=0`.
  - Empirical scaling laws usually exhibit **diminishing returns (logarithmic/convex growth)**. The curve must shoot up quickly and then flatten out. Do NOT draw exponential/accelerating curves for performance scaling; this is a fundamental logical error.
- **NEVER Risk In-Plot SVG Legends (MANDATORY)**: Never try to manually hardcode `<text>` tags inside an `<svg>` to label curves (e.g., `D ≈ 20N`). It is extremely fragile and AI agents constantly miscalculate the `(x, y)` coordinates, causing text to overlap the curve. ALWAYS place legends safely in standard HTML `<div>` blocks *outside* the SVG area (e.g., directly below the chart).
- **Horizontal Safe Margins (Visual Overflow)**: In a 9:16 vertical canvas (1080px wide), complex LaTeX formulas and long side-by-side flex layouts will easily overflow. Always stack complex side-by-side elements vertically (`flexDirection: 'column'`) and reduce font sizes aggressively for long equations to prevent edge clipping.

### Palette

| Role | Color |
|---|---|
| Cream background | `#FFF6E6` |
| Paper/card surface | `#FFFDF8` |
| Ink text and dark media base | `#2D241F` |
| Primary orange | `#F26A21` |
| Tangerine highlight | `#FF9E2C` |
| Peach support/accent | `#FFE0B5` |
| Rust emphasis | `#9E3E18` |
| Muted teal contrast | `#156B67` |
| Soft gray secondary text | `#75665B` |

### Layout and background

- Use a clean cream canvas.
- **STRICTLY VERTICAL (9:16) TOP-DOWN FLOW**: You are building for a 1080x1920 portrait screen. **NEVER use left-right (horizontal) Flexbox layouts** (`flexDirection: "row"`). All elements, boxes, and sequence diagrams must flow strictly top-to-bottom (`flexDirection: "column"`).
- **ARROWS MUST POINT DOWN**: Any diagram connecting nodes must use Down Arrows (`⬇️`), never Right Arrows.
- **FULL-SCREEN SPREAD (MANDATORY)**: Do NOT cram elements at the top of the screen! **NEVER** use `justifyContent: "flex-start"` unless a slide is intentionally mostly empty. You MUST spread all elements evenly across the entire 1920px vertical space by defaulting to `justifyContent: "space-evenly"` or `"space-between"`. 
- **STEP TIMELINE INDICATORS**: For multi-step architectures or timelines, add a clean stage-indicator badge at the top (e.g., `STEP 1: DUAL TRUST 👥` or `STEP 4: CLEANUP 🧹`) to visually chunk the flow.
- **VERTICAL CANVAS SPACE BUDGET**: The 1080x1920 portrait canvas is narrow and height-constrained. Keep flow diagrams to a maximum of 3 vertical components (e.g., Node ➔ Arrow ➔ Node). Compress component paddings/margins to prevent elements from colliding or rendering out-of-screen.
- **TRANSITION FLASH PROTECTION**: Never leave the canvas completely blank or empty at the start of a scene transition. Ensure structural host nodes (like headers or system actors) enter immediately at `frame 0` of the sequence, reserving delayed entrance transitions (e.g. `frame - 10`) exclusively for transient data packets or success banners.
- **BRAND SIGNATURE FRAMING (MANDATORY)**: To establish visual identity across all videos in the channel:
  - Add a static, 20px-height primary orange rule (`backgroundColor: PALETTE.primaryOrange`) absolute-positioned at the very top of the screen (`top: 0, left: 0, right: 0`) across all frames.
  - Layer exactly two low-opacity (`0.5-0.6`), oversized (1000px-1200px) radial-gradient peach/orange circular accents (`background: radial-gradient(...)`) at the top-left (`top: -300px, left: -300px`) and bottom-right (`bottom: -250px, right: -250px`) to give the cream canvas depth.
- Use one dominant idea and one focal visual per scene.

### Typography

- **MASSIVE FONTS (MANDATORY)**: Mobile viewers read fast. Set Main Titles to at least `90px-110px`, Section Subtitles to `42px-54px`, and paragraph/label/code text to at least `38px-64px`. Formulas or giant numbers can be up to `70px-100px`. If it looks "too big" for desktop, it's correct for mobile Shorts.
- Use bold, high-contrast sans-serif titles.
- **EMOJI INJECTIONS**: Intentionally place 1-2 expressive emojis in slide titles (e.g., `TRAP 🚨`, `RENEWAL ⏳`, `COMPLETED 🛡️`) and key diagram nodes to reduce visual dullness and enhance graphic energy.
- Use restrained monospace type for technical labels, compact badges, counters, framework names, and footer notes.
- Use strong wrapping rules and generous line height; never allow text to extend beyond safe margins.
- Prefer short headlines and concise bullets over paragraphs.

### Cards, panels, media windows, and code blocks

- Use off-white paper surfaces with rounded corners.
- Use a 2–5 px semantic-color border.
- Use only subtle, low-opacity warm shadows.
- Use upright horizontal rounded-rectangle media windows with thin accent borders and a small browser-style label bar when presenting video/code/UI.
- **CODE BLOCKS WITH OS WINDOW HEADERS**: Render code blocks (JSON, YAML) with a dark window header featuring macOS-style buttons (red, yellow, green window dots) and a clear, descriptive monospace label (e.g., `JSON_RPC_PAYLOAD`).
- **FULL CONTEXT FOR TOOL CALLS**: When rendering API payloads or tool call JSONs, don't just dump the raw arguments. Wrap it in a proper response structure (like `{"role": "assistant", "tool_calls": [...]}`) so the viewer understands the context of the JSON block.
- Do **not** use tilted cards, skewed windows, heavy shadows, glassmorphism, decorative diagonal lines, or unnecessary perspective effects.
- **CODE BLOCKS MUST PRESERVE WHITESPACE:** When rendering code (e.g., YAML config, bash scripts) inside a React `div`, you MUST apply `whiteSpace: "pre-wrap"` and `textAlign: "left"` to the container. Furthermore, NEVER use HTML entities (like `&#10;`) in JSX string literal props. ALWAYS pass multi-line code using Javascript string escapes or template literals (e.g., `code={"deploy:\n  mode: global"}`). Otherwise, standard HTML flow will collapse all your beautiful YAML indentation and newlines into a single unreadable line!

### Information hierarchy and motion

- Use a small chapter badge, large headline, primary visual, and explanatory card where appropriate.
- Use `01`, `02`, `03`-style bullets when a list is needed.
- Do not add a mandatory `FIELD NOTE`, recap card, closing takeaway, or dedicated ending scene. Stop immediately when the explanation is complete.
- Use a brief field-note summary only when the user asks for one or when it adds information that has not already been communicated.
- Animate with subtle fade/vertical spring entrances, light staggering, clean transitions, and intentional pauses.
- Motion should reveal hierarchy and sequence; it should never compete with the explanation.

### Attention budget

- Give every scene one dominant technical idea.
- Match copy length to screen time. For a roughly four-second beat, prefer one immediately readable phrase or a very short sentence.
- When the use case is the lesson, size it like a headline; do not demote it to small footer text.
- Remove secondary labels, mechanism descriptions, status codes, and protocol detail unless they are necessary to understand the lesson.
- Review the scene at playback speed, not only as a still frame. If the eye must choose between reading and following motion, simplify the scene.

### Quantitative comparison charts

- Use one consistent visual grammar for the whole comparison. Do not make viewers relearn a new chart type on every scene.
- Do not render a spreadsheet or table as the primary visual. Convert rows into a proportional axis with exact labels.
- Plot values at their true proportional positions. Never give adjacent hierarchy levels equal spacing when their numeric gaps differ.
- In a portrait video, prefer a vertical quantitative axis so the scale uses the long dimension. Labels may flank it on both sides without turning the composition into a horizontal slide.
- When the range is too large, zoom the same axis in stages while preserving its meaning. Do not replace proportional scale with decorative tier widths.
- Print complete units on every tick and value label: write `GB/s`, `TB/s`, and `PFLOP/s`, never unexplained abbreviations such as `T` or `P`.
- Treat mathematical notation as semantic. Use `5 ns = 10× L1`, not a decorative bullet or centered dot between values that form an equality.
- Center the axis mathematically within the drawable content area, then make left/right connector geometry symmetric around it.
- Keep labels outside the axis and reserve explicit non-overlapping positions for every final state. Verify the fully populated frame, not only early sparse frames.
- Synchronize each reveal to the sentence that discusses it, and hold the populated state long enough to read; do not make labels flash past merely to fit narration.
- Remove text that merely describes the encoding—such as `LINEAR`, `ACTIVE`, or an upper-bound explanation—when the axis, units, or color already communicates it.
- Use a thick, solid red border to indicate the item currently under discussion. Do not combine it with an `ACTIVE: ...` label, glow, or decorative motion.
- Let the screen carry exact secondary numbers while narration carries the comparison. Not every visible number must be spoken, but every measured value that is spoken must include its unit.

---

## 6. Phase E — Build a topic-local video project

Create exactly one production workspace inside the topic container. The topic root holds only the current `script.md` and `workspace/`; **every other file or directory** belongs inside `workspace/`.

For example:

```text
<SHORTS_ROOT>/123-topic-baz/
├── script.md
└── workspace/
    └── <all implementation material for this topic>
```

`workspace/` is the sole working directory for production. Nothing except `script.md` and `workspace/` may live at `<PROJECT_ROOT>`.

Inside `workspace/`, use whatever file and subtree names suit the chosen renderer and topic. There is **no required internal directory schema**. For example, a project may use `src/`, `public/`, `audio/`, `previews/`, `out/`, `final/`, a narration manifest, a Jupyter notebook, `node_modules/`, `.venv/`, cache directories, or differently named equivalents. The only invariant is that all source, dependencies, assets, audio, previews, outputs, final video, notebooks, caches, and every other implementation artifact remain somewhere under `<WORKSPACE>`.

This flexibility does **not** mean retaining every iteration. Use stable current-output names and delete superseded numbered outputs immediately after the replacement is verified. Keep history in source control when needed, not in duplicate workspace media or source trees.

Do not create a second project container such as `project/` or `short-project/` inside `workspace/`; `workspace/` itself is the production root.

Rules:

- Install all dependency manifests, packages, virtual environments, and renderer tools inside `<WORKSPACE>`.
- Keep all generated production files under `<WORKSPACE>`.
- Do not use shared or inherited build state.
- Default to **1080×1920, 30fps, H.264** for silent and final video unless the user requests another format.

### Silent-first delivery

1. Implement the Short from the approved script.
2. Render a silent preview.
3. Verify dimensions, FPS, codec/container, and duration with `ffprobe`.
4. Inspect representative frames and/or play the result to check text size, safe margins, crop, overlap, transitions, timing, and whether the copy is realistically readable at playback speed.
5. Share the preview path and a file-browser link when available.
6. After the replacement is verified, remove superseded previews, obsolete stills, and temporary review renders from `<WORKSPACE>`.

Do not generate final narration while the visual direction is still changing unless the user explicitly requests combined iteration.

---

## 7. Phase F — Visual convergence

Use the user’s feedback to refine the silent version.

For every revision:

1. Update the current script revision when speech or on-screen content changes.
2. Update topic-local source and assets.
3. Re-render the changed composition.
4. Re-check visual hierarchy, legibility, vertical safe areas, attention load at playback speed, and technical output.
5. Share the replacement preview.
6. Delete the preview and review artifacts that the replacement supersedes; retain only the current reviewable output.

Proceed to narration only after the user confirms that the silent video has converged.

---

## 8. Phase G — Generate narration with Qwen3-TTS CustomVoice

### Recommended configuration

- Model: `Qwen/Qwen3-TTS-12Hz-1.7B-CustomVoice`
- **Preferred speaker:** `Ryan` for English narration, when the installed model exposes it.
- **Fallback speaker:** select another supported voice that fits the requested language and tone when `Ryan` is unavailable; record the actual selection in the narration manifest.
- Language: `English` by default; use the approved script language when different.
- Precision: `torch.bfloat16`
- Attention implementation: `sdpa`
- Style instruction: `Speak clearly, warmly, and with an upbeat technical-demo tone.`

### Create a narration manifest

Create `<WORKSPACE>/narration-manifest.json` as the single source of truth:

```json
{
  "model": "Qwen/Qwen3-TTS-12Hz-1.7B-CustomVoice",
  "speaker": "<selected supported speaker; Ryan when available>",
  "language": "<approved script language>",
  "instruction": "Speak clearly, warmly, and with an upbeat technical-demo tone.",
  "clips": [
    {
      "id": "first_point",
      "file": "first_point.wav",
      "text": "Exact approved narration goes here."
    }
  ]
}
```

### Generating and validating continuous TTS

> **CRITICAL TTS RULE:** Attempt End-to-End (E2E) single-clip generation first because it best preserves prosody and breath pacing. However, long Qwen3-TTS generations can fade, omit sentences, repeat phrases, hallucinate words, or corrupt dense numeric sequences. Loudness normalization fixes level—not missing or incorrect speech.

> **HARD SPEECH-RATE LIMIT:** Final narration must use its natural generated pace whenever possible. FFmpeg `atempo` must never exceed `1.3`. Values above `1.3` are prohibited even when they would make the video fit a target duration. If narration is too long, shorten redundant spoken wording while preserving required facts on screen, reduce pauses that are genuinely excessive, or split the subject into multiple videos. Never solve an overlong script by making dense technical narration difficult to follow.

1. **Attempt one complete clip:** Define the approved narration as one clip and generate it once.
2. **Transcribe the raw generation:** Reject it if any required clause, number, unit, acronym, or final sentence is missing, duplicated, paraphrased into a different claim, or phonetically corrupted.
3. **Use a minimal semantic split only after E2E fails:** Split into the smallest practical number of balanced clips—usually two—at complete topic or sentence boundaries. Keep the same speaker, instruction, loudness target, and generation style. Never repair a long narration with many tiny patches.
4. **Verify every clip independently:** Transcribe each raw clip before joining. A valid combined transcript cannot prove that a bad clip boundary sounds natural.
5. **Join once, then process globally:** Insert one short, consistent boundary pause. Apply any `atempo`, loudness normalization, resampling, and final padding to the complete joined track—not selectively to one section.
6. **Apply Loudness Normalization & Silence Padding:** Use FFmpeg's `loudnorm` filter (EBU R128) after generation succeeds.
   > **WARNING (Clipped End & Abrupt Stop):** Neural TTS models often stop instantly after generating the final word, leading to an abrupt, unnatural end. Furthermore, audio buffer flushes can cut the last syllable off entirely. **You MUST force FFmpeg to append 1.5 - 2.0 seconds of absolute silence at the end using the `apad=pad_dur=1.5` filter.**
   > **WARNING (Remotion Sequence Extension):** Because you physically extended the audio file via padding, you must also manually increase the `durationInFrames` in your `Composition.tsx` by the corresponding amount (e.g., `+45` frames for 1.5 seconds) AND extend the final scene's `<Sequence>` `durationInFrames` to soak up the extra time.
   > **WARNING (Immediate Content Start):** Do NOT add silent intro padding or delay at the beginning of the video track. The technical content and narration must start immediately at `0.00s`.
   > **WARNING (Sample Rate Resampling):** The `loudnorm` filter has a known bug where it can default the output to an extremely high `192kHz` sample rate, which causes Remotion (and most browsers) to completely mute the audio track during rendering. **You MUST explicitly add `-ar 44100` to force standard resampling.**
   > **WARNING (Audio Cache Trap):** If your TTS script saves audio to `workspace/audio/narration_full.wav`, but Remotion (`staticFile`) reads from `workspace/public/narration_full.wav`, you will continuously render video with an outdated, broken audio file! You MUST `cp audio/narration_full.wav public/narration_full.wav` before rendering.
   ```bash
   ffmpeg -y -i audio/narration_full_raw.wav -af "loudnorm=I=-14:LRA=11:TP=-1.5,apad=pad_dur=1.5" -ar 44100 public/narration_full.wav
   ```

> **CRITICAL RULE FOR TTS SCRIPTING:** The spoken language generated by the TTS must closely match the text displayed on the visual slides. Avoid unnecessary verbosity or divergent phrasing. Keep the narration concise and tightly aligned with the on-screen keywords so the viewer can effortlessly connect the audio with the visuals.

> **TTS PRONUNCIATION & SWALLOW FIXES:** If your script contains camelCase variables (e.g., `sameSite`) or hyphenated acronyms (e.g., `proxy-ssl-ca`), the neural TTS model may choke, swallow syllables/words in the middle of narration, or terminate early. You MUST rewrite these phonetically with spaces (e.g., `same site` or `proxy S S L C A`) in the JSON manifest text to ensure flawless pronunciation.

Do not blindly spell every acronym letter by letter. Test the intended pronunciation and use the smallest reliable rewrite: for example, `D-RAM`, `A one hundred`, or ordinary `CUDA`. Keep the human-readable spelling on screen even when the narration manifest uses a phonetic form.

For dense numeric narration:

- Include a unit every time a measured value is spoken; never say “L2 takes seven” or “bandwidth reaches five hundred four.”
- Separate long numeric runs into natural sentences. If the TTS still corrupts them, narrate the endpoints or central comparison and retain intermediate exact values on screen.
- If a duration limit requires acceleration, calculate the smallest factor that fits and apply it uniformly to the entire joined narration. Never accelerate only one clip or section, and never exceed `1.3×`.

> **THE 180-SECOND HARD LIMIT (SHORTS FORMAT):** YouTube Shorts are STRICTLY bounded. A video over 180.00 seconds will be automatically reclassified by YouTube as a standard VOD (Video on Demand), killing its Shorts algorithm reach. 
> To ensure you never hit this ceiling, **aggressively strip all fluff words** from the script:
> - Remove hooks and setup language. Begin with the first technical fact, operation, or question.
> - Remove summary "takeaways," conclusions, sign-offs, and ending transitions. Stop immediately after the last necessary explanation.
> - Delete conversational padding like "observed from massive amounts of data" or "exactly". Every single second counts.

### Audio-Visual Sync via Whisper (MANDATORY)

- **INTEGRATED PIPELINE EXECUTION**: Always build an all-in-one automation script (e.g. `generate_tts.py`) that encapsulates voice generation, loudness normalization, and Whisper timestamp transcription in a single step. This keeps the asset workspace robust and makes timestamps instantly reviewable in a single console log.


1. **Install & Cache-First Whisper**: You MUST use OpenAI Whisper (`pip install openai-whisper`) for timestamp extraction. Before deciding on a model size (`tiny.en`, `base.en`, etc.), you MUST check the local cache (e.g., `~/.cache/whisper/`) and strictly prefer a model that is already downloaded to save bandwidth and time. Do NOT attempt to build custom silence-chunking fallback logic.
2. **NO OVER-ENGINEERING**: Do NOT write complex Python scripts with "sliding window matching" or automated text-search algorithms to parse the Whisper output. This is prone to edge-case failures.
3. **Manual LLM-in-the-Loop Alignment**: Let the AI agent simply run a tiny script to print the `whisper` transcript with timestamps to the console, and visually read the output to identify the exact start times of the scenes.
4. **Frame Calculation & Hardcoding**: The AI agent will manually convert these timestamps to frame offsets (e.g., `seconds * 30 fps`) and hardcode them into the `FRAMES` constants in the React file using standard file replacement tools.
5. **Single Audio Element**: Ensure the Remotion composition only has one root `<Audio src={...narration_full.wav} />` component.

### Validate and Render

1. Verify `narration_full.wav` exists in `<WORKSPACE>/public/`.
2. Probe the file: valid decode, correct length, and non-silent samples.
3. Render the narrated video and verify the expected streams, dimensions, FPS, duration, and audio/video synchronization.
4. **MANDATORY FINAL WHISPER VERIFICATION**: Before showing the user the final output, write a tiny `verify.py` script that uses `whisper.load_model("base").transcribe("out/video.mp4")` to extract the text *directly from the final rendered MP4*. This is the ONLY way to guarantee the final video isn't using a cached, broken audio track. Share this whisper transcript with the user.
5. Compare the final MP4 transcript with the exact last required technical fact. If that fact or its unit is absent, corrupted, or cut by the composition duration, do not deliver the file.

---

## 9. Phase H — Final review and YouTube Studio Draft

After the user approves the narrated final:

1. Verify the exact final MP4 locally: it exists, is non-empty, decodes, and has expected video/audio streams.
2. If the destination channel is unknown, ask the user which channel to use.
3. Propose a direct, attractive, truthful title and a one-line subtitle/description. Use user-supplied wording exactly when given.
4. Follow the runtime’s documented browser-upload procedure and browser-visible path mapping.
5. Confirm YouTube Studio accepts the exact file and completes relevant processing before editing metadata.
6. Set and verify:
   - title;
   - subtitle/description;
   - required audience declaration (ask when unknown; never infer it);
   - requested playlist or additional metadata.
7. Save the upload as a **Draft**.
8. Verify the channel content list shows the expected title, Draft state, and duration.
9. Tell the user the draft is ready and explicitly hand off all remaining Studio steps.

Do not continue to visibility settings, publication, scheduling, or release without a new explicit instruction.
