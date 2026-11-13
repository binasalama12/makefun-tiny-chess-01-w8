# Bright Falcon Chess — a deterministic 4 KB engine

An original tiny chess engine written for the make.fun
**tiny-chess-01** competition. The whole engine is `main.py`:
a single Python file, standard library only, that talks to the
competition environment at `MAKEFUN_ENV_URL` and plays every game
of the gauntlet it is served.

## Design

- **Board**: 0x88 mailbox board in a flat 128-slot list; pieces are
  small integers (type in the low bits, colour in the high bit).
- **Move generation**: pseudo-legal generation with offset tables
  per piece type, sliding rays for bishop/rook/queen, pawn pushes,
  captures, en passant, promotions and both castles; legality is
  confirmed by making the move and testing the own king.
- **Search**: iterative-deepening negamax with alpha-beta windows,
  a Zobrist transposition table, null-move pruning, late-move
  reductions and killer-move ordering. The quiescence search only
  extends captures. Everything is budgeted in **nodes, never
  time**, so the same position always produces the same move.
- **Evaluation**: material plus formula-generated piece-square
  tables, and an endgame king table blended in as non-pawn
  material disappears.

## This variant

Bright Falcon believes in pawns and patience. Its pawn is the
most valuable in the family (110 points), its null-move pruning
is the deepest (the reduced search runs four plies shallower
instead of three, so only clearly overwhelming positions are
trusted), and its transposition table is allowed to grow to
800,000 entries, half again the family norm. Like the stork it
keeps a **pawn shield** around its king — up to eight points per
friendly pawn on the king's neighbouring squares, faded by
remaining material — and like the stork it skips the bishop-pair
bonus entirely. Knight squares use the flattest centrality curve
in the family (5c − 16). Node schedule: 50k / 27k / 13.5k, with
its own seeded Zobrist keys. Locally it passes the family's
tactical suite and sweeps the kit starter.

## Files

- `main.py` — the engine (the readable source is packed into the
  file as an lzma + base85 payload under a two-line unpacker, a
  format the rules explicitly allow; both parts count toward the
  4,096-byte cap).
- `LICENSE` — MIT.
- `.makefun/owner.json` — the entrant wallet for this entry.

## Running

```sh
MAKEFUN_ENV_URL=http://127.0.0.1:PORT python3 main.py
```

The bot loops on `POST /game` and answers `POST /move` until the
environment reports the trial done.
