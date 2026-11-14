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

## Design notes

Representation: a 0x88 board in a flat 128-slot list (piece codes 1-6 for
white, 9-14 for black), incremental material/PST sums for both game phases,
and a Zobrist key maintained move by move (pieces, castling rights, en
passant file, side to move) so the transposition table and the repetition
counter never rescan the board. Move generation is pseudo-legal with a
post-make legality test on the mover's king; castling is generated only
through unattacked, empty transit squares.

Search: root iterative deepening re-searches the previous best move first
and stops on the deterministic node budget (120,000 nodes), never on a
clock, which keeps every trial byte-for-byte replayable. Inside, negamax
alpha-beta adds a bound-flagged transposition table, null-move pruning
(gated on a non-pawn material presence and on the static eval reaching
beta), late-move reductions with PVS re-searches, check extensions, and
killer-move ordering behind the TT move and MVV-LVA captures. Quiescence
covers captures, promotions and all check evasions, with repetition and
50-move draws scored as 0 in search.

Evaluation: tapered material + piece-square tables — middlegame and
endgame tables are built from one packed string by mirroring, the endgame
king gets its own table, and the phase weight is the remaining non-pawn
material. On top sit a bishop-pair bonus and a doubled-pawn penalty per
extra pawn on a file.

License: MIT (see LICENSE).

## Practice results

Local practice with the kit's harness before submission: **Elo 1654**
against the house gauntlet, with perft verified correct (start position
depth 2 = 400 nodes, matching the known value), so the move generator
underneath the search is sound. The packed `main.py` produced from this
source is 3,440 bytes, inside the 4,096-byte cap.
