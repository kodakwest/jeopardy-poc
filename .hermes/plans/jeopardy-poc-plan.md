# Plan: Jeopardy POC — Firebase AI Logic + Firestore

## Objective
Build a working Jeopardy game board as a single-page app hosted on Firebase Hosting, using Firebase AI Logic (Gemini Developer API) for clue generation and Firestore for game state persistence.

## Stack
- **Hosting:** Firebase Hosting (static files)
- **AI:** Firebase AI Logic SDK (`firebase/ai`) — Gemini Developer API (free tier, key stays server-side)
- **DB:** Cloud Firestore (game state, scores, answered questions)
- **Auth:** Firebase Anonymous Auth (for POC)
- **Frontend:** Vanilla JS, modular Firebase SDK (v10+), CSS Grid board

## What's Already Done
- Firebase project `jeopardy-poc` created ✅
- Hosting deployed at https://jeopardy-poc.web.app ✅
- Anonymous Auth enabled ✅
- Firestore enabled ✅
- AI Logic enabled (Gemini Developer API provider) ✅
- Initial HTML exists at `/mnt/s/Projects/jeopardy-poc/public/index.html` (uses compat SDK, needs rewrite)

## What Codex Needs to Build

### 1. Rewrite `public/index.html`
Replace the compat SDK approach with **modular Firebase SDK v10+** using `<script type="module">` and importmap for CDN loading:

```html
<script type="importmap">
{
  "imports": {
    "firebase/app": "https://www.gstatic.com/firebasejs/12.13.0/firebase-app.js",
    "firebase/ai": "https://www.gstatic.com/firebasejs/12.13.0/firebase-ai.js",
    "firebase/firestore": "https://www.gstatic.com/firebasejs/12.13.0/firebase-firestore.js",
    "firebase/auth": "https://www.gstatic.com/firebasejs/12.13.0/firebase-auth.js"
  }
}
</script>
```

### 2. Firebase AI Logic Integration
Use `firebase/ai` SDK to call Gemini for clue generation:

```js
import { getAI, getGenerativeModel, GoogleAIBackend } from "firebase/ai";
import { initializeApp } from "firebase/app";

const app = initializeApp(firebaseConfig);
const ai = getAI(app, { backend: new GoogleAIBackend() });
const model = getGenerativeModel(ai, { model: "gemini-3-flash-preview" });

// Generate structured JSON for 6 categories x 5 clues
const result = await model.generateContent({
  contents: [{ role: "user", parts: [{ text: prompt }] }],
  generationConfig: {
    temperature: 1.0,
    maxOutputTokens: 4096,
    responseMimeType: "application/json"
  }
});
```

### 3. Firestore Game State
Store/load game state using `firebase/firestore`:
- `games/{doc}` — categories, clues, scores, answered questions, timestamps
- `getDoc`, `setDoc`, `updateDoc`, `serverTimestamp`
- Query most recent game by `uid` ordered by `createdAt desc`

### 4. Jeopardy Board UI
Keep the current design dark theme (navy #0a0a2e, gold #ffd700 accents):
- 6 category headers across top
- 5 clue rows (values: $200–$1000)
- Click clue → modal with question → show answer → score buttons
- Score tracking
- "New Game" button to regenerate

### 5. File Changes
- **MODIFY:** `public/index.html` — complete rewrite using modular SDK
- **KEEP:** `firebase.json`, `.firebaserc`, `firestore.rules` (already correct)
- **REMOVE:** `functions/` directory (not needed — AI Logic handles proxying)
- **DEPLOY:** `firebase deploy --only hosting` after changes

## Security
- Firestore rules: `allow read, write: if request.auth != null` (dev mode)
- Firebase App Check can be added later for production
- AI Logic key stays server-side, never in client code

## Edge Cases
- Loading state while generating clues (Gemini takes 5-10s)
- Error state if AI generation fails
- Empty board state (no game loaded)
- Already-answered clues should be visually grayed out and non-clickable
- Modal should handle rapid clicks (debounce)
- Score persistence across page refreshes

## Out of Scope (for POC)
- Multiplayer / buzzer system
- Category editing
- Question difficulty tuning
- Analytics
- Mobile responsive refinements (basic responsive exists)
