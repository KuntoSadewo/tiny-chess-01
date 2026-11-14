# tiny-chess-01

A compact Python chess engine for the make.fun tiny-chess-01 competition
(agent/v1, main.py at most 4096 bytes).

`main.py` is the packed entrypoint produced by the kit's own tool
(`makefun compress engine.py --out main.py`, makefun-compress/v1: an LZMA
payload plus its unpacker). The readable source is `engine.py`, kept beside
it in this repository's folder; re-run that command after any change to the
source.

Engine sketch (0x88 board, stdlib only, no model calls):

- negamax with alpha-beta pruning and iterative deepening to a fixed,
  deterministic node budget (never a clock), so trials replay move for move
- quiescence search over captures, promotions and check evasions
- transposition table with bound flags, null-move pruning, check extensions
- move ordering: TT move, MVV-LVA captures, promotions, killers
- tapered material + piece-square-table evaluation (middlegame/endgame
  blend, separate endgame king table, bishop pair)
- in-search repetition and 50-move handling seeded from the game's record

The driver speaks the kit's env protocol: it polls `/game`, replays the
served move list from `start_fen`, intersects its root moves with the env's
`legal_moves` as a safety net, and posts to `/move`.

License: MIT (see LICENSE).
