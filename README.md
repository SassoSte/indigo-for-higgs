# for-higgs: one-shot the Act 1C arrival directly in Higgsfield

Everything needed to prompt the Act 1C "arrival at the camp" shot by hand in the Higgsfield app,
without the CLI pipeline. Bring the result back as an MP4 and the conform step at the bottom
drops it into the master.

## The shot

- 5.0 s, 24 fps, 120 frames. Master frames 289 to 408 of a 1041-frame scope film.
- Content: a white step-van slides into a desert camp from screen-left with a dust wake; a rust-red SUV follows
  on the nearer left lane; both brake and stop. A bare steel loading ramp and an olive-drab missile launcher
  truck are already parked on the right. The operator (the viewer) sees a staged, ordinary lot.
- Hard vehicle order left to right at the end: **red SUV, white van, steel ramp, olive launcher.**
- The next master frame (409) cuts to an EO/IR sensor plate of this same parked tableau, so the shot has to end
  on vehicles stopped in roughly the layout of `images/02-END-parked-tableau-16x9.png`. It does not need to
  land pixel-exact on that frame; the cut at 409 is a hard cut anyway.
- What we want that Cut 15 lacks: a real moving camera. Low wide-angle, tracking or craning with the van so
  the ramp and launcher are revealed by the move instead of being pre-staged in a locked wide.

## Taste rules (operator's, non-negotiable)

1. Matte, sun-bleached, dust-filmed vehicles. No gloss, no candy red, no toy look.
2. No bleach, no blown sky or ridge, no radioactive dust. Blockbuster contrast is fine, overexposure is not.
3. Grey dusty desert palette. No sunset oranges.
4. **No morphing, no redraw.** Terrain, shrubs, rocks, the ramp and the launcher must be rigid across every frame.
   Parked vehicles are rock solid. This is the rule every generated take so far has broken.
5. Only two vehicles arrive. No extra vehicles, tents, people, text.

## Files

| Path | Use |
|---|---|
| `images/01-START-lowangle-16x9.png` | Start frame. GPT Image 2 re-photograph of the end tableau from a low 18 mm position. Letterboxed 16:9 (2176x1224, picture rows 148 to 1076). |
| `images/02-END-parked-tableau-16x9.png` | End tableau. Use as end frame only if you accept the morph risk below; better as a reference image. |
| `images/03-locked-wide-vehicles-entering-16x9.png` | First frame of the Cut 15 locked plate (vehicles entering at left edge). Alternative start frame if you want the move to begin from the wide. |
| `images/04-act1b-frame288-...png` | The frame immediately before this shot. Lens, speed and dust language only. Its grade is deliberately bleached and must **not** be copied. |
| `images/05-cut15-plate-midshot-grade-ref.png` | Mid-shot of the Cut 15 plate. This is the grade target. |
| `reference-video/cut15-act1c-locked-plate-16x9-letterboxed.mp4` | The shipped locked-off Act 1C, letterboxed to 16:9. Use as a video reference where the model accepts one (Seedance 2.5 omni_reference / video_edit). |
| `reference-video/cut15-act1c-locked-plate-120fr-2176x928.mp4` | Same, native scope, for viewing. |
| `prompts/try-*.txt` | Three untried prompts, one per approach. Paste as-is. |
| `prompts/used-*.txt` | Every prompt already run, for the record. Do not rerun them unchanged. |
| `failed-takes/*.mp4` | Two of the failures, so you can see what "redraw" and "set changes" look like before judging a new take. |

## Settings that were used

- Kling v3.0, mode pro, 5 s, 16:9, sound off, 1920x1080 out. Takes 1, 2, 4, 5.
- Seedance 2.0, 1080p, 5 s, 16:9, genre drama, audio off. Takes 0, 3.
- Start keyframe: GPT Image 2, 16:9, 2k, image input = end tableau, prompt in `prompts/used-start-keyframe-gpt-image-2.txt`.

## What failed and why (six takes)

| Approach | Result |
|---|---|
| Kling, end image only, no start | Rejected by the server. Kling needs a start image. |
| Kling, start + end image (takes 1, 2) | Camera reaches a near-final composition at about 2.3 s, then the whole frame dissolves toward the end image for 2.7 s. Terrain, tracks and shrubs redraw every frame. Take 1 also grew a second van. |
| Seedance 2.0, start + end image (take 3) | Snaps to the wide in ~8 frames, then holds with repeated frames while converging on the end image. |
| Seedance 2.0 with frame 288 as image reference (take 0) | Copied the bleached grade, hard cut at frame 98, van drove past the ramp. |
| Kling, start only, no end image (takes 4, 5) | Less redraw but the set changes: a tent appears, the launcher turns into a cargo truck, the ramp vanishes, or a second SUV appears and the van overshoots the ramp. |

Pattern: an end image makes the model fake the landing by morphing; no end image lets it redress the set.
The camera move itself was fine in every take. The problem is always the last two seconds.

## Three things to try, in order

1. **Seedance 2.5, omni_reference or video_edit, reference video = the Cut 15 plate** (`prompts/try-1-...`).
   The plate pins the set and vehicle paths; the prompt asks only for a camera move. Most likely to respect rule 4.
2. **Kling 3.0 pro, start image only, with a short move that stops dead at 2 s** (`prompts/try-2-...`).
   Gives the model three seconds of "camera locked, nothing changes" instead of asking it to land a crane on a
   target it cannot reach. If it still redraws during the hold, Kling is out.
3. **Cinema Studio Video 3.0, start image only** (`prompts/try-3-...`). Different engine, same keyframe.

Judge a take by scrubbing the last two seconds at full size: watch one shrub and one patch of tire track.
If either changes shape while the camera is still, it is a redraw and the take is dead regardless of how good
the move looks.

## Bringing a result back

Save the MP4 (any 16:9 size, 24 fps preferred) and run from the repo root:

```bash
python3 tools/conform_take_v6.py <downloaded.mp4> bay/master_arrival/takes_v6/<name>.mp4
```

That crops the letterbox band to 2176x928, trims to 120 frames, CRF 14 BT.709, no frame interpolation. Then:

```bash
python3 tools/build_cut16_sbs.py bay/master_arrival/takes_v6/<name>.mp4 bay/master_arrival/sbs_<name>.mp4 "<label>"
```

```bash
python3 tools/assemble_conformed_master.py --meetup bay/master_arrival/takes_v6/<name>.mp4 --output bay/film-shahed-strike-cut16-<name>.mp4
```

```bash
python3 tools/verify_cut14.py bay/film-shahed-strike-cut16-<name>.mp4
```

The verifier checks container, frame count, black and freeze frames, and cut points. It does **not** detect
redraw. Full history and measurements: `bay/cut16-trial-notes.md`.
