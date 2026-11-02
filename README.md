# Two-Player Columns in MIPS Assembly
 
A competitive two-player version of the puzzle game *Columns*, written entirely in MIPS assembly. It draws directly to a memory-mapped bitmap display and runs two boards at once, with an attack system that sends garbage rows to your opponent.
 
**▶ Demo video:** [Watch on YouTube](https://youtu.be/Or2_q631Kgs)
 
> **About the source code:** this was a course project for Computer Organization at the University of Toronto. At the course's request, the source isn't public, so that future students can't reuse it. This repository documents the design. The code can be shared privately on request.
 
---
 
## Gameplay
 
Each player controls a falling column of three coloured gems on their own board. Line up **three or more gems of the same colour** vertically, horizontally or diagonally to clear them. Gems above then fall, which can set off **chain reactions**.
 
Clearing chains attacks your opponent:
 
| Chain reaction | Garbage rows sent |
| --- | --- |
| 2-chain | 1 row |
| 3-chain | 3 rows |
| 4+ chain | 5 rows |
 
Garbage rows enter at the bottom of the opponent's board and push everything up. A player loses when a new column can't spawn at the top. The result screen then shows **W** and **L** on the two boards.
 
## Features
 
- **Two simultaneous players**, each with their own board, active piece and fall timer
- **Gravity:** columns fall automatically at a fixed interval
- **Chain reactions:** match, remove, apply gravity, and repeat until the board is stable
- **Garbage attack system:** chains send rows to the opponent (table above)
- **Ghost piece:** a greyed preview of where the column will land
- **Hard drop:** instantly places the column at its lowest valid position
- **Preview panel:** a queue of upcoming columns
- **Pause:** a pause symbol in the centre of the screen
- **Freeze:** player 1 can freeze player 2's column
- **Heart display:** a centre UI element that fills with colour as the game goes on
## Controls
 
| Action | Player 1 | Player 2 |
| --- | --- | --- |
| Move left / right | `A` / `D` | `J` / `L` |
| Shuffle gem order | `W` | `I` |
| Soft drop | `S` | `K` |
| Hard drop | `X` | `M` |
 
| Other | Key |
| --- | --- |
| Pause / resume | `P` |
| Freeze player 2 | `B` |
| Quit | `Q` |
 
## How it works
 
### Drawing to memory
 
There is no graphics library. Every frame is drawn by storing colour values directly into the bitmap display's memory region.
 
| Setting | Value |
| --- | --- |
| Unit size | 8 × 8 pixels |
| Display size | 256 × 168 pixels (32 × 21 units) |
| Base address | `0x10008000` |
 
A helper converts a (row, column) position into a pixel address: `base + (row × width + col) × 4`. The renderer redraws only when the game state changes.
 
### Two boards, one set of routines
 
Each player's board is a block of memory with one word per cell. A parallel block holds per-cell **delete flags** used during matching. Address helpers pick the right base address for the current player. After that, the same routines for movement, collision, matching, gravity and garbage serve both players. The game loop updates player 1, switches the player selector, and updates player 2.
 
### Code organization
 
The program is split into about 90 small procedures, grouped by role:
 
| Area | Responsibility |
| --- | --- |
| Game loop & timing | Main loop, per-player fall timers, input and chain-animation pauses |
| Input | Key dispatcher plus one handler per key |
| Piece state | Save/load active column, shuffle, hard drop, lock into board, spawn |
| Piece queue | Random colours, preview queue initialisation and advance |
| Movement & collision | Bounds and collision checks for left, right, down and spawn |
| Board memory | Board/flag base addresses, cell address helpers, read/write cell colour |
| Rendering | Pixel addresses, board, active column, ghost, preview queue, borders |
| Matching | Count runs in four directions, flag matches, remove flagged gems, resolve chains |
| Gravity | Drop unsupported gems, per column and for the whole board |
| Garbage | Shift board up, fill bottom row, send garbage to opponent |
| UI | Pause screen, heart display, W/L result screen |
 
## What I learned
 
- **Memory is just addresses.** Laying out two boards and their flag arrays by hand, then computing every cell's address, made memory layout concrete in a way high-level languages hide.
- **Designing reusable assembly.** Running both players from one set of routines through a base-address selector avoided writing every routine twice. It also meant each procedure had to save and restore registers carefully.
- **Debugging without abstractions.** Chain reactions, gravity and garbage all change the same memory. Getting them to interact correctly meant stepping through the code in the simulator and watching memory change.
## Author
 
**Min Son**, Computer Science & Mathematics, University of Toronto
[GitHub](https://github.com/minson0530) · minn.son@mail.utoronto.ca
