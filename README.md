# Jeopardy POC

**Demo Link:** [https://jeopardy-poc.web.app](https://jeopardy-poc.web.app)

## What it is
Jeopardy POC is an AI-powered Jeopardy game board built as a single-page web application. It leverages the latest Firebase technologies and Gemini AI to dynamically generate 30 unique Jeopardy clues across 6 distinct categories for every new game. Players can click on clues, see the question, optionally reveal the answer, and mark themselves right or wrong—updating their score dynamically.

## Stack
- **Hosting:** Firebase Hosting
- **Database:** Firebase Firestore (persists game state, scores, and answered clues)
- **AI Logic:** Firebase AI Logic / Gemini Developer API (`gemini-3-flash-preview`)
- **Auth:** Firebase Anonymous Authentication
- **Frontend:** Vanilla HTML/CSS/JS (Single `public/index.html` file using modular Firebase SDK v12.13.0 via importmap)

## How it works
1. **Authentication:** When a user opens the app, they are signed in anonymously using Firebase Auth.
2. **Game Initialization:** The app checks Firestore for an existing game session for that user. If none exists, or the user clicks "New Game," the app prompts the AI to generate a new board.
3. **AI Clue Generation:** The app calls the `gemini-3-flash-preview` model via the `firebase/ai` SDK. The AI is prompted to return exactly 6 categories, each with 5 clues of escalating values ($200 to $1000). The model returns structured JSON with the clue text and the correct "Jeopardy format" answer (e.g., "What is...").
4. **Gameplay:**
   - The user clicks on a clue value (e.g., $400).
   - A modal displays the clue.
   - The user can click "Show Answer".
   - The user clicks "Correct" or "Wrong" based on their guess.
   - The result is persisted to Firestore, which stores the user's score and a list of answered clues.
   - The UI updates to show the updated score and visually disables the answered clue.

## Architecture
- **Single Page Application:** The frontend is entirely contained in `public/index.html`.
- **No Build Step:** The app uses native ES modules (`<script type="module">`) and an import map to load Firebase SDKs directly from `gstatic.com`.
- **Client-Side Orchestration:** The client handles AI generation directly via the Firebase AI SDK and writes the generated board structure to Firestore.

## Local Dev Setup
1. Clone the repository:
   ```bash
   git clone https://github.com/kodakwest/jeopardy-poc.git
   cd jeopardy-poc
   ```
2. Install the Firebase CLI if you haven't already:
   ```bash
   npm install -g firebase-tools
   ```
3. Login to Firebase:
   ```bash
   firebase login
   ```
4. Run the local emulator/dev server:
   ```bash
   firebase serve
   ```
   *(Or just open `public/index.html` via any local static file server, like `npx serve public` or VS Code Live Server).*

## Deployment Commands
To deploy the application to Firebase Hosting:
```bash
firebase deploy --only hosting
```

To deploy Firestore rules and indexes:
```bash
firebase deploy --only firestore
```

---

## Next Steps Roadmap
The current Proof of Concept successfully demonstrates AI integration and core gameplay loop. To transition to a production-ready application, here is the recommended roadmap prioritized by Impact vs. Effort.

### High Impact, Low/Medium Effort
- **Firebase App Check for Security:**
  - *Impact:* Prevents unauthorized abuse of the Firebase AI SDK and database endpoints.
  - *Effort:* Low. Integrating App Check with reCAPTCHA (web) is straightforward and crucial before exposing the Gemini API to the public.
- **Mobile Refinements:**
  - *Impact:* Huge. Most users access web apps via mobile.
  - *Effort:* Low to Medium. Improve the responsive CSS for smaller screens, ensure the modal is scrollable, and fix touch targets.
- **Category Editing / Regeneration:**
  - *Impact:* Allows users to curate their game board if they don't like the AI's suggestions.
  - *Effort:* Medium. Add a "regenerate" button next to category headers to swap out specific columns.

### High Impact, High Effort
- **Multiplayer & Buzzer System:**
  - *Impact:* Transforms the app from a solo trivia tool into a party game.
  - *Effort:* High. Requires real-time presence, game room codes/lobbies, synchronization of the active clue, and strict latency handling for the buzzer logic.
- **Leaderboard:**
  - *Impact:* Adds a competitive edge and encourages replayability.
  - *Effort:* Medium to High. Requires moving from Anonymous Auth to permanent accounts (Google/Email) and building global/friends leaderboard views.

### Medium Impact, Medium Effort
- **Clue Difficulty Tuning / Progressive Difficulty:**
  - *Impact:* Improves the quality of AI-generated clues.
  - *Effort:* Medium. Refine the Gemini prompt to ensure $200 clues are trivial and $1000 clues are expert-level. Possibly add difficulty modes (Easy, Normal, Hard) that adjust the prompt instructions.
- **Question Import from J! Archive:**
  - *Impact:* Provides authentic, broadcast-accurate clues instead of (or in addition to) AI-generated ones.
  - *Effort:* Medium. Would require a backend scraper/importer, storing the archive in Firestore, and adding a toggle for users to play "AI Mode" vs "Archive Mode."
