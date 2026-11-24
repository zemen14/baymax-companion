# Baymax Companion

**An interactive AI assistant character that lives on your screen: it watches your cursor, listens for its name, speaks back, and does small jobs for you, then quietly waits in the background until you need it again.**

Built as a single self-contained web app. No build step, no server, no accounts, no dependencies. Open one HTML file in Chrome and Baymax is alive.

---

## Why I built this

Most assistant interfaces are a chat box. I wanted to explore what an assistant feels like when it has a *presence*: a character that notices you, follows what you're doing with its eyes, answers out loud, and gets out of the way when you're busy. Baymax Companion is that experiment, a voice-first, character-driven assistant interface built entirely on native web platform APIs.

The gaze system is inspired by Avi Vashishta's [robot hero that follows your cursor](https://www.avivashishta.com/blog/robot-hero-follows-your-cursor-media-io), which picks frames from a look-around video. This project takes the same interaction idea and re-engineers it for a **single still photograph**, which meant solving the eye-tracking problem differently (see *How it works*).

## What it does

- **Cursor-tracking gaze.** Baymax's pupils follow your mouse or finger in real time, and his head tilts with the movement. Stop moving for three seconds and he looks around the room on his own. He blinks.
- **Wake-word voice interaction.** Tap the microphone once and Baymax keeps listening in the background. Say his name, then a command. No button presses per command.
- **Spoken replies.** Every answer is spoken in a soft, slow, gentle voice (pitch and rate tuned on the browser's speech synthesis), with Baymax-style phrasing, and shown in a speech bubble.
- **Real work, not just chat.** A command engine routes intents to actions:
  - *"What time is it?"* / *"What day is it?"*
  - *"Open GitHub"* (also LinkedIn, Gmail, YouTube, Google, Instagram)
  - *"Search for ..."*, opens a web search
  - *"Remember to ..."*, saves the note to his on-screen memory list
  - *"Tell me a joke"*, *"What can you do?"*, and graceful fallbacks that offer a search instead of failing silently
- **Background mode.** Say *"Baymax, go to background"* (or tap the button) and he shrinks into a small floating companion in the corner of the screen, still listening for his name. Click him and he's back.
- **Type instead of talk.** Every voice command also works from the text box, and spoken replies still play.

## How it works

### The gaze engine

The hard part: a photograph can't turn its head. The blog's approach (hundreds of extracted video frames) doesn't apply, so the gaze is synthesized:

1. **The photo's eyes are located precisely** (image coordinates measured, not guessed), and lightweight pupil overlays are placed in *percentages* of the image box, so they stay registered on the face at any screen size.
2. **Gaze is measured from the character's own center**, not the window's. This was the key fix during development: measuring against the viewport gave almost no vertical range, so Baymax could look left and right but barely up or down. Re-centering the coordinate system on Baymax gives the full range in every direction.
3. **Exponential easing** (`gaze += (target - gaze) * 0.07` per frame) makes the eyes glide and settle like the frame-blending in the video version, instead of snapping.
4. **Idle behavior:** after 3 seconds without input, the target wanders to random points, so he looks around on his own.
5. **Blinking** is integrated into the same render loop (a scale factor on the pupils), so blinks and gaze never fight over the same transform.
6. The head itself gets a small 3D tilt (`rotateX`/`rotateY` plus translation) layered under the pupil motion, which sells the direction change. `prefers-reduced-motion` disables the tilt for accessibility.

### The voice system

- **Input:** the Web Speech API (`SpeechRecognition`) in continuous mode, running a tiny two-state machine: *waiting for the wake word* → *armed for one command* → back to waiting. Recognition restarts itself when the browser ends a session, and a blocked microphone degrades gracefully to typed commands.
- **Output:** `speechSynthesis` with a selected US English voice, rate 0.88 and pitch 0.82, tuned toward Baymax's calm delivery. (It's a tuned browser voice, an approximation of the character, not the film actor's voice.)

### The command engine

A rule-based intent router (`doWork()` in `index.html`) parses the transcript, strips the wake word, and dispatches: time/date, site opening, web search, note capture, jokes, capability listing, background/expand control, and conversation basics. Unknown intents get an honest "I can't do that yet" plus an offer to search, never a fake answer.

### Architecture

Everything lives in **one HTML file** (~29 KB, photo embedded as base64): markup, styles, gaze engine, voice system, and command router. That is a deliberate constraint, the whole product ships as a single artifact you can open, share, or host anywhere with zero setup.

## Run it

1. Download or clone this repo.
2. Open `index.html` in **Chrome** (voice input requires Chrome and microphone permission).
3. Click **Talk to Baymax**, allow the microphone, and say: *"Baymax, what time is it?"*

No microphone? Type any command in the box and Baymax will still do the work and speak the answer.

## Roadmap

- **True head turns:** the full video-frame pipeline from the original blog post, generating a look-around clip of the character, extracting transparent frames, tracking the eyes per frame, and picking frames from a direction grid.
- **More intents:** timers, weather, and reading a daily brief aloud.
- **Persistent memory:** notes saved across sessions.
- **A real voice model** to replace the tuned browser voice.

## Tech

`HTML` · `CSS (3D transforms, custom properties)` · `Vanilla JavaScript` · `Web Speech API (SpeechRecognition + speechSynthesis)` · `requestAnimationFrame` render loop

## Credits

- Gaze interaction inspired by Avi Vashishta's [robot hero](https://www.avivashishta.com/blog/robot-hero-follows-your-cursor-media-io).
- Character photo provided by the author. Baymax is a Disney character; this is a personal, non-commercial project.
