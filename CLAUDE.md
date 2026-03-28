# YouTube Practice Looper

Single-file HTML app that loops sections of YouTube videos for music practice. Hosted on Vercel at youtube-practice-looper.vercel.app.

## Stack

- Single `index.html` file, no build step, no dependencies
- YouTube IFrame API for playback control
- Deployed via `vercel --yes --prod`
- GitHub repo: jules19/youtube-practice-looper

## Done

- Loop counter — tracks reps per session
- Keyboard shortcuts — A/B/Space/R/arrows
- Draggable A/B markers on visual timeline
- Tap timeline to seek
- Paste any YouTube URL
- localStorage persistence (video, A/B, speed)
- iPad-friendly touch UI
- Deployed on Vercel

## Roadmap

### High priority (practice flow)
1. Speed ramp trainer — start at 0.5x, auto-increment (e.g. every 5 loops) until 1x. Configurable step size and loops-per-step
2. Micro-loop — button to create a temporary ±1.5s loop around the current playhead for isolating tiny phrases
3. Section memory — save multiple named loops per video ("Intro", "Bridge voicing", "Ending lick"), jump between them
4. Smoother loop transition — seek slightly before B to eliminate the tiny glitch when jumping back to A
5. Practice mode — cycle: play loop N times, pause for a beat, play again. Gives your brain a reset

### Medium priority (usability)
6. Count-in — seek to a couple seconds before A so you have time to get ready
7. Speed fine-tuning — +/- 0.05x buttons for granular manual control
8. Shareable URL — encode video ID, A, B, and speed in the URL hash
9. Import/export — save loops as a JSON file for backup or sharing a practice routine
10. PWA manifest — proper home screen app name and icon on iPad
11. Fullscreen mode — hide browser chrome, especially on iPad

### Stretch goals
12. Metronome overlay — detect BPM, play a click track alongside the video using Web Audio API
13. MIDI keyboard control — WebMIDI mapping (e.g. C3=restart, D3=set A, E3=set B) for hands-free control
14. Automatic phrase detection — detect silence gaps to suggest loop points for solos/licks/vocals
15. Waveform on timeline — visual cues for where sections start/end
16. Tap tempo / BPM display — tap to the beat to see target BPM
17. Dark/light mode toggle
18. QR code sharing
