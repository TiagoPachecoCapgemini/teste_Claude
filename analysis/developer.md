────────────────────────────────────────────────────────
TECHNICAL PROPOSAL – “Plants vs Zombies Minesweeper”
────────────────────────────────────────────────────────
Goal: deliver a single-player, browser-based Minesweeper whose visuals (tiles, flags, win/lose screens, sounds) are re-skinned with Plants vs Zombies (PvZ) assets; no multiplayer, no monetisation, no leaderboards.

========================================================
1. High-Level Architecture
========================================================
Front-End SPA (React + TypeScript)
  • Game engine & UI live completely on the client; no server round-trip is
    required during play → instant response <200 ms (NFR1.2).
  • Packaged as static files (HTML/CSS/JS/Assets) – can be hosted on any CDN
    or simple web host (S3 + CloudFront, Netlify, Vercel, GitHub Pages).

Optional Thin Back-End  (Node/Express, serverless or container)
  • Only required if we later add telemetry or remote configuration.
  • For now it only exposes a GET /config endpoint so we satisfy “API / Endpoints”
    requirement without over-engineering (zero data persists).

Build/Tooling
  • Vite or Create-React-App for local dev and bundling.
  • eslint, prettier, jest, react-testing-library for quality.

Assets
  • Store PvZ spritesheets & audio under /public/assets.
  • Use CSS variables / theming to swap classic Minesweeper colours for PvZ ones.

========================================================
2. Required Components / Modules
========================================================
1. GameEngine.ts
   - Board generation (first click safe, flood-fill reveal).
   - Core actions: revealCell, toggleFlag, checkWin.
2. GameContext.tsx
   - React Context holding game state (board, minesLeft, status).
3. BoardGrid.tsx
   - Renders board as CSS grid; each Cell delegates to Tile.tsx.
4. Tile.tsx
   - Handles user interaction (left-click reveal, right-click flag).
   - Visual states: unrevealed (grass), flagged (PvZ shovel flag), revealed
     number (1–8 = different plant icons), mine (basic zombie head).
5. HUD.tsx
   - Mine/flag counter, timer, restart button (Crazy Dave icon).
6. ModalWinLoss.tsx
   - Themed overlay: win = “Plants Won 🎉”, loss = “Zombies Ate Your Brains 💀”.
7. AudioManager.ts
   - Plays SFX (reveal, flag, explosion, victory).
8. ConfigService.ts
   - GET /config => returns JSON with default rows/cols/mines, asset URLs.
9. Unit tests (GameEngine logic) & component tests.

========================================================
3. Suggested Data Model (in-memory)
========================================================
type Cell = {
  row: number;
  col: number;
  hasMine: boolean;
  adjacentMines: number;   // 0-8
  isRevealed: boolean;
  isFlagged: boolean;
};

type GameState = {
  status: 'idle' | 'playing' | 'won' | 'lost';
  board: Cell[][];
  rows: number;
  cols: number;
  totalMines: number;
  flagsLeft: number;
  startedAt?: number;
  finishedAt?: number;
};

No persistence → all data lives in React state.  
If we ever need to save best times, we can serialise to localStorage.

========================================================
4. APIs / Endpoints
========================================================
Purely client-side public API (for UI ↔ engine)
  • startGame({ rows, cols, mines }): GameState
  • revealCell(row, col): GameState
  • toggleFlag(row, col): GameState
  • resetGame(): GameState
  • getGameState(): GameState

Optional HTTP (Node/Express, can be serverless)
GET /config
  Response: { rows:number, cols:number, mines:number, assetsBaseUrl:string }
  Purpose: allows tuning difficulty without redeploying client.

(Everything else—scores, auth—remains out of scope per PO.)

========================================================
5. Implementation Plan (4 short sprints)
========================================================
Sprint 0 – Setup (½ week)
  • Repo, CI/CD, tooling, skeleton React app, routing placeholder.
  • Decide asset list, start obtaining/creating PvZ-style graphics (risk: licence).

Sprint 1 – Core Gameplay (1 week)
  • Implement GameEngine with tests.
  • Render basic board grid & tile states (text numbers, no art yet).
  • Ensure AC1-AC4.

Sprint 2 – Flagging + Win/Loss + UX polish (1 week)
  • Flag logic, HUD counters, timer.
  • Win/loss modal, restart, keyboard accessibility.
  • Mobile responsive layout.
  • Meet AC5-AC6, NFR2.

Sprint 3 – Theming & Audio (1 week)
  • Integrate PvZ sprites, CSS theming, sound effects.
  • Final performance pass (<5 s load, <200 ms interaction).
  • Cross-browser tests, PWA manifest if desired.
  • Achieve AC7, NFR1 & NFR3.

Release / UAT (½ week)
  • Stakeholder demo, bug fixing, deploy to production hosting.

========================================================
6. Technical Risks & Mitigations
========================================================
1. Asset Licensing
   Risk: PvZ IP is owned by EA/PopCap; unlicensed use may be blocked.
   Mitigation: verify licence or create original “inspired” art.

2. Performance on Low-End Mobiles
   Mitigation: avoid heavy canvas; use CSS sprites; lazy-load audio.

3. First-click Safety & Flood Fill Recursion Depth
   Mitigation: iterative flood algorithm to prevent stack overflow on large boards.

4. Right-Click Handling on Mobile (flagging)
   Mitigation: long-press or separate flag mode toggle.

5. Browser Variances (audio autoplay policies)
   Mitigation: central AudioManager with user-gesture unlock.

6. Scope Creep (leaderboards, extra modes)
   Mitigation: clear backlog boundaries; architecture leaves room but not built now.

========================================================
Summary
========================================================
A lightweight React SPA backed by an optional config endpoint fulfils every functional and non-functional requirement while staying highly maintainable and scalable (we can later add larger grids, save games, leaderboards or multiplayer without rewriting the core). The plan fits comfortably into ~4 weeks of work for a small team, keeping risk low and avoiding unnecessary complexity.