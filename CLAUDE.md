# Fieldtone (working title)

A generative ambient sound instrument: a unique, continuous, Eno-inspired soundscape for every location and moment on Earth. Fully procedural (no audio files), built with Tone.js. Single static page in `index.html` — no build step, no dependencies to install.

## Immediate task

Deploy to Vercel production:

```
vercel deploy --prod --yes
```

Run from this directory. The owner is logged into Vercel on this machine (or run `vercel login` first). Static site — no build settings needed.

## Core design decisions (settled — do not relitigate)

- **Two-tier seeding.** A place's *character* is deterministic: lat/lng quantized to ~100m cells (0.001°), hashed → seeded PRNG (mulberry32) picks scale, root note, voice count, rhythm presence, tempo, Euclidean pattern shapes. *Details* reseed each session from character key + timestamp: loop lengths, octave choices, pattern rotations. Same place = same character, different performance each visit.
- **Rhythm depends on the place.** ~30% of cells are pure drone (rhythm = 0). Others range from faint pulse to steady groove. Rhythm uses Euclidean patterns of different step counts (polymeter) so grooves phase and never repeat. Synthesized percussion only: MembraneSynth kick, NoiseSynth hats, FMSynth bell, all sharing the drone's reverb so beats sit inside the atmosphere.
- **Drones phase Eno-style.** 4–6 voices, each a Tone.Loop with an incommensurate interval (6–20s), slow attacks, sine/triangle, through FeedbackDelay → Reverb (decay 10) → lowpass → master. Scales restricted to Lydian / major pentatonic / Dorian / whole tone so any combination stays ethereal.
- **Time is real and local.** Local hour at the map center (timezone from Open-Meteo `utc_offset_seconds`, longitude/15 estimate as fallback) drives note density and filter brightness continuously, plus the sky-gradient background. Clock ticks every 20s.
- **Weather is live.** Open-Meteo current conditions every 10 min: precipitation → high "plink" shimmer probability + hat density; wind → filtered pink-noise air bed level + timing swing; temperature → filter brightness offset. No API key needed.
- **Travel = dissolve.** Moving to a new cell crossfades: old engine ramps to 0 over ~2.6s and is disposed, new engine builds and ramps in over ~3.2s, BPM ramps between tempos.
- **UI decisions from the owner:** title is the LOCATION (reverse geocoded via BigDataCloud client API), never the generated sound name (removed entirely). No sliders — everything driven by real location/time/weather. No dot-grid map. Navigation = search (Open-Meteo geocoding, any place on Earth) + geolocate button + geolocate on load (Brooklyn fallback). Aesthetic: minimal/Bauhaus, Space Grotesk + IBM Plex Mono, sky gradient tracks local hour, breathing rings (one per drone voice), center dot pulses at tempo when the place grooves.
- **Bloom chord** fires immediately on play so there's sound within a second (drones alone take ~10s to establish).

## Known issues / next steps (roadmap, in rough priority)

1. Deploy to Vercel (now).
2. iOS: audio failed inside the Claude-app webview during prototyping; verify it works in mobile Safari on the deployed URL. If the silent switch mutes it, consider routing through a MediaStreamDestination + `<audio>` element (media playback category ignores the switch) — attempted before, needs testing on a real deploy.
3. Screen-lock/background playback on iOS eventually requires a native wrapper (Capacitor) with background-audio entitlement; PWA manifest + service worker is a good intermediate step.
4. Optional: real map (Google Maps or MapLibre) for browsing; owner originally wanted Google Earth-style navigation. Requires API key decision.
5. Optional: neighborhood-level reverse geocoding granularity (e.g. "Crown Heights" not just "Brooklyn") — BigDataCloud's `locality` field sometimes has this.
6. Open design question (owner undecided): during very long sessions, slowly re-arrange details every 20–30 min (movements) vs. one unbroken performance per sitting.
7. Naming: "Fieldtone" is a placeholder. Owner enjoys naming exercises (past projects gravitated to VHS/broadcast-era references).

## Security note

Two Vercel tokens were pasted into the prior chat and must be revoked at vercel.com/account/settings/tokens if not already done.
