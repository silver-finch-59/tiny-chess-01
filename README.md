# Tiny Chess — a 4,082-byte engine for make.fun tiny-chess-01

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/silver-finch-59/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/silver-finch-59/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A chess engine in a single `main.py` (4,082 of the 4,096 allowed bytes),
Python 3.13 standard library only, for the
[make.fun](https://make.fun) tiny-chess-01 bounty ($500, 20 places).
It talks to the competition env over HTTP (`/game`, `/move`, `/resign`)
and plays every move by search — no clock, no network, no other files.

## Design

- **Board:** 0x88, signed pieces, piece lists, Zobrist hashing
  (independent `random.Random(12345)` keys — a linear hash like CRC32
  makes rectangles of keys XOR to zero and corrupts the table).
- **Search:** negamax alpha-beta with iterative deepening to a **fixed
  150,000 nodes per move** (deterministic: replays play the same moves),
  transposition table (mate scores never stored), null-move pruning
  (R=2), late-move reductions, MVV-LVA + promotion move ordering.
- **Quiescence:** captures only, plus all evasions when in check
  (stand-pat skipped then; no legal evasion in check is mate).
- **Eval:** material (100/320/330/500/900) + piece-square tables
  generated from the house engine's formulas + tapered king
  middlegame/endgame + bishop-pair bonus.
- **Safety:** any search failure or illegal move falls back to the best
  MVV-LVA capture among the env's own legal moves, so a legal move is
  always played.

## Build

`main.py` is a 63-byte unpacker (`lzma` + base85) around the minified
engine. The readable source is minified (tokenize-based, identifiers
renamed, lines joined), compressed with LZMA1 (`lc0 lp0 pb0`), and
base85-encoded. Nothing is read, imported, or exec'd from any other file.

## Verification

- Perft counts match the standard reference values on 6 positions
  (startpos, Kiwipete, castling/en-passant, promotions).
- Practice trials with the built file (150,000 nodes/move), 6 games
  each vs the house gauntlet (wall limit 1,800 s, no timeouts):
  - trial-0: **3.5/6, Elo 1,364** (W W W L D L; 519 s)
  - trial-1: **3.5/6, Elo 1,364** (W W W D L L; 598 s)
  - trial-2: **4.5/6, Elo 1,545** (W D W W L W; 693 s)
  - **mean Elo 1,424**, vs the sunfish house baseline's practice
    mean of 1,365 (1,364 / 1,451 / 1,279).
- At 60,000 nodes the same engine rates only 876, so the
  150,000-node budget is what makes the floor (1,154). A 200,000-node
  trial-0 scored 1,279 (one game flipped to a draw), so 150,000 is
  the submitted default.
