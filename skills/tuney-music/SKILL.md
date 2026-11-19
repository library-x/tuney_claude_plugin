---
name: tuney-music
description: Use when the user needs background music, a soundtrack, a jingle, a music bed for a video, ad, podcast, game or app, or wants music composed to a video (score a video), or to adjust or download a Tuney track. Drives the Tuney MCP tools (find_tracks, generate_track, adjust_track, download_track, score_video).
---

# Tuney music

Tuney composes royalty-free music from hand-made, modular elements and fits it to an exact length.
All work goes through the `tuney` MCP connector; if its tools are missing, tell the user to add the
Tuney connector (`/mcp` in Claude Code, or the Tuney connector on claude.ai). No sign-in is needed.

## Without sign-in, and signed in

- **Without sign-in** every tool works. Results then say `listen_only`: all audio has a pulsing
  tone over it (previews, downloads and the music in rendered videos), and downloads are free.
  Tell the user once, in a sentence, and carry on.
- **Signed in**, audio is clean and downloads spend the user's Tuney credits.
- Call `sign_in` only when the user asks to sign in or wants clean files. Say first what happens:
  the app shows its Tuney sign-in (in Claude Code: `/mcp`, choose Tuney, Authenticate), an account
  is free to create, and the Tuney tools may pause until they finish.
- After they have signed in, call `get_track` with the `cue_id` of each track made before: it comes
  back clean under a new `cue_id` (the result says `copied_from`). Use the new id from then on, and
  call `render_video` again to get clean music under a video.
- If a tool says the free tries for this hour are used up, tell the user and offer `sign_in`.

## Workflow

1. Turn the brief into a style: call `list_styles` once and pick the genre, mood and tempo that fit
   (one or two of them is enough). `find_tracks` matches those exactly. A free-text `prompt` is
   matched on overall feel only and knows nothing about instruments, so use it when no style fits,
   and check the genre, mood and tempo of what comes back.
2. Prefer `find_tracks` first: previews are free. It returns three tracks by default; offer them
   with their preview links and lengths.
3. Use `generate_track` when nothing fits or the user needs a specific length from the start, then
   `wait_for_task` with the returned `task_id`. Call `wait_for_task` again if it reports a timeout.
4. Fit the track with `adjust_track`: `length_seconds` for an exact duration (for video, use the
   clip length), `time_stretch` or `pitch_shift` for feel and key, `mute` to drop stems such as
   drums for voice-over sections, `turning_point_seconds` to make the music change character at a
   given second (combine with `length_seconds` to lengthen a track and place the turning point in
   one call), `ending` for a `fade_out` or `short` ending, `regenerate` with stem names for a fresh
   take. Every adjustment returns a task; wait for it.
5. Only call `download_track` when the user asks for the file. Signed in, it spends Tuney credits;
   without sign-in it is free and the file has the pulsing tone. Give the link promptly because
   signed links expire. Use `format="stems"` when the user will mix it.

## Scoring a video

1. Get the video in. A public link to the file itself: `import_video`. A local file (attached, or in
   the working directory): `get_video_upload_link`, then run the `example` curl command it returns;
   the response holds the `video_id`.
2. Call `score_video` with the `video_id` and a `prompt` taken from the brief (or the `cue_id` of a
   track the user picked). It returns the track, the plan and the rendered video. If the result says
   `timed_out`, call `wait_for_task` with its `task_id` and `kind: "score_video"`.
3. Report the plan in a sentence or two: where the turning point and the hits landed, where the
   music ends (before the sonic logo if there is one), and any notes. Offer changes: another turning
   point (`turning_point_seconds`), a short ending, ending earlier (`music_end_seconds`), or another
   track (a different prompt).
4. After `adjust_track` on the scored track, call `render_video` so the new version plays under the video.
5. Call `analyze_video` first only when the user wants to talk through the edit before scoring.

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
- Scored videos: open `video.url` in the browser pane when you can; otherwise give the link.
- Links (`preview_url`, `download_url`, `video.url`) are on the Tuney API host and work where Cloud
  Storage is blocked; `*_direct_url` is the Cloud Storage original. If your sandbox blocks both,
  give the user the links and say they can allow the API host in their network settings.
- A download that returns a `task_id` is still rendering: call `wait_for_task` with
  `kind: "download"` until it returns `download_url`.

## Rules

- Never download repeatedly to "try" formats, and never download the same track twice; previews are free, and a signed-in download spends credits.
- If a tool reports missing credits, show the purchase link it returns and stop.
- Videos: up to 100 MB through the upload link, up to 200 MB and 10 minutes through import_video.
- Lengths are seconds between 5 and 600. Round to the user's real need, not the nearest minute.
- Keep responses short: track name, length, bpm, key, preview link.
