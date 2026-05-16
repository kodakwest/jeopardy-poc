# AGENTS.md — Jeopardy POC

## Project Context
AI-powered Jeopardy game board built as a single-page app on Firebase.
- **Stack:** Firebase Hosting, Firestore, Firebase AI Logic (Gemini Developer API), Anonymous Auth
- **Live:** https://jeopardy-poc.web.app
- **Repo:** kodakwest/jeopardy-poc

## Design Conventions
- Dark theme: bg `#0a0a2e`, panels `#1a1a5e`, ink `#f2f0e8`
- Accent: Gold `#ffd700` (Jeopardy classic)
- Fonts: Inter (UI), system-ui fallback

## POC Current State
- Single `public/index.html` with modular Firebase SDK (v12.13.0 via importmap)
- Firebase AI Logic (`firebase/ai`) generates 6 categories x 5 clues via `gemini-3-flash-preview`
- Firestore persists game state (score, answered clues)
- 30 interactive clue cells with modal reveal + scoring

## Next Steps Needed
See Jules dispatch for full roadmap advice.

## Deployment
- `firebase deploy --only hosting` to push updates
- `firebase deploy --only firestore` for rules/indexes
