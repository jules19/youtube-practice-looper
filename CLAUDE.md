# YouTube Practice Looper

Single-file HTML app that loops sections of YouTube videos for music practice. Hosted on Vercel at youtube-practice-looper.vercel.app.

## Stack

- Single `index.html` file, no build step, no dependencies
- YouTube IFrame API for playback control
- Deployed via `vercel --yes --prod`
- GitHub repo: jules19/youtube-practice-looper

## Roadmap

1. Saved loops — named loops in localStorage (e.g. "chorus riff", "bridge solo")
2. Gradual speed-up — auto-increment playback speed by a small amount each loop until reaching 1x
3. Shareable URL — encode video ID, A, B, and speed in the URL hash
4. Count-in — seek to a couple seconds before A so you have time to get ready
5. Speed fine-tuning — +/- 0.05x buttons for granular manual control
6. Practice setlist — define multiple loops (even across videos) and move through them like a playlist
7. Import/export — save loops as a JSON file for backup or sharing a practice routine
8. PWA manifest — proper home screen app name and icon on iPad
9. Fullscreen mode — hide browser chrome, especially useful on iPad
10. Dark/light mode toggle — for bright room use
11. Smoother loop transitions — seek slightly before B to avoid audio clipping at the boundary
12. QR code sharing — generate from the shareable URL for easy cross-device scanning
13. Waveform on timeline — visual cues for where sections start/end (stretch goal)
14. Tap tempo / BPM display — tap to the beat to see target BPM
