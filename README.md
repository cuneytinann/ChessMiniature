**[English](#chessminiature)** · **[Türkçe](#turkce)**

# ChessMiniature

A two-player chess arbiter in **1,897 bytes** of one HTML file, and its twin in **1,838 bytes**. No libraries, no build step, no server, no packer. Download a file, double-click, play.

Both files enforce the same rules. `index.html` stores the board as letters on sixty-four squares, the way a FEN does. `hexadecimal.html` stores it as numbers on a 0x88 board, the representation FideLite's engine uses. Put side by side, they are a small course in how few characters the rules of chess need, and in what a choice of representation costs.

Part of the [Golfstack](https://www.fidelite.art/) project.

## Play

| file | board | GitHub Pages | project site |
| --- | --- | --- | --- |
| `index.html` | letters | [ChessMiniature](https://cuneytinann.github.io/ChessMiniature/) | [L2_string_base.html](https://www.fidelite.art/builds/special/L2_string_base.html) |
| `hexadecimal.html` | numbers | [hexadecimal.html](https://cuneytinann.github.io/ChessMiniature/hexadecimal.html) | [L2_base.html](https://www.fidelite.art/builds/special/L2_base.html) |

On the project site both builds sit under `builds/special/`, as variants of the `L2` rules level.

Click a piece, click a destination. Legal targets turn amber, the selected square and then the last destination keep a green outline, and the board flips to the side to move after every ply.

## Read it in the browser

Nothing here is packed or minified by a tool. **Right-click → View Page Source** (`Ctrl` `U`, or `⌥` `⌘` `U` on macOS) shows the entire program, and that is the intended way to use this repository: open the page, play a few moves, then read the source that just refereed them.

The script is one comma-separated declaration chain in dependency order, broken so that one line holds one idea. Both files are 56 lines long and run in the same order, part by part: the page, the state and the start of `G` sit at the same height in both, and so do the verdict, the click handler and the first draw. In between they differ where the engines do. Line breaks carry no meaning: strip them from either script and the syntax tree is identical.

```
index.html         aliases → state → G → T → M → L → C → setup → d → A → S
hexadecimal.html   aliases → state → G → T → U → L → C → M → setup → d → A → S
```

Read across: geometry, attack, raw move, legality, castling rights, then the real move, setup, drawing, apply and judge, click. The raw move is `M` in `index.html` and `U` in `hexadecimal.html`. Only `hexadecimal.html` keeps the real move in a function of its own, its `M`; `index.html` makes it inside `A` (Lesson 3).

---

## What's in it

- **All piece movement**, derived from arithmetic. No direction tables, no offset arrays.
- **Full legality.** A move that leaves your own king in check is never accepted.
- **Castling**, both sides, with every condition: right still held, rook path clear, king not in check, king not crossing an attacked square, king not landing in check.
- **En passant**, including the case almost everyone gets wrong (Lesson 3).
- **Promotion with a picker.** Queen, rook, bishop, knight. The board locks until you choose.
- **Check, checkmate, stalemate**, told apart.
- **Insufficient material** to FIDE's reading (Lesson 4).
- **5-fold repetition** and the **75-move rule**, declared without a claim.
- **Status line**: corner coordinates, `C!` when the side to move is in check, `M=` the halfmove clock, `R=` how many times the current position has occurred. When the game ends, the line becomes the result code.

## What's not in it

- No clock.
- No draw offer and no draw claim, so no 3-fold claim and no 50-move claim. The counters run and are shown; only the automatic thresholds end the game.
- No detection of **blocked positions**, the other half of FIDE Article 5.2.2, where material is sufficient but the pawns have locked and mate is impossible anyway. FideLite's `L2` builds find those positions with a bitboard analysis of the pawn structure; here that detector was removed deliberately. The result code is therefore `IM` (insufficient material) rather than `DP` (dead position): the code names what is actually tested. A blocked position still ends as a draw, later, by repetition or by the 75-move rule.
- No bot, no undo, no FEN import or export, no PGN.

For the full arbiter — clock, draw offers and claims, resignation, flag fall, dead positions, fifteen result codes — see [fidelite.art](https://www.fidelite.art/).

## Result codes

| code | meaning |
| --- | --- |
| `W#` | White delivers mate |
| `B#` | Black delivers mate |
| `SM` | Stalemate |
| `IM` | Insufficient material |
| `5R` | 5-fold repetition |
| `75` | 75-move rule |

---

## Lesson 1 — The state is a FEN record

```js
b=[...'rnbqkbnrpppppppp'.padEnd(48,e=_='-')+'PPPPPPPPRNBQKBNR'],t=1,c=15,n=z=0,   // index.html
b=[],[...'dcebaecd99999999'+'0'.repeat(32)+'1111111154632645'].map((p,j)=>b[j*2-j%8]='0x'+p-0),t=8,c=15,e=8,n=0,   // hexadecimal.html
```

Both lines are a FEN, field for field: **piece placement, side to move, castling rights, en passant square, halfmove clock.** The fullmove number is the only field left out, because no rule reads it: the 50- and 75-move rules count plies, and repetition is decided by position, not by move number.

- **Letters.** Index 0 is a8, the order a FEN is written in. Uppercase is White, lowercase is Black, `-` is an empty square. White to move is `t=1`, and no en passant square is `-`. That letter is born inside the board: `padEnd(48,e=_='-')` fills the thirty-two empty squares, names the letter `_` and sets the en passant field to it, all in one argument, so `e` comes first in the chain instead of fourth. `n=z=0` starts the result code `z` along with the clock.
- **Numbers.** The board is 0x88: a square is `16 × rank + file`, index 0 is a1, and each rank is sixteen slots wide, of which only the left eight are squares. The sixty-four digits come in the same three pieces as the letters — sixteen for one side, thirty-two for the empty middle (`'0'.repeat(32)` here, `padEnd(48,…)` on the letter board), sixteen for the other — and `j*2-j%8` spreads them over that layout, leaving the right half of every row unwritten. Each digit is a piece code: `0` empty, the type in the low three bits, plus 8 for White (Lesson 5). The side to move `t` is that colour bit itself, 8 or 0, and an en passant square of `8` means none: square 8 is off the board.

Read the two lines against each other and the files line up while the ranks run the other way: the letters start from a8 like a FEN, the numbers from a1 like the 0x88 index. The numeric line keeps FEN's order of fields exactly.

FideLite's own builds write the white pawns and the empty middle as `10n**40n-10n**32n`, a BigInt evaluating to `99999999` followed by thirty-two zeros — six bytes shorter, and a full stop for anyone reading. Here it is spelled out on purpose. These files are meant to be read, and a board that has to be computed before it can be seen defeats the point of showing it.

The repetition key is the same four variables concatenated in both files:

```js
R=G[k=b+t+e+c]=-~G[k]     // letters: board, side to move, en passant square, castling rights
k=b+e+t+c                 // numbers: written by M; A stores it with R=G[M(i,f,y)]=-~G[k]
```

`-~undefined` is `1`, so a position seen for the first time needs no default. The table is `G` itself: a function is an object, so the move generator doubles as the repetition table and no variable is spent on it. The order of the fields is not decoration. Past the board they run together without a separator, and each order is chosen so that a key can only be read back one way; in the numeric key, `t` keeps `e` and `c` apart. On the letter board the key reads almost like a FEN when printed.

## Lesson 2 — Geometry is arithmetic

`G(i,f,m)` answers one question: can the piece on square `i` reach square `f`, ignoring everything but the board? With `m` set it answers for an attack instead of a move, which is how `T` uses it. The two boards derive the answer from different numbers.

The letter board starts from two, the file distance `h` and the rank distance `v`:

```js
h=a(i%8-f%8),v=a(y-(f>>3))     file and rank distance
d=h|v                          squares spanned by a line move: h|v is max(h,v) when h==0, v==0 or h==v
s=(f-i)/d                      the step vector — ±1, ±8, ±7, ±9 — from one division
h*v==2                         the knight: only 1×2 and 2×1 give 2
m|v|h^2?d<2:…                  the king: one square, or two along the rank as castling
r=f+=s>0||-2                   the castling rook's square, counted from the king's target: one on from g1 (true counts as 1), two back from c1; r keeps it for M
h*v==1                         a pawn capture: one file and one rank
y%6<2                          the pawn home ranks: y%6 is below 2 only on ranks 0, 1, 6 and 7, and a pawn never stands on 0 or 7
f%56<8                         the last rank, for either colour
f^8                            the en passant victim: XOR with 8 steps one rank back, in the right direction
i*9/8%2                        the colour of a square: i*9/8 is index plus rank, and its parity is the colour
```

The 0x88 board needs only one, the difference `d=f-i`. With sixteen slots to a rank every direction is a fixed difference — ±1 along a rank, ±16 along a file, ±15 and ±17 on the diagonals — so the step is a question of divisibility:

```js
d=f-i                          the difference, which on a 0x88 board names the direction by itself
x=d%15?d%16?d%17?1:17:16:15    the step: 15 or 17 on a diagonal, 16 on a file, 1 on a rank or on no line at all
s=d<0?-x:x                     the step with its sign
d*d%765%128==68                the knight: ±14, ±18, ±31 and ±33, squared and folded twice, land on 68, and no other difference does
~m+3%d?d==s:…                  the king: one step, or two along the rank as castling; the test is zero only for d=±2 outside an attack question
r=f+=s-!~s                     the castling rook's square, counted from the king's target: one on from g1, two back from c1; r keeps it for U
m^d<d*t                        a pawn moves forward: d<d*t means d>0 for White (t=8) and d<0 for Black (t=0); in an attack question the pawn belongs to the other side, and m^ turns it round
1&~x%d                         a pawn capture: odd only for d=±15 and ±17
i%96<24                        the pawn home ranks: true only on ranks 0, 1, 6 and 7
f%112>7                        not the last rank, for either colour; in U it tells a promotion apart
f^16                           the en passant victim: XOR with 16 steps one rank back
i%15%2                         the colour of a square: 16·rank+file leaves rank+file modulo 15, and its parity is the colour
w&8                            off the board: the right half of every row, never written, and skipped when d draws
```

Sliding pieces walk the line one step at a time with `S`, stopping at the target or at the first square that is not empty. The pawn's double step and castling use the same walk: two steps along the file for the pawn, the king's way to its rook for castling. Which pieces may walk which lines is where the two files first diverge, and it is worth reading both:

```js
(h*v?h==v&P<'R':P>'B')&S()     letters: B < Q < R in the alphabet, so P<'R' lets B and Q move
                               diagonally and P>'B' lets Q and R move straight
P+P*4%x&S()                    numbers: queen 3, rook 5, bishop 6; P+P*4%x is odd exactly when
                               the type may move along step x, and & keeps that last bit
```

The numeric test is worth checking by hand. Along a rank (`x=1`) it gives 3, 5 and 6; along a file (`x=16`) 15, 9 and 14; on a diagonal (`x=15` or `17`) 15 or 15, 10 or 8, and 15 or 13. The queen is odd everywhere, the rook only on straight lines, the bishop only on diagonals.

There is a second lesson in what stops the walk. On the letter board a read past the edge returns `undefined`, which is not `'-'`, so a walk on a non-line stops by itself and `&` is safe. On the 0x88 board the same holds by design. The walk asks `b[i]<1`; the right half of every row is never written, a step off the board in any direction lands in that gap or outside the array, and `undefined<1` is false. The edge stops a walk the way a piece does, even when `x` has fallen back to `1` for a difference that is no line at all. The old 8×8 numeric file tested emptiness with `!b[i]`, and `!undefined` is `true`, so it had to guard every walk with `&&`.

## Lesson 3 — Legality: play it, ask, take it back

Legality is not a second move generator. It is one trick:

- `T` asks the whole board whether a square is attacked — by default, the king of the side to move. Both boards read the side from `t`, so neither has to name it.
- `L(i,x)` tries the candidate move on a copy of the board, asks `T`, and puts the board back.
- It tries the move with the **raw move**, which only moves pieces, on a fresh copy — `[...b]` on the letter board, `b.slice()` on the numeric one — so `L` puts the board back with `b=q` and nothing else. The raw move is `M` on the letter board and `U` on the numeric one. What only a real move does — castling rights, the clock, the side to move, the en passant square, the key — is written once per real move: inside `A` on the letter board, in the numeric board's `M`, which only `A` calls. Either way, `A` is the only function that makes anything permanent.

The raw move takes the rook along when the king castles: a second raw move, from the rook's square to the square the king passed — `j(p)+d=='K2'&&M(r,Q)` on the letter board, a king two squares away read as one string, and `d-8||U(r,Q)` on the numeric one. It finds the rook where `G` left it: checking a castling move, `G` writes the rook's square to `r` (Lesson 2). So `L` has to run right before the raw move for the same move — and it always does, because the click handler asks `L` and only then calls `A`.

The dependency runs backwards in exactly two places, and both come from the rules, not from the code. `G` calls `T` because castling cannot be judged without asking whether the king stands in check or crosses an attacked square; the path to the rook is walked last, after both attack tests. The real move calls `L` because the en passant square may only be written if the capture is genuinely legal:

```js
e=M(i,f,y),t^=1,
b.some((p,x)=>p=='pP'[t]&&L(x,e))||(e=_)     // letters, in A
e=U(i,x,p),t^=8,
e=b.some((p,j)=>p==1+t&&L(j,e))?e:8          // numbers, in M
```

A pawn has just moved two squares. The naive version writes the en passant square at once. This one writes it provisionally, asks whether an enemy pawn can *legally* capture there, and reverts if not. Both boards do it the same way. The raw move returns the square the pawn jumped over, or the no-square value after any other move — `_` on the letter board, `8` on the numeric one — and that goes into `e` first. Then, with the side already switched, every pawn of the side now to move is asked: `'pP'[t]` on the letter board, code `1+t` on the numeric one. `G` turns down every pawn that is not beside the target, and when there is no square every question ends at once: `L` finds nothing to capture on `b['-']` or on square 8, which is off the board.

The difference never shows in the capture itself — an illegal capture is rejected by the move generator either way. It shows in the **repetition counter**, because `e` is part of the position key. FIDE treats two positions as the same only if the same moves are available in both, en passant included. Record a square nobody can use, and one position lands under two keys: a repetition draw fires late, or never.

The classic case: Black plays g7–g5 while the white pawn on f5 is pinned to its king on f1 by a rook on f8. `fxg6` would expose the king, so the capture is illegal, so the square is never recorded. The check costs **41 bytes** on the letter board and **36** on the numeric one, where the naive versions are just `e=M(i,f,y)` and `e=U(i,x,p)`.

The question is asked once per real move and never inside a trial, because the trial is the raw move and the raw move never touches `e`. The letter board used to ask it inside its `M`, after every move `M` wrote, trial moves included. That cost twice: `L` had to save and restore the en passant square and the clock around every trial, and perft at depth 4 or 5 slowed down sharply. Asking it in `A` made both go away (October 2026, see Verification).

## Lesson 4 — The rules everyone skips are cheap

Chess programs in code-golf collections routinely drop the halfmove clock, the repetition counter and the insufficient-material test as expensive bookkeeping. Measured in these files:

| rule | letters | bytes | numbers | bytes |
| --- | --- | --- | --- | --- |
| Halfmove clock | `n=P\|b[f]>_?0:n+1` | 16 | `n=!b[x]*b[i]&6&&n+1` | 19 |
| Repetition counter | `R=G[k=b+t+e+c]=-~G[k]` | 21 | `k=b+e+t+c` · `R=G[…]=-~G[k]` | 21 |
| Castling rights, all four | `C=i=>'20003001'[i%56]<<i/28` | 27 | `C=i=>'20003001'[i%112]<<i/56` | 28 |
| Insufficient material, both sides | `(m=W=0,b.map((p,i)=>(P=j(p))=='B'?m\|=i*9/8%2+1:W+=P>'N'?9:P>'M'),W*2+m<3)` | 73 | `(m=W=0,b.map((p,i)=>p&&(p%8>5?m\|=i%15%2+1:W+=p%8==4\|\|p%8-2&&9)),W*2+m<3)` | 72 |
| Every ending | `?'IM':R>4?'5R':n>149&&'75':T()?t?'B#':'W#':'SM'` | 47 | `?'IM':R>4?'5R':n>149&&'75':T()?t?'B#':'W#':'SM'` | 47 |
| **total** | | **184** | | **187** |

About a tenth of either file. The expensive part of a chess program was never the rulebook.

**The halfmove clock** on the numeric board is one product: `!b[x]` is 1 only when the target square is empty, and `b[i]&6` is 0 only for a pawn, so the clock runs on exactly when no pawn moves and nothing is captured. On the letter board the clock is written in `A`, from `P`: the click handler sets `P` while it checks the moving piece for a promotion, so `A` knows whether a pawn moved without asking again.

**Castling rights** are one string: the digit under a square of the first or last rank is the right that square forfeits when a piece leaves it or lands on it, shifted to the other colour's bits on the far rank — by `i/28` on the letter board, `i/56` on 0x88: 0 on one end of the array, 2 on the other. King and rook squares carry their bits; every other square yields `0`. Every move clears both of its squares: `c&=~C(i)&~C(f)` in `A` on the letter board, `c&=~C(i)&~C(x)` in `M` on the numeric one.

**Insufficient material** is one pass and one comparison. `m` collects the square colours of the bishops as bits (1 or 2; both colours make 3), written `i*9/8%2` on the letter board and `i%15%2` on 0x88. `W` counts a knight as 1 and any pawn, rook or queen as 9; kings add nothing. On the letter board `P>'N'?9:P>'M'` says exactly that: `P`, `Q` and `R` sort after `N`, `N` after `M`, and a king or the empty square `-` before both, so empty squares need no test of their own. Then `W*2+m<3` holds exactly for bare kings, a single knight, or bishops that all stand on one square colour — on either side. Two knights, bishop and knight, opposite-coloured bishops, or a knight on each side all come out at 3 or more, and play continues.

**Claims and declarations.** A draw that must be *claimed* needs a player to claim it, which needs a button. This build has no such layer, so it keeps only what an arbiter declares on its own: 5-fold repetition, the 75-move rule, insufficient material, mate and stalemate. You can still watch `R=` climb to 3 and `M=` pass 99 — under FIDE those two thresholds are claims, not declarations, so they do not end the game here.

## Lesson 5 — Letters or numbers

A letter says one thing: which piece. Colour has to be recovered from its case, and emptiness from a separate character, so almost every question asks for an extra comparison or a case conversion (`j` is `toUpperCase`). A number can be chosen so that the questions are already answered by its bits:

| code | 0 | 1 · 9 | 2 · 10 | 3 · 11 | 4 · 12 | 5 · 13 | 6 · 14 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| piece | empty | pawn | king | queen | knight | rook | bishop |

The low three bits are the type, `p&7`, and 8 is White: the first code of each pair is Black, the second White. These are the codes of FideLite's 0x88 engine, and the order is not alphabetical, it is functional. Queen 3, rook 5 and bishop 6 make the slider test of Lesson 2 work. The side to move `t` is the colour bit itself, 8 or 0, so the pawn of the side to move is `1+t`, its king `2+t`, a promoted piece `p|t`, and `p&8^t` picks the other side in `T`. `p&6` is zero only for a pawn, which is all the halfmove clock needs, and `p%8>5` is a bishop. `G` branches on `P-4`, `P-2` and `P-1` — knight, king, pawn — and everything else slides.

The same questions, asked of each board:

| question | letters | bytes | numbers | bytes |
| --- | --- | --- | --- | --- |
| empty square | `b[i]==_` | 7 | `!b[f]` | 5 |
| occupied square | `b[f]>_` | 6 | `b[f]` | 4 |
| a candidate attacker (in `T`) | `p>_&p<U!=t` | 10 | `p&8^t` | 5 |
| is it a knight (in `G`) | `P=='N'` | 6 | `P-4` | 3 |
| is it a pawn (letters in `M`, numbers in `U`) | `j(p)=='P'` | 9 | `P-1` | 3 |
| the promoted piece | `t?j(u):u` | 8 | `p\|t` | 3 |
| the king of the side to move | `b.indexOf('kK'[t])` | 18 | `b.indexOf(2+t)` | 14 |
| the pawn that may capture en passant | `p=='pP'[t]` | 10 | `p==1+t` | 6 |
| White's text colour | `b[D]<U&b[D]>_` | 13 | `b[D]&8` | 6 |
| bishop? heavy piece? (material) | `(P=j(p))=='B'` … `P>'N'?9:P>'M'` | 26 | `p%8>5` … `p%8==4\|\|p%8-2&&9` | 21 |
| the piece glyph | `'\xA0♝♚♞♟♛♜'['-BKNPQR'.search(j(b[D]))]` | 51 | `'\xA0♟♚♛♞♜♝'[b[D]&7]` | 32 |

`P-4` and `P-1` are zero for their piece, not true: `G` and `U` branch on them the other way round. In `U`, `P` is `b[i]-t`, the type of a piece of the side to move.

Choosing an encoding that packs colour, capability and order into one small integer is shorter than writing extra conditions around a string. It is not free everywhere, though, and the part-by-part count shows where numbers lose:

| part | letters | numbers | letters − numbers |
| --- | --- | --- | --- |
| aliases | 49 | 13 | +36 |
| state | 75 | 106 | −31 |
| `G` geometry | 276 | 250 | +26 |
| `T` attack | 61 | 52 | +9 |
| moves: `M` `L` `A` · `U` `L` `M` `A` | 520 | 529 | −9 |
| `d` draw | 284 | 266 | +18 |
| everything else | 632 | 622 | +10 |
| **file** | **1,897** | **1,838** | **+59** |

- **The starting position is longer as numbers:** both boards spell themselves, but the numeric one pays for an empty array and a `.map()` that turns its digits into numbers and spreads them over the 0x88 rows with `b[j*2-j%8]`.
- **The aliases go away on 0x88.** The letter board needs `a` (`Math.abs`) for its distances, `U` (`'a'`) for the case boundary and `j` (`toUpperCase`); the numeric board keeps only `N`. A signed difference carries its own direction, and the colour is a bit.
- **The moves are counted together,** because the real move sits in different places. Both boards try a move with a raw move, `M` and `U`; the numeric board makes the real move in a function of its own, its `M`, and only judges in `A`, while the letter board does both in `A` (Lesson 3). Taken as a whole, the letters now pay 9 bytes less: they save the extra function.
- **Everywhere else the letters pay too,** in `G`, `T` and `d`: every colour test, emptiness test and case conversion in the table above. The markup is the same in both files, byte for byte.

What 59 bytes buy on the letter side: a board you can read straight out of a debugger, and a repetition key that looks like the position it stands for.

## Lesson 6 — Standards, not tricks

Byte-golfed HTML usually runs on obsolete markup, because obsolete markup is shorter: `<center>`, `bgcolor`, `align`, `cellspacing`, quirks mode. Neither file does. Layout and colour are CSS; the script writes `style.background`, not `bgColor`.

**Both files validate with zero errors and zero warnings** in the W3C Nu Html Checker, CSS included (version 26.9.16; checked again with 24.10.17, and the October 2026 `index.html` with 26.10.12). The board, the promotion picker and all 64 cells are generated at runtime, so three more documents were checked for each file: the DOM after loading, the DOM after `1.e4 e5` and a selection, and the markup strings exactly as the script writes them, before any parser repairs them. All clean.

What validity costs, per file:

| part | bytes | why it is there |
| --- | --- | --- |
| `<!DOCTYPE html>` | 15 | standards mode, where CSS box sizes mean what they say |
| `<html lang=en>` | 14 | the document language; without it the checker warns |
| `<meta charset=utf-8>` | 20 | the encoding, visible in the source and immune to editors that strip a BOM |
| `<title>C</title>` | 16 | a title is required |
| `</table>` | 8 | the table's end tag is not optional |
| `</button>` | 9 | the one end tag in the generated markup that is not optional; without it the checker reports a button opened inside a button |

Some things that look like shortcuts are not. `</td>` and `</tr>` are left out, and `<head>` and `<body>` are never written: those end tags and elements are optional by specification. Unquoted attribute values such as `onclick=S(id)` are valid wherever the value holds no space, quote, `=`, `<`, `>` or backtick; `onclick="A('q')"` in `index.html` is quoted because its value contains quotes. Numeric ids are valid in HTML5 — `id=0` … `id=63` on the letter board, the sixty-four on-board 0x88 indices from `id=0` to `id=119` on the numeric one — and the script reaches them by named access: `this[w]`.

Piece glyphs are Unicode `U+265A`–`U+265F`, recoloured with CSS for White; the picker uses `U+2655`–`U+2658`. An empty square holds a no-break space, written as the escape `\xA0` so that it stays visible in the source. No image, no font download, no CDN request.

---

## Anatomy

Every byte of both files, by part, in the order they appear.

| part | `index.html` | `hexadecimal.html` | |
| --- | --- | --- | --- |
| markup and CSS | 277 | 277 | board, status line, picker, colours, layout |
| `<script>` tags | 17 | 17 | |
| aliases | 49 | 13 | `N` `a` `U` `j` · `N` |
| state | 75 | 106 | the FEN fields; in `index.html` the chain also starts the result code `z` |
| `G` | 276 | 250 | can this piece reach that square |
| `T` | 61 | 52 | is this square attacked |
| raw move: `M` · `U` | 141 | 138 | pieces only, on a fresh copy of the board; promotion, en passant victim, rook hop; returns the square a double step jumps over |
| `L` | 62 | 55 | is this move legal — try it, ask, take it back |
| `C` | 27 | 28 | which castling right a square forfeits |
| `M` | — | 114 | the real move: clock, castling rights, `U`, side to move, en passant square, key |
| `z`, `R` | 16 | 19 | result code, repetition count; the table is `G` (in `index.html` `z` starts with the state) |
| setup | 158 | 154 | promotion buttons and the 64 cells, generated |
| `d` | 284 | 266 | draw the board and the status line |
| `A` | 317 | 222 | apply the move, then the verdict; in `index.html` also the real move: castling rights, clock, `M`, side to move, en passant square, key |
| `S` | 95 | 83 | the click handler |
| `d()` | 3 | 3 | first draw |
| commas, semicolons, line breaks | 39 | 41 | the layout is worth its weight |
| **total** | **1,897** | **1,838** | |

Split another way: the rules — state, `z`/`R`, `G`, `T`, the raw move, `L`, `C`, `M`, `A` — take **975** and **984** bytes; the page that shows them — markup, CSS, script tags, setup, `d`, `S` and the first draw — takes **834** and **800**. The rest is aliases and separators. The rulebook and the board that displays it cost about the same.

---

## Verification

The engines were checked by running them, not by reading them. Both files were driven through a DOM implementation (jsdom) and Node, and for the 0x88 rewrite through headless Chromium; the markup was checked with a local copy of the Nu Html Checker.

- **perft**, both files: starting position 20 / 400 / 8,902 at depths 1–3; Kiwipete (CPW position 2) 48 / 2,039; CPW position 3 14 / 191 / 2,812.
- **En passant**, four cases, both files: pinned capturer (square not set), legal capturer (set), no capturer (not set), a neighbour that wraps across the board edge (not set).
- **Every result code**, reached by clicking real moves: `W#` after 1.e4 e5 2.Bc4 Nc6 3.Qh5 Nf6 4.Qxf7#, `B#` after 1.f3 e5 2.g4 Qh4#, `SM` by Qg5–g6 with Kf7 against Kh8, `IM` by capturing the last pawn, `5R` on the 16th ply of knights going out and back, `75` at `M=150`.
- **Insufficient material**, ten endings after a capture: K–K, K+N–K, K+B–K, K+B–K+B on one colour and K+B+B on one colour end as `IM`; K+B–K+B on opposite colours, K+N+N–K, K+B+N–K, K+N–K+N and K+P–K play on.
- **Random self-play**, 30 games per file (10,060 and 10,472 plies), with the board array checked for corruption after every ply.
- **Rendering** compared cell by cell — glyph, colour, background, outline, status line, picker — against the previous build of each file across random games, promotions included.
- **Markup**, as described in Lesson 6.
- **The rule optimizations of September 2026**, both files, against the previous build. The pawn's double step and castling walk through `S`, the castling rook is counted from the king's target, the start-rank and square-colour tests are shorter, the pawn branch asks one shared question for capture and push, and the en passant square is written with one list. Random games with the legal move lists and the en passant square compared after every ply (50 and 85 games), a castling stress test that crowds the back ranks and parks enemy rooks and queens on the squares the king crosses (241,920 geometry calls per file), and 811 double steps in pawn-heavy positions with pins, comparing the en passant square and the repetition key: no difference anywhere. perft on `hexadecimal.html` passes on the CPW positions to depth 3. The markup, `d` and the result lines did not change.
- **The rule optimizations of October 2026**, both files, against the previous build. The castling rook's square is written `f+=k>0||-2`, the start-rank test `y%6<2`, and the material test writes the square colour as `i*9/8%2+1` — in `hexadecimal.html` the weight as `p>9?p>11:9` too. `index.html` went from 1,978 to 1,975 bytes and `hexadecimal.html` from 1,892 to 1,888. The move generator for every piece, target and threat mode, the legal moves, the material test and the full state after every ply were compared with the previous build, over random self-play for `index.html` and 250 positions for `hexadecimal.html`: no difference. Sanity perft passes.
- **The 0x88 rewrite of October 2026.** `hexadecimal.html` moved from its own 8×8 board to FideLite's 0x88 engine: `G`, `T`, `U`, `L`, `C` and `M` are, line breaks aside, the corresponding parts of FideLite's `engine.js`. The page around them looks and behaves as before: the same markup apart from the table's id, now `u` because `T` names the attack test, and the same picker, status line and result codes. `index.html` took the same names wherever the two do the same job — `T` for the attack test, `R` for the repetition count, `k` for the key, `G` as the repetition table, `u` for the table — and went from 1,975 to 1,972 bytes; `hexadecimal.html` went from 1,888 to 1,838. perft on the new numeric file: 197,281 from the starting position at depth 4, 97,862 on Kiwipete at depth 3, and 43,238, 9,467 and 62,379 on CPW positions 3, 4 and 5 at depths 4, 3 and 3. Each file was then played in headless Chromium against its previous build, the same display square clicked in both after every ply, comparing every cell's glyph and style, the status line, the picker, the full game state and the legal move lists: 1,358 comparisons for the numeric file, with every result code reached and promotions through the picker, and 270 for the letter file. No difference. Both files and the three generated documents of Lesson 6 validate clean.
- **The 0x88 tactics of October 2026.** `index.html` took over the numeric file's way of moving. Its raw move `M` only moves pieces, on a fresh copy, so `L` puts back the board alone. The real move — castling rights, clock, side to move, en passant square, key — moved into `A`. The en passant question is asked once per real move, after the side switch, of every pawn of the side to move. The raw move hops the castling rook from the square `G` writes to `r`, `T` reads the side from `t`, and the starting position is built with `padEnd`, which also names the empty square and sets the en passant field. Markup, CSS, picker, status line and drawing did not change. `index.html` went from 1,972 to 1,897 bytes; `hexadecimal.html` is unchanged. The two files were driven by the same clicks, the same display square in both, with the full state compared after every click — board, side to move, castling rights, en passant square, clock, repetition count, result and status line: 300 games, 105,888 clicks with mistaken ones in between, 59 castlings, 44 en passant captures and 181 promotions through the picker. Then targeted castling, promotion and en passant cases, the pinned capturer included. In headless Chromium the two files were played side by side: 488 comparisons of every cell's glyph and style, the status line and the picker, and 274 screenshots compared pixel for pixel. perft on the letter file, against FideLite's `engine.js`: CPW positions 1 to 6 and the mirrored 4 to depth 3, and two en passant positions. No difference anywhere. Both files and the three generated documents of Lesson 6 validate clean with Nu 26.10.12.

---

## Related

- [chess1023byte](https://github.com/cuneytinann/chess1023byte) — the other end of the scale: the core rules alone, packed, in 1,023 bytes
- [fidelite.art](https://www.fidelite.art/) — the full arbiter: clock, draw offers and claims, resignation, flag fall, dead positions, fifteen result codes, eight front ends

## License

MIT

---
---

<a id="turkce"></a>

# ChessMiniature (Türkçe)

Tek bir HTML dosyasında **1.897 bayt** içinde yazılmış iki kişilik bir satranç hakemi ve onun **1.838 baytlık** ikizi. Kütüphane yok, derleme adımı yok, sunucu yok, paketleyici yok. Bir dosyayı indirin, çift tıklayın, oynayın.

İki dosya da aynı kuralları uygular. `index.html` tahtayı bir FEN gibi altmış dört karede harflerle tutar; `hexadecimal.html` sayılarla, FideLite motorunun kullandığı 0x88 tahtasında. Yan yana okunduklarında, satranç kurallarının ne kadar az karakterle yazılabileceğine ve bir veri gösterimi seçiminin neye mal olduğuna dair küçük bir derstirler.

[Golfstack](https://www.fidelite.art/) projesinin bir parçasıdır.

## Oyna

| dosya | tahta | GitHub Pages | proje sitesi |
| --- | --- | --- | --- |
| `index.html` | harfler | [ChessMiniature](https://cuneytinann.github.io/ChessMiniature/) | [L2_string_base.html](https://www.fidelite.art/builds/special/L2_string_base.html) |
| `hexadecimal.html` | sayılar | [hexadecimal.html](https://cuneytinann.github.io/ChessMiniature/hexadecimal.html) | [L2_base.html](https://www.fidelite.art/builds/special/L2_base.html) |

Proje sitesinde iki sürüm de `builds/special/` altında, `L2` kural seviyesinin varyantları olarak durur.

Bir taşa, sonra hedef kareye tıklayın. Yasal hedefler kehribar rengine döner; seçili kare, ardından da son hamlenin hedef karesi yeşil çerçeveyle işaretlenir; tahta her yarım hamleden sonra sırası gelen tarafa döner.

## Tarayıcıda okuyun

Buradaki hiçbir şey bir araçla paketlenmemiş ya da küçültülmemiştir. **Sağ tık → Sayfa Kaynağını Görüntüle** (`Ctrl` `U`, macOS'ta `⌥` `⌘` `U`) programın tamamını gösterir ve bu deponun amaçlanan kullanımı da budur: sayfayı açın, birkaç hamle oynayın, sonra o hamleleri az önce yöneten kaynağı okuyun.

Betik, bağımlılık sırasına dizilmiş, virgülle ayrılmış tek bir bildirim zinciridir; her satırda bir fikir olacak şekilde bölünmüştür. İki dosya da 56 satırdır ve parça parça aynı sırayla ilerler: sayfa, durum ve `G`'nin başı ikisinde de aynı yüksekliktedir; hüküm, tıklama işleyicisi ve ilk çizim de öyle. Aradaki satırlar motorların ayrıldığı yerde ayrılır. Satır sonlarının bir anlamı yoktur: iki betikten de silindiğinde sözdizimi ağacı birebir aynı kalır.

```
index.html         takma adlar → durum → G → T → M → L → C → kurulum → d → A → S
hexadecimal.html   takma adlar → durum → G → T → U → L → C → M → kurulum → d → A → S
```

Soldan sağa: geometri, saldırı, ham hamle, yasallık, rok hakları, ardından gerçek hamle, kurulum, çizim, uygula ve hükmet, tıklama. Ham hamle `index.html`'de `M`, `hexadecimal.html`'de `U`'dur. Gerçek hamleyi ayrı bir fonksiyonda, kendi `M`'sinde tutan yalnızca `hexadecimal.html`'dir; `index.html` onu `A`'nın içinde yapar (Ders 3).

---

## İçinde neler var

- **Tüm taş hareketleri**, aritmetikten türetilmiş. Yön tablosu yok, ofset dizisi yok.
- **Tam yasallık kontrolü.** Kendi şahınızı şah altında bırakan bir hamle asla kabul edilmez.
- **Rok**, iki yöne de, tüm koşullarıyla: rok hakkı duruyor, kale yolu açık, şah şah altında değil, şah saldırı altındaki bir kareden geçmiyor, şah şah altına girmiyor.
- **Geçerken alma**, neredeyse herkesin yanlış yaptığı durum dahil (Ders 3).
- **Seçicili terfi.** Vezir, kale, fil, at. Seçim yapılana kadar tahta kilitlenir.
- **Şah, şah mat, pat** birbirinden ayırt edilir.
- **Yetersiz materyal**, FIDE'nin okumasıyla (Ders 4).
- **5 kez tekrar** ve **75 hamle kuralı**, talep gerekmeden ilan edilir.
- **Durum satırı**: köşe koordinatları, sırası gelen taraf şah altındaysa `C!`, yarım hamle sayacı `M=`, geçerli konumun kaç kez oluştuğu `R=`. Oyun bittiğinde satır sonuç koduna dönüşür.

## İçinde neler yok

- Saat yok.
- Beraberlik teklifi ve beraberlik talebi yok; dolayısıyla 3 kez tekrar talebi ve 50 hamle talebi de yok. Sayaçlar çalışır ve gösterilir; oyunu yalnızca otomatik eşikler bitirir.
- **Kilitli pozisyon** algılaması yok. Bu, FIDE 5.2.2 maddesinin diğer yarısıdır: materyal yeterlidir ama piyonlar kilitlenmiştir ve mat zaten imkânsızdır. FideLite'ın `L2` sürümleri bu konumları piyon yapısının bitboard analiziyle bulur; burada o dedektör bilerek çıkarılmıştır. Bu yüzden sonuç kodu `DP` (ölü pozisyon) değil `IM`'dir (yetersiz materyal): kod, gerçekte neyin test edildiğini söyler. Kilitli bir pozisyon yine beraberlikle biter, yalnızca daha geç: tekrarla ya da 75 hamle kuralıyla.
- Bot yok, geri alma yok, FEN içe ya da dışa aktarma yok, PGN yok.

Eksiksiz hakem için — saat, beraberlik teklifleri ve talepleri, terk, süre bitimi, ölü pozisyonlar, on beş sonuç kodu — [fidelite.art](https://www.fidelite.art/) adresine bakın.

## Sonuç kodları

| kod | anlamı |
| --- | --- |
| `W#` | Beyaz mat eder |
| `B#` | Siyah mat eder |
| `SM` | Pat |
| `IM` | Yetersiz materyal |
| `5R` | 5 kez tekrar |
| `75` | 75 hamle kuralı |

---

## Ders 1 — Durum bir FEN kaydıdır

```js
b=[...'rnbqkbnrpppppppp'.padEnd(48,e=_='-')+'PPPPPPPPRNBQKBNR'],t=1,c=15,n=z=0,   // index.html
b=[],[...'dcebaecd99999999'+'0'.repeat(32)+'1111111154632645'].map((p,j)=>b[j*2-j%8]='0x'+p-0),t=8,c=15,e=8,n=0,   // hexadecimal.html
```

İki satır da alan alan bir FEN'dir: **taş dizilimi, sırası gelen taraf, rok hakları, geçerken alma karesi, yarım hamle sayacı.** Dışarıda kalan tek alan tam hamle numarasıdır, çünkü onu hiçbir kural okumaz: 50 ve 75 hamle kuralları yarım hamle sayar, tekrar ise hamle numarasına göre değil konuma göre belirlenir.

- **Harfler.** 0 numaralı eleman a8'dir; FEN de bu sırayla yazılır. Büyük harf Beyaz, küçük harf Siyah, `-` boş karedir. Sıra Beyaz'daysa `t=1`'dir; geçerken alma karesi yoksa `-`. Bu harf tahtanın içinde doğar: `padEnd(48,e=_='-')` otuz iki boş kareyi doldurur, harfe `_` adını verir ve geçerken alma alanını ona eşitler, hepsi tek bir argümanda; bu yüzden zincirde `e` dördüncü değil ilk sıradadır. `n=z=0`, sayaçla birlikte sonuç kodu `z`'yi de başlatır.
- **Sayılar.** Tahta 0x88'dir: bir kare `16 × yatay + dikey`'dir, 0 numaralı eleman a1'dir ve her yatay on altı yuva genişliğindedir; bunların yalnızca soldaki sekizi karedir. Altmış dört rakam harflerdeki üç parçayla gelir — bir taraf için on altı, boş orta için otuz iki (burada `'0'.repeat(32)`, harf tahtasında `padEnd(48,…)`), öteki taraf için on altı — ve `j*2-j%8` onları bu düzene yayar; her satırın sağ yarısı hiç yazılmaz. Her rakam bir taş kodudur: `0` boş, alt üç bit tür, Beyaz için artı 8 (Ders 5). Sırası gelen taraf `t` bu renk bitinin kendisidir, 8 ya da 0; geçerken alma karesinin `8` olması ise kare yok demektir: 8 numaralı kare tahtanın dışındadır.

İki satırı yan yana okuyun: dikeyler hizalanır, yataylar ters yönde akar; harfler FEN gibi a8'den, sayılar 0x88 dizini gibi a1'den başlar. Sayısal satır FEN'in alan sırasını birebir korur.

FideLite'ın kendi sürümleri beyaz piyonları ve boş ortayı `10n**40n-10n**32n` diye yazar; `99999999` ve ardından otuz iki sıfır veren bir BigInt — altı bayt daha kısa, ve okuyan için tam bir duraklama. Burada bilerek açık yazıldı. Bu dosyalar okunmak için yazıldı, ve görülebilmesi için önce hesaplanması gereken bir tahta, onu göstermenin amacını ortadan kaldırır.

Tekrar anahtarı iki dosyada da aynı dört değişkenin birleştirilmesidir:

```js
R=G[k=b+t+e+c]=-~G[k]     // harfler: tahta, sırası gelen taraf, geçerken alma karesi, rok hakları
k=b+e+t+c                 // sayılar: M yazar; A onu R=G[M(i,f,y)]=-~G[k] ile saklar
```

`-~undefined` değeri `1`'dir; bu yüzden ilk kez görülen bir konum için varsayılan değer yazmak gerekmez. Tablo `G`'nin kendisidir: fonksiyon da bir nesnedir, böylece hamle üreticisi aynı zamanda tekrar tablosu olur ve ona ayrı bir değişken harcanmaz. Alanların sırası süs değildir. Tahtadan sonra alanlar ayraçsız birbirine eklenir ve her iki sıra da bir anahtar yalnızca tek bir biçimde geri okunabilsin diye seçilmiştir; sayısal anahtarda `e` ile `c`'yi `t` ayırır. Harf tahtasında anahtar yazdırıldığında neredeyse bir FEN gibi okunur.

## Ders 2 — Geometri aritmetiktir

`G(i,f,m)` tek bir soruyu yanıtlar: `i` karesindeki taş, tahtadan başka hiçbir şeye bakmadan `f` karesine ulaşabilir mi? `m` verildiğinde hamle yerine saldırı için yanıtlar; `T` onu böyle kullanır. İki tahta yanıtı farklı sayılardan türetir.

Harf tahtası iki sayıdan başlar: dikey sıra farkı `h` ve yatay sıra farkı `v`.

```js
h=a(i%8-f%8),v=a(y-(f>>3))     dikey ve yatay sıra farkı
d=h|v                          doğrusal bir hamlenin kat ettiği kare sayısı: h==0, v==0 ya da h==v iken h|v, max(h,v)'dir
s=(f-i)/d                      adım vektörü — ±1, ±8, ±7, ±9 — tek bir bölmeyle
h*v==2                         at: 2 sonucunu yalnızca 1×2 ve 2×1 verir
m|v|h^2?d<2:…                  şah: bir kare ya da rok olarak yatayda iki kare
r=f+=s>0||-2                   rok kalesinin karesi, şahın hedefinden sayılarak: g1'den bir ileri (true 1 sayılır), c1'den iki geri; r onu M için saklar
h*v==1                         piyon alışı: bir dikey, bir yatay
y%6<2                          piyonların başlangıç yatayları: y%6 yalnız 0., 1., 6. ve 7. yatayda 2'den küçük, piyon da 0. ve 7. yatayda hiç durmaz
f%56<8                         son yatay, iki renk için de
f^8                            geçerken alınan piyon: 8 ile XOR, doğru yönde bir yatay geri gider
i*9/8%2                        bir karenin rengi: i*9/8 indeks artı yatay, tekliği de karenin rengi
```

0x88 tahtası tek bir sayıyla yetinir: fark, `d=f-i`. Bir yataya on altı yuva düşünce her yön sabit bir farktır — yatay boyunca ±1, dikey boyunca ±16, çaprazlarda ±15 ve ±17 — ve adım bir bölünebilirlik sorusuna dönüşür:

```js
d=f-i                          fark; 0x88 tahtasında yönü tek başına söyler
x=d%15?d%16?d%17?1:17:16:15    adım: çaprazda 15 ya da 17, dikeyde 16, yatayda ya da hiçbir hatta değilse 1
s=d<0?-x:x                     işaretiyle birlikte adım
d*d%765%128==68                at: ±14, ±18, ±31 ve ±33'ün karesi iki kez katlanınca 68'e düşer, başka hiçbir fark düşmez
~m+3%d?d==s:…                  şah: bir adım ya da rok olarak yatayda iki kare; test yalnızca saldırı sorusu dışında d=±2 iken sıfırdır
r=f+=s-!~s                     rok kalesinin karesi, şahın hedefinden sayılarak: g1'den bir ileri, c1'den iki geri; r onu U için saklar
m^d<d*t                        piyon ileri gider: d<d*t, Beyaz için (t=8) d>0, Siyah için (t=0) d<0 demektir; saldırı sorusunda piyon öbür tarafındır ve m^ yönü çevirir
1&~x%d                         piyon alışı: yalnızca d=±15 ve ±17 için tektir
i%96<24                        piyonların başlangıç yatayları: yalnızca 0., 1., 6. ve 7. yatayda doğru
f%112>7                        son yatay değil, iki renk için de; U'da terfiyi ayırt eder
f^16                           geçerken alınan piyon: 16 ile XOR bir yatay geri gider
i%15%2                         bir karenin rengi: 16·yatay+dikey'in 15'e bölümünden kalan yatay+dikey'dir, tekliği de karenin rengi
w&8                            tahtanın dışı: her satırın hiç yazılmayan sağ yarısı; d çizerken onu atlar
```

Uzun menzilli taşlar hattı `S` ile adım adım yürür; hedefe ya da boş olmayan ilk kareye gelince durur. Piyonun çift adımı ve rok da aynı yürüyüşü kullanır: piyon için dikey boyunca iki adım, rok için şahtan kaleye giden yol. Hangi taşın hangi hatta yürüyebileceği, iki dosyanın ilk ayrıştığı yerdir ve ikisini de okumaya değer:

```js
(h*v?h==v&P<'R':P>'B')&S()     harfler: alfabede B < Q < R; P<'R' fil ve veziri çaprazda,
                               P>'B' vezir ve kaleyi düz hatta geçirir
P+P*4%x&S()                    sayılar: vezir 3, kale 5, fil 6; P+P*4%x tam olarak tür x adımıyla
                               gidebiliyorsa tektir, & da o son biti tutar
```

Sayısal testi elle sınamaya değer. Yatayda (`x=1`) 3, 5 ve 6 verir; dikeyde (`x=16`) 15, 9 ve 14; çaprazda (`x=15` ya da `17`) 15 ya da 15, 10 ya da 8, 15 ya da 13. Vezir her yerde tektir, kale yalnızca düz hatlarda, fil yalnızca çaprazlarda.

Yürüyüşü neyin durdurduğunda ikinci bir ders saklıdır. Harf tahtasında kenarın ötesinden okunan değer `undefined`'dır ve `'-'` değildir; bu yüzden hat olmayan bir yürüyüş kendiliğinden durur ve `&` güvenlidir. 0x88 tahtasında aynı şey tasarım gereği doğrudur. Yürüyüş `b[i]<1` diye sorar; her satırın sağ yarısı hiç yazılmaz, tahtadan herhangi bir yöne taşan bir adım o boşluğa ya da dizinin dışına düşer ve `undefined<1` yanlıştır. Kenar, yürüyüşü bir taş gibi durdurur; `x`, hiçbir hatta olmayan bir fark için `1`'e düşmüş olsa bile. Eski 8×8 sayısal dosya boşluğu `!b[i]` diye sınıyordu ve `!undefined` değeri `true`'dur; bu yüzden her yürüyüşü `&&` ile korumak zorundaydı.

## Ders 3 — Yasallık: oyna, sor, geri al

Yasallık ikinci bir hamle üreticisi değildir, tek bir numaradır:

- `T`, bir karenin saldırı altında olup olmadığını bütün tahtaya sorar; varsayılan olarak sırası gelen tarafın şahını. İki tahta da tarafı `t`'den okur; hiçbirinin onu ayrıca belirtmesi gerekmez.
- `L(i,x)`, aday hamleyi tahtanın bir kopyasında dener, `T`'ye sorar ve tahtayı geri koyar.
- Hamleyi **ham hamle** ile dener: yalnızca taşları, yeni bir kopya üzerinde oynatan fonksiyon — harf tahtasında `[...b]`, sayısal tahtada `b.slice()` — bu yüzden `L` tahtayı `b=q` ile geri koyar, başka hiçbir şeyi değil. Ham hamle harf tahtasında `M`, sayısal tahtada `U`'dur. Yalnızca gerçek bir hamlenin yaptıkları — rok hakları, sayaç, sıra, geçerken alma karesi, anahtar — gerçek hamle başına bir kez yazılır: harf tahtasında `A`'nın içinde, sayısal tahtada onun `M`'sinde; o `M`'yi de yalnızca `A` çağırır. Her iki durumda da kalıcı bir şey yapan tek fonksiyon `A`'dır.

Şah rok yaptığında kaleyi de ham hamle götürür: kalenin karesinden şahın geçtiği kareye ikinci bir ham hamle — harf tahtasında `j(p)+d=='K2'&&M(r,Q)`, iki kare giden şah tek bir dizge olarak okunur; sayısal tahtada `d-8||U(r,Q)`. Kaleyi `G`'nin bıraktığı yerde bulur: bir rok hamlesini sınarken `G`, kalenin karesini `r`'ye yazar (Ders 2). Bu yüzden aynı hamle için `L`'nin ham hamleden hemen önce çalışması gerekir — ve her zaman öyle olur, çünkü tıklama işleyicisi önce `L`'ye sorar, `A`'yı ancak ondan sonra çağırır.

Bağımlılık tam iki yerde geriye doğru akar ve ikisi de koddan değil kurallardan gelir. `G`, `T`'yi çağırır; çünkü rok, şahın şah altında olup olmadığı ya da saldırı altındaki bir kareden geçip geçmediği sorulmadan değerlendirilemez; kaleye giden yol iki saldırı testinden sonra, en son yürünür. Gerçek hamle `L`'yi çağırır; çünkü geçerken alma karesi ancak alış gerçekten yasalsa yazılabilir:

```js
e=M(i,f,y),t^=1,
b.some((p,x)=>p=='pP'[t]&&L(x,e))||(e=_)     // harfler, A içinde
e=U(i,x,p),t^=8,
e=b.some((p,j)=>p==1+t&&L(j,e))?e:8          // sayılar, M içinde
```

Bir piyon az önce iki kare ilerledi. Saf uygulama geçerken alma karesini hemen yazar. Bu uygulama kareyi geçici olarak yazar, bir rakip piyonun oraya *yasal olarak* alış yapıp yapamayacağını sorar, yapamıyorsa geri alır. İki tahta da bunu aynı biçimde yapar. Ham hamle piyonun üzerinden atladığı kareyi, başka her hamleden sonra da "kare yok" değerini döndürür — harf tahtasında `_`, sayısal tahtada `8` — ve bu önce `e`'ye yazılır. Sonra, sıra zaten değişmişken, artık sırası gelen tarafın her piyonuna sorulur: harf tahtasında `'pP'[t]`, sayısal tahtada `1+t` kodu. Hedefin yanında olmayan her piyonu `G` reddeder; kare yoksa da her soru hemen biter: `L`, `b['-']`'de de tahtanın dışındaki 8 numaralı karede de alınacak bir şey bulmaz.

Fark alışın kendisinde hiç görünmez: yasadışı bir alışı hamle üreticisi zaten reddeder. Fark **tekrar sayacında** görünür, çünkü `e` konum anahtarının parçasıdır. FIDE iki konumu ancak ikisinde de aynı hamleler — geçerken alma dahil — mümkünse aynı sayar. Kimsenin kullanamayacağı bir kare yazarsanız aynı konum iki farklı anahtar altına düşer: tekrar beraberliği geç gelir ya da hiç gelmez.

Klasik örnek: Siyah g7–g5 oynar; f5'teki beyaz piyon, f8'deki kale tarafından f1'deki şahına açmazlanmıştır. `fxg6` şahı açığa çıkarır, alış yasadışıdır, kare hiç yazılmaz. Bu kontrol harf tahtasında **41 bayt**, sayısal tahtada **36 bayt** tutar; saf biçimler yalnızca `e=M(i,f,y)` ve `e=U(i,x,p)`'dir.

Soru gerçek hamle başına bir kez sorulur, hiçbir denemenin içinde sorulmaz; çünkü deneme ham hamledir ve ham hamle `e`'ye hiç dokunmaz. Harf tahtası bu soruyu eskiden kendi `M`'sinin içinde, `M`'nin yazdığı her hamleden sonra — deneme hamleleri dahil — soruyordu. Bunun iki bedeli vardı: `L` her denemenin etrafında geçerken alma karesini ve sayacı saklayıp geri yüklemek zorundaydı, perft de 4. ve 5. derinlikte belirgin biçimde yavaşlıyordu. Sorunun `A`'ya taşınmasıyla ikisi de ortadan kalktı (Ekim 2026, bkz. Doğrulama).

## Ders 4 — Herkesin atladığı kurallar ucuzdur

Kod golfü koleksiyonlarındaki satranç programları yarım hamle sayacını, tekrar sayacını ve yetersiz materyal testini pahalı defter tutma diye sıklıkla çıkarır. Bu dosyalarda ölçülen değerler:

| kural | harfler | bayt | sayılar | bayt |
| --- | --- | --- | --- | --- |
| Yarım hamle sayacı | `n=P\|b[f]>_?0:n+1` | 16 | `n=!b[x]*b[i]&6&&n+1` | 19 |
| Tekrar sayacı | `R=G[k=b+t+e+c]=-~G[k]` | 21 | `k=b+e+t+c` · `R=G[…]=-~G[k]` | 21 |
| Dört rok hakkının hepsi | `C=i=>'20003001'[i%56]<<i/28` | 27 | `C=i=>'20003001'[i%112]<<i/56` | 28 |
| Yetersiz materyal, iki taraf birden | `(m=W=0,b.map((p,i)=>(P=j(p))=='B'?m\|=i*9/8%2+1:W+=P>'N'?9:P>'M'),W*2+m<3)` | 73 | `(m=W=0,b.map((p,i)=>p&&(p%8>5?m\|=i%15%2+1:W+=p%8==4\|\|p%8-2&&9)),W*2+m<3)` | 72 |
| Oyunun bütün bitişleri | `?'IM':R>4?'5R':n>149&&'75':T()?t?'B#':'W#':'SM'` | 47 | `?'IM':R>4?'5R':n>149&&'75':T()?t?'B#':'W#':'SM'` | 47 |
| **toplam** | | **184** | | **187** |

İki dosyanın da aşağı yukarı onda biri. Bir satranç programının pahalı kısmı hiçbir zaman kural kitabı olmadı.

**Yarım hamle sayacı** sayısal tahtada tek bir çarpımdır: `!b[x]` yalnızca hedef kare boşsa 1'dir, `b[i]&6` de yalnızca piyon için 0'dır; sayaç tam olarak hiçbir piyon oynamadığında ve hiçbir şey alınmadığında ilerler. Harf tahtasında sayaç `A`'da, `P`'den yazılır: tıklama işleyicisi oynayan taşa terfi için bakarken `P`'yi kurar, böylece `A` bir piyonun oynayıp oynamadığını yeniden sormadan bilir.

**Rok hakları** tek bir dizgedir: ilk ya da son yataydaki bir karenin altındaki rakam, o kareden bir taş ayrıldığında ya da oraya bir taş geldiğinde kaybedilen haktır; uzak yatayda öbür rengin bitlerine kaydırılır — harf tahtasında `i/28`, 0x88'de `i/56` kadar: dizinin bir ucunda 0, öbür ucunda 2. Şah ve kale kareleri kendi bitlerini taşır; diğer bütün kareler `0` verir. Her hamle iki karesini de temizler: harf tahtasında `A` içinde `c&=~C(i)&~C(f)`, sayısal tahtada `M` içinde `c&=~C(i)&~C(x)`.

**Yetersiz materyal** tek geçiş ve tek karşılaştırmadır. `m`, fillerin durduğu kare renklerini bit olarak toplar (1 ya da 2; iki renk birden 3 eder); harf tahtasında `i*9/8%2`, 0x88'de `i%15%2` diye yazılır. `W` bir atı 1, herhangi bir piyon, kale ya da veziri 9 sayar; şahlar bir şey eklemez. Harf tahtasında `P>'N'?9:P>'M'` tam olarak bunu söyler: `P`, `Q` ve `R` alfabede `N`'den, `N` de `M`'den sonra gelir; şah ve boş kare `-` ikisinden de önce. Bu yüzden boş kareler için ayrı bir sınama gerekmez. Böylece `W*2+m<3` tam olarak şu durumlarda doğrudur: yalnız şahlar, tek bir at ya da hepsi aynı renk karede duran filler — hangi tarafta olursa olsun. İki at, fil ve at, zıt renkli filler ya da her iki tarafta birer at 3 ya da daha fazla verir ve oyun sürer.

**Talepler ve ilanlar.** *Talep* edilmesi gereken bir beraberlik, onu talep edecek bir oyuncuya, o da bir düğmeye ihtiyaç duyar. Bu sürümde böyle bir katman yok; bu yüzden yalnızca bir hakemin kendiliğinden ilan ettiği bitişler tutulur: 5 kez tekrar, 75 hamle kuralı, yetersiz materyal, mat ve pat. `R=`'nin 3'e çıktığını, `M=`'nin 99'u geçtiğini yine de izleyebilirsiniz — FIDE'ye göre bu iki eşik ilan değil taleptir, bu yüzden burada oyunu bitirmezler.

## Ders 5 — Harfler mi, sayılar mı

Bir harf tek bir şey söyler: hangi taş. Renk harfin büyüklüğünden, boşluk ayrı bir karakterden çıkarılmak zorundadır; bu yüzden neredeyse her soru fazladan bir karşılaştırma ya da büyük harfe çevirme ister (`j`, `toUpperCase`'tir). Bir sayı ise soruları bitleri zaten yanıtlayacak şekilde seçilebilir:

| kod | 0 | 1 · 9 | 2 · 10 | 3 · 11 | 4 · 12 | 5 · 13 | 6 · 14 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| taş | boş | piyon | şah | vezir | at | kale | fil |

Alt üç bit türdür, `p&7`; 8 de Beyaz'dır: her çiftin ilk kodu Siyah, ikincisi Beyaz. Bunlar FideLite'ın 0x88 motorunun kodlarıdır ve sıra alfabetik değil, işlevseldir. Vezir 3, kale 5 ve fil 6, Ders 2'deki uzun menzil testini çalıştırır. Sırası gelen taraf `t` renk bitinin kendisidir, 8 ya da 0; böylece sırası gelen tarafın piyonu `1+t`, şahı `2+t`, terfi eden taşı `p|t` olur, `p&8^t` de `T` içinde öbür tarafı seçer. `p&6` yalnızca piyon için sıfırdır, yarım hamle sayacının ihtiyacı da bundan ibarettir; `p%8>5` ise fildir. `G`, `P-4`, `P-2` ve `P-1` üzerinde dallanır — at, şah, piyon — geri kalan her şey kayar.

Aynı sorular, iki tahtaya ayrı ayrı:

| soru | harfler | bayt | sayılar | bayt |
| --- | --- | --- | --- | --- |
| boş kare | `b[i]==_` | 7 | `!b[f]` | 5 |
| dolu kare | `b[f]>_` | 6 | `b[f]` | 4 |
| saldırabilecek aday (`T` içinde) | `p>_&p<U!=t` | 10 | `p&8^t` | 5 |
| at mı (`G` içinde) | `P=='N'` | 6 | `P-4` | 3 |
| piyon mu (harflerde `M`, sayılarda `U` içinde) | `j(p)=='P'` | 9 | `P-1` | 3 |
| terfi eden taş | `t?j(u):u` | 8 | `p\|t` | 3 |
| sırası gelen tarafın şahı | `b.indexOf('kK'[t])` | 18 | `b.indexOf(2+t)` | 14 |
| geçerken alabilecek piyon | `p=='pP'[t]` | 10 | `p==1+t` | 6 |
| Beyaz'ın yazı rengi | `b[D]<U&b[D]>_` | 13 | `b[D]&8` | 6 |
| fil mi? ağır taş mı? (materyal) | `(P=j(p))=='B'` … `P>'N'?9:P>'M'` | 26 | `p%8>5` … `p%8==4\|\|p%8-2&&9` | 21 |
| taş karakteri | `'\xA0♝♚♞♟♛♜'['-BKNPQR'.search(j(b[D]))]` | 51 | `'\xA0♟♚♛♞♜♝'[b[D]&7]` | 32 |

`P-4` ve `P-1` kendi taşları için doğru değil sıfırdır: `G` ve `U` onlara göre ters yönden dallanır. `U` içinde `P`, `b[i]-t`'dir, yani sırası gelen tarafın bir taşının türü.

Rengi, yeteneği ve sırayı tek bir küçük tam sayıya yükleyen bir kodlama seçmek, bir dizgenin etrafına fazladan koşullar yazmaktan daha kısadır. Ama her yerde bedava değildir; parça parça sayım sayıların nerede kaybettiğini gösterir:

| parça | harfler | sayılar | harfler − sayılar |
| --- | --- | --- | --- |
| takma adlar | 49 | 13 | +36 |
| durum | 75 | 106 | −31 |
| `G` geometri | 276 | 250 | +26 |
| `T` saldırı | 61 | 52 | +9 |
| hamleler: `M` `L` `A` · `U` `L` `M` `A` | 520 | 529 | −9 |
| `d` çizim | 284 | 266 | +18 |
| geri kalan her şey | 632 | 622 | +10 |
| **dosya** | **1.897** | **1.838** | **+59** |

- **Başlangıç konumu sayılarla daha uzundur:** iki tahta da kendini heceler, ama sayısal olan boş bir diziye ve rakamlarını sayıya çevirip `b[j*2-j%8]` ile 0x88 satırlarına yayan bir `.map()`'e öder.
- **0x88'de takma adlar gider.** Harf tahtası uzaklıkları için `a`'ya (`Math.abs`), büyük-küçük harf sınırı için `U`'ya (`'a'`) ve `j`'ye (`toUpperCase`) ihtiyaç duyar; sayısal tahtada yalnızca `N` kalır. İşaretli bir fark yönünü kendisi taşır, renk de bir bittir.
- **Hamleler birlikte sayılır,** çünkü gerçek hamle farklı yerlerde durur. İki tahta da bir hamleyi ham hamleyle, `M` ve `U` ile dener; sayısal tahta gerçek hamleyi ayrı bir fonksiyonda, kendi `M`'sinde yapar ve `A`'da yalnızca hükmeder, harf tahtası ikisini de `A`'da yapar (Ders 3). Bütün olarak harfler artık 9 bayt az öder: fazladan fonksiyonun bedelini ödemezler.
- **Geri kalan yerlerde de harfler öder,** `G`, `T` ve `d` içinde: yukarıdaki tablodaki her renk testi, boşluk testi ve büyük harfe çevirme. İşaretleme iki dosyada da bayt bayt aynıdır.

Harf tarafında 59 baytın karşılığı: bir hata ayıklayıcıdan doğrudan okunabilen bir tahta ve temsil ettiği konuma benzeyen bir tekrar anahtarı.

## Ders 6 — Hile değil, standart

Bayt golfü yapılmış HTML genellikle eskimiş işaretlemeyle çalışır, çünkü eskimiş işaretleme daha kısadır: `<center>`, `bgcolor`, `align`, `cellspacing`, quirks kipi. İki dosya da bunların hiçbirini kullanmaz. Yerleşim ve renk CSS'tedir; betik `bgColor` değil `style.background` yazar.

**İki dosya da W3C Nu Html Checker'da, CSS dahil, sıfır hata ve sıfır uyarıyla doğrulanır** (sürüm 26.9.16; 24.10.17 ile yeniden denetlendi, `index.html`'in Ekim 2026 hâli de 26.10.12 ile). Tahta, terfi seçicisi ve 64 hücrenin hepsi çalışma anında üretildiği için her dosya için üç belge daha denetlendi: yüklemeden sonraki DOM, `1.e4 e5` ve bir seçimden sonraki DOM, ve betiğin yazdığı işaretleme dizgelerinin, ayrıştırıcı onları onarmadan önceki birebir hâli. Hepsi temiz.

Geçerliliğin dosya başına maliyeti:

| parça | bayt | neden orada |
| --- | --- | --- |
| `<!DOCTYPE html>` | 15 | standart kipi; CSS kutu boyutları söylediği anlama gelir |
| `<html lang=en>` | 14 | belgenin dili; yazılmazsa denetleyici uyarır |
| `<meta charset=utf-8>` | 20 | kodlama; kaynakta görünür ve BOM'u silen editörlerden etkilenmez |
| `<title>C</title>` | 16 | başlık zorunludur |
| `</table>` | 8 | tablonun bitiş etiketi isteğe bağlı değildir |
| `</button>` | 9 | üretilen işaretlemedeki isteğe bağlı olmayan tek bitiş etiketi; yazılmazsa denetleyici düğme içinde açılmış bir düğme bildirir |

Kısayol gibi görünen bazı şeyler kısayol değildir. `</td>` ve `</tr>` yazılmaz, `<head>` ve `<body>` hiç yazılmaz: bu bitiş etiketleri ve öğeler spesifikasyona göre isteğe bağlıdır. `onclick=S(id)` gibi tırnaksız öznitelik değerleri, değer boşluk, tırnak, `=`, `<`, `>` ya da ters tırnak içermediği sürece geçerlidir; `index.html`'deki `onclick="A('q')"` ise değeri tırnak içerdiği için tırnaklıdır. Sayısal id'ler HTML5'te geçerlidir — harf tahtasında `id=0` … `id=63`, sayısal tahtada `id=0`'dan `id=119`'a tahta içindeki altmış dört 0x88 dizini — ve betik onlara isimli erişimle ulaşır: `this[w]`.

Taş karakterleri Unicode `U+265A`–`U+265F` aralığındadır ve Beyaz için CSS ile yeniden renklendirilir; seçici `U+2655`–`U+2658` kullanır. Boş kare bölünmez bir boşluk tutar; kaynakta görünür kalsın diye `\xA0` kaçış dizisiyle yazılmıştır. Görsel yok, yazı tipi indirme yok, CDN isteği yok.

---

## Anatomi

İki dosyanın her baytı, parça parça, dosyadaki sırasıyla.

| parça | `index.html` | `hexadecimal.html` | |
| --- | --- | --- | --- |
| işaretleme ve CSS | 277 | 277 | tahta, durum satırı, seçici, renkler, yerleşim |
| `<script>` etiketleri | 17 | 17 | |
| takma adlar | 49 | 13 | `N` `a` `U` `j` · `N` |
| durum | 75 | 106 | FEN alanları; `index.html`'de zincir sonuç kodu `z`'yi de başlatır |
| `G` | 276 | 250 | bu taş o kareye ulaşabilir mi |
| `T` | 61 | 52 | bu kare saldırı altında mı |
| ham hamle: `M` · `U` | 141 | 138 | yalnızca taşlar, tahtanın yeni bir kopyasında; terfi, geçerken alınan piyon, kale atlaması; çift adımın üzerinden atladığı kareyi döndürür |
| `L` | 62 | 55 | bu hamle yasal mı — dene, sor, geri al |
| `C` | 27 | 28 | bir karenin hangi rok hakkını kaybettirdiği |
| `M` | — | 114 | gerçek hamle: sayaç, rok hakları, `U`, sıra, geçerken alma karesi, anahtar |
| `z`, `R` | 16 | 19 | sonuç kodu, tekrar sayısı; tablo `G`'dir (`index.html`'de `z` durumla başlar) |
| kurulum | 158 | 154 | terfi düğmeleri ve 64 hücre, üretilmiş |
| `d` | 284 | 266 | tahtayı ve durum satırını çiz |
| `A` | 317 | 222 | hamleyi uygula, sonra hükmü ver; `index.html`'de gerçek hamle de burada: rok hakları, sayaç, `M`, sıra, geçerken alma karesi, anahtar |
| `S` | 95 | 83 | tıklama işleyicisi |
| `d()` | 3 | 3 | ilk çizim |
| virgüller, noktalı virgüller, satır sonları | 39 | 41 | bu yerleşim maliyetine değer |
| **toplam** | **1.897** | **1.838** | |

Başka bir açıdan bölünce: kurallar — durum, `z`/`R`, `G`, `T`, ham hamle, `L`, `C`, `M`, `A` — **975** ve **984** bayt tutar; onları gösteren sayfa — işaretleme, CSS, betik etiketleri, kurulum, `d`, `S` ve ilk çizim — **834** ve **800**. Geri kalanı takma adlar ve ayraçlardır. Kural kitabı ile onu gösteren tahta aşağı yukarı aynı tutar.

---

## Doğrulama

Motorlar okunarak değil çalıştırılarak denetlendi. İki dosya da bir DOM uygulaması (jsdom) ve Node üzerinde, 0x88'e geçişte de başsız Chromium'da sürüldü; işaretleme, Nu Html Checker'ın yerel bir kopyasıyla denetlendi.

- **perft**, iki dosyada da: başlangıç konumunda 1–3 derinlikte 20 / 400 / 8.902; Kiwipete (CPW 2. konum) 48 / 2.039; CPW 3. konum 14 / 191 / 2.812.
- **Geçerken alma**, dört durum, iki dosyada da: açmazdaki alıcı (kare yazılmaz), yasal alıcı (yazılır), alıcı yok (yazılmaz), tahtanın kenarından öbür tarafa sarılan komşu (yazılmaz).
- **Her sonuç kodu**, gerçek hamleler tıklanarak elde edildi: 1.e4 e5 2.Bc4 Nc6 3.Qh5 Nf6 4.Qxf7# ile `W#`, 1.f3 e5 2.g4 Qh4# ile `B#`, Kh8'e karşı Kf7 varken Qg5–g6 ile `SM`, son piyonu alarak `IM`, atların çıkıp geri döndüğü 16. yarım hamlede `5R`, `M=150`'de `75`.
- **Yetersiz materyal**, bir alıştan sonra kalan on oyun sonu: Ş–Ş, Ş+A–Ş, Ş+F–Ş, aynı renkte Ş+F–Ş+F ve aynı renkte Ş+F+F `IM` ile biter; zıt renkte Ş+F–Ş+F, Ş+A+A–Ş, Ş+F+A–Ş, Ş+A–Ş+A ve Ş+P–Ş sürer.
- **Rastgele kendi kendine oyun**, dosya başına 30 oyun (10.060 ve 10.472 yarım hamle); her yarım hamleden sonra tahta dizisi bozulmaya karşı denetlendi.
- **Görüntü**, her dosyanın bir önceki sürümüyle rastgele oyunlarda, terfiler dahil, hücre hücre karşılaştırıldı: taş karakteri, renk, arka plan, çerçeve, durum satırı, seçici.
- **İşaretleme**, Ders 6'da anlatıldığı gibi.
- **Eylül 2026 kural optimizasyonları**, iki dosyada da, bir önceki sürüme karşı. Piyonun çift adımı ve rok `S` ile yürüyor, rok kalesi şahın hedefinden sayılıyor, başlangıç yatayı ve kare rengi testleri kısaldı, piyon dalı alış ve itiş için tek bir ortak soru soruyor, geçerken alma karesi tek bir listeyle yazılıyor. Her yarım hamleden sonra yasal hamle listeleri ve geçerken alma karesi karşılaştırılan rastgele oyunlar (50 ve 85 oyun), arka yatayları doldurup şahın geçtiği karelere rakip kale ve vezir koyan bir rok zorlama testi (dosya başına 241.920 geometri çağrısı) ve açmazlı, piyon yoğun pozisyonlarda 811 çift adımda geçerken alma karesi ile tekrar anahtarının karşılaştırılması: hiçbir yerde fark yok. `hexadecimal.html`'de CPW pozisyonlarında 3. derinliğe kadar perft geçiyor. İşaretleme, `d` ve sonuç satırları değişmedi.
- **Ekim 2026 kural optimizasyonları**, iki dosyada da, bir önceki sürüme karşı. Rok kalesinin karesi `f+=k>0||-2`, başlangıç yatayı testi `y%6<2` diye yazılıyor; materyal testi kare rengini `i*9/8%2+1` diye yazıyor, `hexadecimal.html`'de ağırlığı da `p>9?p>11:9` diye. `index.html` 1.978 bayttan 1.975'e, `hexadecimal.html` 1.892'den 1.888'e indi. Her taş, hedef ve tehdit kipi için hamle üretici, yasal hamleler, materyal testi ve her yarım hamleden sonraki tam durum bir önceki sürümle karşılaştırıldı — `index.html` için rastgele kendi kendine oyunlarda, `hexadecimal.html` için 250 pozisyonda: fark yok. Sanity perft geçiyor.
- **Ekim 2026 0x88 geçişi.** `hexadecimal.html` kendi 8×8 tahtasından FideLite'ın 0x88 motoruna geçti: `G`, `T`, `U`, `L`, `C` ve `M`, satır sonları dışında, FideLite'ın `engine.js` dosyasındaki karşılıklarıyla aynıdır. Etraftaki sayfa eskisi gibi görünür ve çalışır: işaretleme, tablonun id'si dışında aynıdır (`T` artık saldırı testinin adı olduğu için id `u` oldu); seçici, durum satırı ve sonuç kodları da aynıdır. `index.html`, iki dosyanın aynı işi yaptığı her yerde aynı adları aldı — saldırı testi için `T`, tekrar sayısı için `R`, anahtar için `k`, tekrar tablosu olarak `G`, tablo için `u` — ve 1.975 bayttan 1.972'ye indi; `hexadecimal.html` 1.888'den 1.838'e. Yeni sayısal dosyada perft: başlangıç konumunda 4. derinlikte 197.281, Kiwipete'te 3. derinlikte 97.862, CPW 3., 4. ve 5. konumlarda 4., 3. ve 3. derinlikte 43.238, 9.467 ve 62.379. Ardından her dosya başsız Chromium'da bir önceki sürümüyle yan yana oynatıldı; her yarım hamleden sonra ikisinde de aynı ekran karesine tıklanıp her hücrenin taş karakteri ve stili, durum satırı, seçici, oyunun tam durumu ve yasal hamle listeleri karşılaştırıldı: sayısal dosyada 1.358 karşılaştırma, her sonuç koduna ulaşılarak ve seçiciyle terfiler dahil; harf dosyasında 270. Fark yok. İki dosya ve Ders 6'daki üç üretilmiş belge temiz doğrulanıyor.
- **Ekim 2026 0x88 taktikleri.** `index.html`, sayısal dosyanın hamle yapma biçimini devraldı. Ham hamlesi `M` yalnızca taşları, yeni bir kopya üzerinde oynatıyor; bu yüzden `L` yalnız tahtayı geri koyuyor. Gerçek hamle — rok hakları, sayaç, sıra, geçerken alma karesi, anahtar — `A`'ya taşındı. Geçerken alma sorusu gerçek hamle başına bir kez, sıra değiştikten sonra, sırası gelen tarafın her piyonuna soruluyor. Rok kalesini ham hamle, `G`'nin `r`'ye yazdığı kareden atlatıyor; `T` tarafı `t`'den okuyor; başlangıç konumu, boş kare harfini de adlandıran ve geçerken alma alanını kuran `padEnd` ile yazılıyor. İşaretleme, CSS, seçici, durum satırı ve çizim değişmedi. `index.html` 1.972 bayttan 1.897'ye indi; `hexadecimal.html` değişmedi. İki dosya aynı tıklamalarla, ikisinde de aynı ekran karesine tıklanarak sürüldü ve her tıklamadan sonra tam durum karşılaştırıldı — tahta, sıra, rok hakları, geçerken alma karesi, sayaç, tekrar sayısı, sonuç ve durum satırı: arada yanlış tıklamalarla 300 oyun, 105.888 tıklama, 59 rok, 44 geçerken alma ve seçiciyle 181 terfi. Ardından açmazdaki alıcı dahil hedefli rok, terfi ve geçerken alma durumları. Başsız Chromium'da iki dosya yan yana oynatıldı: her hücrenin taş karakteri ve stili, durum satırı ve seçici 488 kez karşılaştırıldı, 274 ekran görüntüsü piksel piksel karşılaştırıldı. Harf dosyasında FideLite'ın `engine.js`'ine karşı perft: CPW 1–6. konumlar ve aynalanmış 4. konum 3. derinliğe kadar, ayrıca iki geçerken alma konumu. Hiçbir yerde fark yok. İki dosya ve Ders 6'daki üç üretilmiş belge Nu 26.10.12 ile temiz doğrulanıyor.

---

## İlgili

- [chess1023byte](https://github.com/cuneytinann/chess1023byte) — ölçeğin öbür ucu: yalnızca temel kurallar, paketlenmiş, 1.023 bayt
- [fidelite.art](https://www.fidelite.art/) — eksiksiz hakem: saat, beraberlik teklifleri ve talepleri, terk, süre bitimi, ölü pozisyonlar, on beş sonuç kodu, sekiz ön yüz

## Lisans

MIT
