# 2048
Tugas coding 2048 by
import curses
from random import choice, randrange


KEYMAP = {
    curses.KEY_UP: "UP",
    curses.KEY_DOWN: "DOWN",
    curses.KEY_LEFT: "LEFT",
    curses.KEY_RIGHT: "RIGHT",
    ord('q'): "QUIT",
    ord('r'): "RESTART",
}

def init_grid():
    grid = [[0] * 4 for _ in range(4)]
    add_tile(grid)
    add_tile(grid)
    return grid

def add_tile(grid):
    empty_cells = [(i, j) for i in range(4) for j in range(4) if grid[i][j] == 0]
    if not empty_cells:
        return
    i, j = choice(empty_cells)
    grid[i][j] = 4 if randrange(10) == 0 else 2

def compress(row):
    nonzeros = [v for v in row if v]
    return nonzeros + [0] * (4 - len(nonzeros))

def merge(row):
    for i in range(3):
        if row[i] and row[i] == row[i + 1]:
            row[i] *= 2
            row[i + 1] = 0
    return row

def move_left(grid):
    new_grid = []
    for row in grid:
        row = compress(row)
        row = merge(row)
        row = compress(row)
        new_grid.append(row)
    return new_grid

def rot90(grid):
    return [list(r) for r in zip(*grid[::-1])]
def move_grid(grid, direction):
    if direction == "LEFT":
        return move_left(grid)

    if direction == "RIGHT":
        reversed_rows = [row[::-1] for row in grid]
        moved = move_left(reversed_rows)
        return [row[::-1] for row in moved]

    if direction == "UP":
        rotated = rot90(rot90(rot90(grid)))
        moved = move_left(rotated)
        return rot90(moved)

    if direction == "DOWN":
        rotated = rot90(grid)
        moved = move_left(rotated)
        return rot90(rot90(rot90(moved)))

    return grid

def can_move(grid):
    for i in range(4):
        for j in range(4):
            value = grid[i][j]
            if value == 0:
                return True
            if j < 3 and value == grid[i][j + 1]:
                return True
            if i < 3 and value == grid[i + 1][j]:
                return True
    return False

def draw(stdscr, grid, score):
    stdscr.clear()
    stdscr.addstr("   2048  (q=Quit, r=Restart)\n")
    stdscr.addstr(f"   Score: {score}\n\n")

    for row in grid:
        stdscr.addstr("+------+------+------+------+ \n")
        stdscr.addstr("|")
        for value in row:
            cell = str(value).center(6) if value else " ".center(6)
            stdscr.addstr(cell + "|")
        stdscr.addstr("\n")
    stdscr.addstr("+------+------+------+------+ \n")
    stdscr.refresh()

def game_loop(stdscr):
    curses.curs_set(0)
    grid = init_grid()
    score = 0

    while True:
        draw(stdscr, grid, score)
        key = stdscr.getch()

        if key not in KEYMAP:
            continue

        action = KEYMAP[key]

        if action == "QUIT":
            return
        if action == "RESTART":
            grid = init_grid()
            score = 0
            continue

        new_grid = move_grid(grid, action)

        if new_grid != grid:
            score += sum(
                new_grid[i][j] - grid[i][j]
                for i in range(4)
                for j in range(4)
                if new_grid[i][j] > grid[i][j]
            )
            grid = new_grid
            add_tile(grid)

        if not can_move(grid):
            draw(stdscr, grid, score)
            stdscr.addstr("\n   GAME OVER! Press R to restart or Q to quit.\n")
            stdscr.refresh()

            while True:
                key = stdscr.getch()
                if key in KEYMAP and KEYMAP[key] == "RESTART":
                    grid = init_grid()
                    score = 0
                    break
                if key in KEYMAP and KEYMAP[key] == "QUIT":
                    return

curses.wrapper(game_loop)
