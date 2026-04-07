# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

A Progressive Web App (PWA) for iPhone to practice English from the Language Hub B2+/C1 course with teacher Julieta Ayub. All app logic, CSS, and data live inline in `index.html`. No build step, no dependencies.

## Files

- `index.html` — The entire app: 122 questions, quiz engine, spaced repetition, XP/streak system, 5 views, AI generator
- `manifest.json` — PWA manifest (name: "EnglishHub C1", theme: #58CC02)
- `sw.js` — Service worker for offline caching and push notifications
- `icons/` — App icons (not yet created; app works without them)

## Serving locally for iPhone testing

```bash
# Python (built-in)
python3 -m http.server 8080

# Then open http://YOUR_LOCAL_IP:8080 in Safari on iPhone
# Find your local IP: ifconfig | grep "inet 192"
```

To install on iPhone: Safari → Share button → "Add to Home Screen"

## Key app architecture

**State** — All state in `localStorage` under key `eh_state`. Structure: `{streak, lastDate, xp, totalAnswered, totalCorrect, todayCount, todayDate, goalCelebrated, weights:{qId:number}, unitProgress:{unitId:{c,t}}, notifOn}`. Loaded/saved via `loadState()` / `save()`.

**Question bank** — Array `Q` with 122 questions. Each question: `{id, u(unit 1-17), t('mcq'|'fill'), q(text), o(options array, MCQ only), a(answer index or string), e(explanation)}`.

**Spaced repetition** — Each question has a weight (default 1.0). Correct answer: `weight *= 0.5` (min 0.1). Wrong answer: `weight *= 2.5` (max 10). `pickQuestion()` does weighted random selection.

**Streak logic** — Increments only on the FIRST answer of a new day. If `lastDate` was yesterday: streak++. If more than 1 day ago: streak resets to 1.

**Daily goal** — 10 questions/day. At question 10, confetti + celebration modal fires.

**AI generation** — Calls `https://api.anthropic.com/v1/messages` directly from the browser (requires `anthropic-dangerous-direct-browser-access: true` header). API key stored in `localStorage` under `eh_apikey`. Model: `claude-sonnet-4-20250514`.

**Views** — Five views toggled by `showView(name)`: `home`, `units`, `progress`, `settings`, `quiz`. Quiz view is special — exits back to home.

## Design tokens (CSS variables)

```
--green: #58CC02  --green-dark: #46A302
--blue: #1CB0F6   --red: #FF4B4B
--yellow: #FFC800  --purple: #CE82FF
--orange: #FF9500
--bg: #131f24  --card: #1f2f38
--border: #2d4a5a  --muted: #8fa3b0
```

## Adding questions

Add to the `Q` array in `index.html`. Follow the pattern:
```js
{id:123, u:UNIT_ID, t:'mcq'|'fill', q:'Question text',
 o:['A','B','C','D'],  // MCQ only
 a:INDEX_OR_STRING,    // 0-3 for MCQ, string for fill
 e:'Explanation shown after answer.'}
```
Fill answers support multiple acceptable answers separated by `/` (e.g., `a:'While/Although/Whereas'`).
