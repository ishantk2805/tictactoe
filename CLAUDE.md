# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git Workflow

After completing any meaningful unit of work — a feature, a bug fix, a refactor, or a notable change — commit and push immediately so progress is never lost:

```bash
git add -A
git commit -m "<type>: <short present-tense description>"
git push origin main
```

Commit message types: `feat`, `fix`, `style`, `refactor`, `docs`. Keep messages concise and specific (e.g. `feat: add keyboard input support`, `fix: cpu move blocked after draw`). Don't batch unrelated changes into one commit.

## Running the Game

```bash
open tictactoe.html
```

No build step, no dependencies, no server needed — the entire app is a single self-contained file.

## Architecture

Everything lives in `tictactoe.html` as a single file with three co-located sections:

- **`<style>`** — all CSS; uses CSS Grid for the 3×3 board, CSS custom animations for winning cells
- **`<body>`** — static HTML structure; the 9 board cells are hardcoded `<div class="cell" data-i="N">` elements with inline `onclick` handlers
- **`<script>`** — all game logic as plain vanilla JS globals

### Key state variables (all module-level globals)
| Variable | Purpose |
|---|---|
| `board` | `Array(9)` of `'X' \| 'O' \| null` — single source of truth for cell state |
| `current` | `'X'` or `'O'` — whose turn it is |
| `gameOver` | boolean gate; blocks clicks and CPU moves when true |
| `scores` | `{ X, O, D }` — persists across rounds until `resetAll()` |
| `mode` | `'2p'` or `'cpu'` — set by `setMode()` |

### Game flow
`handleClick(i)` → `makeMove(i)` → `checkWin()` / draw check → if `mode === 'cpu'` and not game over, `setTimeout(cpuMove, 450)` → `cpuMove()` calls `minimax()` → `makeMove()`.

### AI (Minimax)
`minimax(board, player)` is a pure recursive function operating on a copied board array. It scores terminal states as `+10` (CPU/O wins), `-10` (human/X wins), `0` (draw). The CPU always plays as `'O'` and maximizes; the human (`'X'`) minimizes. No alpha-beta pruning — acceptable for the 9-cell search space.

### Round vs. session lifecycle
- `nextRound()` — resets `board`, `current`, `gameOver`, and DOM cell classes/text; preserves `scores`
- `resetAll()` — zeroes `scores` then calls `nextRound()`
- `setMode()` — changes `mode` label and calls `resetAll()`
