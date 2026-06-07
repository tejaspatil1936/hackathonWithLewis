# Sudoku Game

A browser-based Sudoku game built with plain HTML, CSS, and JavaScript — no frameworks, no build step, no backend.

## Overview

This repository contains a single-page Sudoku game (`Sudoku_Game/`). It generates a valid, fully-solved 9x9 grid at runtime using a randomized backtracking algorithm, then removes a set number of cells to produce the puzzle the player solves. The game runs entirely in the browser: puzzle generation, validation, timing, and save/resume are all handled client-side with `localStorage`, with no server component.

A distinguishing feature is a built-in idle timer: if the player goes 7 seconds without selecting a cell or entering a number, the entire board is silently regenerated into a brand-new puzzle — a playful, hackathon-style twist on the classic game.

## Features

- **Procedural puzzle generation** — a recursive backtracking solver (`sudokuCreate`) fills a 9x9 grid so that every row, column, and 3x3 box contains 1–9 exactly once.
- **Six difficulty levels** (Easy, Medium, Hard, Very hard, Insane, Inhuman) that control how many cells are cleared from the solved grid before the puzzle is presented.
- **Cell and conflict highlighting** — selecting a cell highlights its row, column, and box; entering a number that already exists in that row/column/box flags the conflicting cells with an error animation.
- **Idle shuffle** — a 7-second inactivity timer regenerates the entire puzzle if the player pauses too long mid-game.
- **Pause / resume** with a running timer, plus a win screen that reports total solve time.
- **Save and continue** — player name, difficulty, elapsed time, and grid state are persisted to `localStorage` so a game can be resumed after closing the tab.
- **Dark / light theme toggle** with the preference stored in `localStorage` and synced to the mobile browser's status-bar color via the `theme-color` meta tag.
- **Random taunt messages** — a bank of playful messages ("Sure about that number?", "That's an odd pick.") pops up with a notification sound after each number entry.
- **Responsive layout** with a dedicated breakpoint (800px) that shrinks the grid and controls for mobile screens.

## Tech Stack

**Frontend**
- HTML5, CSS3 (custom properties for theming, CSS Grid and Flexbox for layout)
- Vanilla JavaScript (ES6)
- [Boxicons](https://boxicons.com/) and Google Fonts (Potta One, Poppins), loaded via CDN `<link>` tags

**Other**
- Browser `localStorage` for game-state and preference persistence — no backend, database, or build tooling is used.

## Architecture

The game is split into three plain scripts loaded in order from `index.html`:

| File | Responsibility |
|---|---|
| `static/js/constant.js` | Shared constants: grid size, box size, the list of difficulty names, and the number of cells removed per difficulty level. |
| `static/js/sudoku.js` | Pure puzzle logic — grid creation, row/column/box safety checks, the recursive generator (`sudokuCreate`), a completeness/validity checker (`sudokuCheck`), and `removeCells` / `sudokuGen`, which carve a puzzle out of a completed grid. |
| `static/js/app.js` | DOM and UI layer — screen transitions (start / game / pause / result), cell selection and input handling, the timer, dark-mode toggle, `localStorage` save/load, and the idle-shuffle timer. |

The puzzle-generation logic in `sudoku.js` has no DOM dependency and could be reused or unit-tested independently of the UI layer.

## Project Structure

```
Sudoku_Game/
├── index.html                  # Single-page markup for all four screens
├── app.css                     # Layout, theming (CSS variables), animations, responsive rules
├── README.md
├── AudioFile/
│   └── notification_tone.wav   # Sound played alongside taunt messages
└── static/
    ├── images/                 # Favicon and light/dark background art
    └── js/
        ├── constant.js         # Game constants and difficulty levels
        ├── sudoku.js           # Puzzle generation and validation
        └── app.js              # UI logic, event handlers, persistence
```

## Getting Started

### Prerequisites

- Any modern web browser. No Node.js, package manager, or build tool is required.

### Run locally

Because the game only uses relative paths for its own assets, you can open it directly:

```bash
cd Sudoku_Game
xdg-open index.html   # or just double-click index.html
```

If your browser restricts local file access (e.g. for audio playback), serve the folder instead:

```bash
cd Sudoku_Game
python3 -m http.server 8000
# then open http://localhost:8000
```

### Build

No build step exists or is needed — the app is served as static files as-is.

## Usage

1. Enter a player name (required before starting).
2. Click the difficulty button to cycle through the six levels.
3. Click **New Game** to generate a puzzle, or **Continue** to resume a previously saved game.
4. Click an empty cell, then click a number (1–9) to fill it, or **X** to clear it.
5. Use the pause button to stop the timer; **Resume** or **New Game** from the pause screen to continue.
6. Filling the grid with a valid, conflict-free solution ends the game and shows the elapsed time.

## Design Decisions

- **Randomized backtracking over a fixed template.** `sudokuCreate` shuffles the candidate digits (`shuffleArray`) before trying each one at the first unassigned cell, so repeated calls produce different completed grids rather than always filling the same pattern.
- **Difficulty as a cell-removal count, not a solver-based rating.** Each difficulty level in `constant.js` maps to a number of cells to blank out (29 for Easy up to 74 for Inhuman) rather than to a measure of solving technique required — a simple approach that keeps generation fast but means puzzle difficulty is approximate.
- **Client-only persistence.** All state (grid, timer, player name, theme) lives in `localStorage`, avoiding any need for accounts or a backend for a single-player, single-device game.
- **Idle-triggered shuffle as a game mechanic.** Rather than only regenerating puzzles on request, `startShuffleTimer`/`shuffleBoard` reset the board automatically after 7 seconds of inactivity, turning idle time into part of the challenge.

## Future Improvements

- `sudokuGen`'s `removeCells` copies the grid with `[...grid]`, which only clones the outer array — the row arrays themselves are shared, so the "solved" grid (`su.original`) ends up mutated to match the puzzle once cells are cleared. It is saved to `localStorage` but never read back, so the original solution is effectively lost; win detection instead re-validates the completed grid's own row/column/box rules via `sudokuCheck`.
- The generator does not check for a unique solution, so puzzles at higher difficulty levels (more cells removed) can theoretically admit more than one valid completion.
- `app.js` contains commented-out earlier versions of `initNumberInputEvent` and `initCellsEvent` left in place alongside the active implementations; removing the dead code would simplify the file.
- There is no hint system or ability to check a single cell without triggering the full row/column/box conflict scan.
