---
name: tuney-music
description: Use when the user needs background music, a soundtrack, a jingle, a music bed for a video, ad, podcast, game or app, or wants to adjust or download a Tuney track. Drives the Tuney MCP tools (find_tracks, generate_track, adjust_track, download_track).
---

# Tuney music

Tuney composes royalty-free music from hand-made, modular elements and fits it to an exact length.
All work goes through the `tuney` MCP connector; if its tools are missing, tell the user to connect
Tuney (`/mcp` in Claude Code, or the Tuney connector on claude.ai) and sign in.

## Workflow

1. Turn the brief into a style. If the user names a genre, mood or tempo, call `list_styles` once
   and use the exact values it returns. Otherwise pass the brief as a free-text `prompt`.
2. Prefer `find_tracks` first: results are instant and previews are free. Offer two or three
   options with their preview links and lengths.
3. Use `generate_track` when nothing fits or the user needs a specific length from the start, then
   `wait_for_task` with the returned `task_id`. Call `wait_for_task` again if it reports a timeout.
4. Fit the track with `adjust_track`: `length_seconds` for an exact duration (for video, use the
   clip length), `time_stretch` or `pitch_shift` for feel and key, `mute` to drop stems such as
   drums for voice-over sections, `turning_point_seconds` to make the music change character at a
   given second (combine with `length_seconds` to lengthen a track and place the turning point in
   one call), `ending` for a `fade_out` or `short` ending, `regenerate` with stem names for a fresh
   take. Every adjustment returns a task; wait for it.
5. Only call `download_track` when the user asks for the file. It spends Tuney credits. Give the
   link promptly because signed links expire. Use `format="stems"` when the user will mix it.

## Presenting results

- When a track is ready, show: name, length in seconds, bpm, key, genre and mood, and the preview
  link. Then offer stems and adjustments (length, ending, turning point, tempo, key, muting or regenerating stems).
- If you can open web pages (the Claude desktop app's browser pane, `navigate` or `preview_start`),
  open the preview URL there so the user can listen immediately; the page shows a standard audio
  player. Otherwise give the link.
- Stems: call `download_track` with `format: "stems"`, then fetch the zip from `download_url`,
  unzip it, rename each file with the `contents` map in the result (`2.mp3` melody, `3.mp3`
  chords, `4.mp3` bass, `Drums.mp3` drums, `Swoosh.mp3` swoosh fx, `Impact.mp3` impact fx) and
  attach the files to the conversation (for example with SendUserFile), one card per stem. All
  stems are the full length of the track and line up in a DAW.
- A download that returns a `task_id` is still rendering: call `wait_for_task` with
  `kind: "download"` until it returns `download_url`.

## Rules

- Never download repeatedly to "try" formats, and never download the same track twice; previews are free, downloads are not.
- If a tool reports missing credits, show the purchase link it returns and stop.
- Lengths are seconds between 5 and 600. Round to the user's real need, not the nearest minute.
- Keep responses short: track name, length, bpm, key, preview link.
