# Remotion YouTube Short Technical-Education Video Workflow

**Purpose:** A reusable process for producing any concise technical-education video as a vertical YouTube Short with Remotion.

**Default delivery:** `1080 × 1920`, `30 fps`, H.264 video + AAC audio.

> **Workspace rule:** Keep the video project, dependencies, caches, source media, previews, and rendered outputs in a dedicated non-source workspace. Do not install dependencies or place render artefacts inside an application/source repository unless the user explicitly requests it.

---

## 1. Content contract before implementation

Create a `script.md` as the single source of truth before building the composition. Organize it into chapters. Each chapter should contain:

- the learning objective;
- on-screen title and concise supporting points;
- visual plan (diagram, code excerpt, chart, demonstration, source clip, or UI);
- exact voice-over text;
- intended pacing and any required pause.

Use one narration audio file per chapter rather than one long audio file. This makes revisions, timing changes, and voice replacement tractable.

Suggested project layout:

```text
short-project/
  script.md
  src/
    index.ts
    Root.tsx
    video.tsx
  public/
    clips/       # optional source videos/demos
    images/      # diagrams, screenshots, illustrations
    audio/       # one narration file per chapter
  out/           # previews and final renders
  package.json
  tsconfig.json
```

Use descriptive, topic-neutral filenames such as `chapter-01-intro.mp3`, `chapter-02-mechanism.mp3`, and `chapter-03-takeaway.mp3`.

---

## 2. Build the Remotion project in the workspace

Install dependencies from the dedicated project directory:

```bash
cd /path/to/dedicated-workspace/short-project
npm install --include=optional
```

`--include=optional` matters on Linux because Remotion's compositor/FFmpeg-related optional packages may otherwise be absent. A missing package can appear only after rendering all frames, during encoding.

Keep Remotion, React, TypeScript, and renderer package versions aligned in `package.json` and commit the lockfile for reproducibility.

---

## 3. Define a vertical composition

Declare the Short explicitly in `src/Root.tsx`:

```tsx
<Composition
  id="TechNoteShort"
  component={TechNoteShort}
  durationInFrames={durationInFrames}
  fps={30}
  width={1080}
  height={1920}
/>
```

Compute the duration from the real chapter timing:

```text
durationInFrames = totalSeconds × fps
```

Do not keep an old hard-coded duration after replacing narration. The composition must end only after the final audio has completed and the final takeaway has remained readable.

---

## 4. Design for a phone screen

### Safe layout

Use explicit horizontal bounds for all text and key content:

```tsx
const WRAP: React.CSSProperties = {
  overflowWrap: 'break-word',
  wordBreak: 'normal',
};

<div style={{
  position: 'absolute',
  left: 60,
  right: 60,
  bottom: 110,
  ...WRAP,
}}>
  Important takeaway
</div>
```

Recommended working margins for a 1080×1920 composition:

```text
left/right: 60 px minimum
bottom: 95 px minimum
```

Keep important controls, captions, and conclusions farther from the right and bottom edges, where platform UI may overlap the video.

### Information density

- Prefer one idea per screen.
- Keep titles to roughly two lines.
- Put related elements into explicit vertical regions: title, visual, explanation, takeaway.
- Do not stack absolutely positioned blocks into the same vertical region.
- Use a readable sans-serif body font; use a monospaced font only for code, commands, labels, or values that benefit from it.
- Favor contrast and legibility over decorative effects.

### Source-video or screenshot windows

When embedding media as evidence or an example:

- use a horizontal, unrotated rounded rectangle;
- keep the border and shadow subtle;
- never rely on skewed frames, diagonal masks, or heavy shadows for meaning;
- preserve enough area for surrounding commentary;
- use `objectFit: 'cover'` only when cropping does not hide material facts.

For multiple source windows on a vertical Short, use a vertical sequence or an edited comparison rather than forcing tiny horizontal columns.

---

## 5. Implement a deterministic timeline

Use Remotion's frame-based primitives:

- `useCurrentFrame()` for the current frame;
- `interpolate()` for continuous values such as opacity and position;
- `spring()` for modest entrances;
- `Sequence` for chapter placement;
- `Audio` for narration; and
- `Video`, `Img`, SVG, or HTML/CSS for visuals.

Place the visual and its narration at the same chapter start frame:

```tsx
<Sequence from={chapterStart} durationInFrames={chapterDuration}>
  <ChapterVisual />
</Sequence>

<Sequence from={chapterStart} durationInFrames={chapterDuration}>
  <Audio src={staticFile('audio/chapter-02.mp3')} />
</Sequence>
```

Timeline rules:

1. Measure each generated audio file before finalizing chapter duration.
2. Allocate each chapter enough frames for its full narration plus a natural end hold.
3. Recompute every later chapter start when an earlier narration changes.
4. Keep the final conclusion visible for at least 1–2 seconds after it has appeared, where pacing allows.
5. Never use wall-clock time, `setTimeout()`, unbounded loops, or uncontrolled playback state for render-critical animation.

A small, controlled entrance movement is usually enough. Animation should clarify the explanation, not compete with it.

---

## 6. TTS workflow and audio verification

Choose the requested voice provider and use a consistent voice across the whole Short unless the script intentionally calls for multiple speakers.

For each chapter:

1. Generate the exact text from `script.md`.
2. Verify the generated asset exists locally.
3. Confirm it contains audible, non-silent samples.
4. Measure its real duration.
5. Copy it into `public/audio/` with a stable filename.
6. Update the chapter start/duration and composition end from those measurements.

Never store API keys, passwords, tokens, or voice-provider credentials in source code, documentation, command history, or chat. Use secure environment injection or the provider's authenticated browser flow. In documentation, represent all credentials as `[REDACTED]`.

Useful duration check:

```bash
ffprobe -v error \
  -show_entries format=duration,size \
  -of json public/audio/chapter-02.mp3
```

When replacing a voice, regenerate every affected chapter and verify that no old clips remain mixed into the render.

---

## 7. QA before the full render

Run TypeScript validation first:

```bash
npx tsc --noEmit
```

Render stills at representative frames:

```bash
npx remotion still src/index.ts TechNoteShort out/qa-opening.png --frame=30
npx remotion still src/index.ts TechNoteShort out/qa-middle.png --frame=450
npx remotion still src/index.ts TechNoteShort out/qa-ending.png --frame=900
```

Inspect these images for:

- title wrapping and screen-edge overflow;
- collisions between text, diagrams, media windows, and captions;
- code or diagram readability at phone scale;
- unintended black frames or missing assets;
- correct final frame and end-card hold.

Then render the complete video. A conservative Linux render command is:

```bash
npx remotion render src/index.ts TechNoteShort \
  out/tech-note-short.mp4 \
  --codec=h264 \
  --pixel-format=yuv420p \
  --concurrency=1
```

`--concurrency=1` is slower but can be more reliable for compositions with multiple video tracks or constrained resources.

---

## 8. Verify the encoded deliverable

Validate the actual MP4 rather than assuming a successful command is sufficient:

```bash
ffprobe -v error \
  -show_entries format=duration,size:stream=codec_name,codec_type,width,height,r_frame_rate \
  -of json out/tech-note-short.mp4
```

Expected essentials:

```json
{
  "video": "h264",
  "audio": "aac",
  "width": 1080,
  "height": 1920,
  "r_frame_rate": "30/1"
}
```

Extract and inspect at least opening, middle, and final frames:

```bash
ffmpeg -y -ss 1  -i out/tech-note-short.mp4 -frames:v 1 out/qa-opening.jpg
ffmpeg -y -ss 15 -i out/tech-note-short.mp4 -frames:v 1 out/qa-middle.jpg
ffmpeg -y -ss 30 -i out/tech-note-short.mp4 -frames:v 1 out/qa-ending.jpg
```

Also listen to the final seconds of the finished MP4. A valid AAC track does not prove that the final sentence was not cut off.

---

## 9. Common failures and fixes

| Symptom | Likely cause | Fix |
|---|---|---|
| Render completes frames but fails while encoding | Linux optional renderer dependency missing | Run `npm install --include=optional` in the project directory, then re-render. |
| TypeScript passes but MP4 fails | Still render and final encoder use different paths | Always perform a complete render and inspect it with `ffprobe`. |
| Final sentence is cut off | Composition or final `Sequence` is shorter than narration | Measure the final audio, extend the sequence and total composition, and add an end hold. |
| Chapters have awkward silence | Later `Sequence from` values were inherited from a previous narration version | Recalculate all starts from the current real audio durations. |
| Video is not vertical | Composition retained horizontal dimensions | Set `width={1080}` and `height={1920}` and confirm with `ffprobe`. |
| Source media cannot load | Asset lives outside Remotion `public/` | Copy/prepare it under `public/` and reference it with `staticFile()`. |
| Long English technical text overflows | Fixed font size without a bounded width | Set `left` and `right`, enable wrapping, shorten copy, or reduce font size only after preserving hierarchy. |
| Text blocks overlap | Absolute elements share the same vertical region | Create a region plan and assign distinct height ranges before styling. |
| On-screen details are too small | Horizontal design was shrunk into a Short | Recompose for vertical reading: one primary visual, fewer simultaneous points. |
| Render is unstable | Too many heavy media tracks render concurrently | Lower concurrency; trim or proxy assets if necessary. |
| TTS voice changes unexpectedly | Only some clips were regenerated | Audit every file in `public/audio/` before delivery. |
| Credentials leaked | A key was pasted into code, docs, logs, or chat | Revoke/rotate it, remove the leak, and use `[REDACTED]` in records. |

---

## 10. Final delivery checklist

- [ ] Project, dependencies, caches, media, and outputs live in a dedicated workspace.
- [ ] `script.md` is current and matches every voice-over line.
- [ ] `npx tsc --noEmit` passes.
- [ ] Representative opening, middle, and ending stills were checked.
- [ ] Every narration file has been measured with `ffprobe`.
- [ ] Visual and audio sequences share the same chapter starts.
- [ ] All chapter starts were recalculated after the latest TTS change.
- [ ] The final narration plays fully, followed by a readable end hold.
- [ ] Canvas is 1080×1920 at 30 fps.
- [ ] The final MP4 is verified as H.264 video + AAC audio.
- [ ] Finished MP4 frames contain no overflow, overlap, missing asset, or black-frame defect.
- [ ] Voice identity is consistent across all clips unless intentionally varied.
- [ ] No credential appears in the project, docs, logs, or deliverables.
