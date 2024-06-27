# tetris-oyunu
import pygame
import random

# Global constants
screen_width = 800
screen_height = 600
block_size = 30
cols = 10
rows = 20

# Colors
colors = [
    (0, 0, 0),
    (255, 0, 0),
    (0, 255, 0),
    (0, 0, 255),
    (255, 255, 0),
    (0, 255, 255),
    (255, 0, 255),
    (255, 255, 255)
]

# Tetromino shapes
shapes = [
    [[1, 1, 1], [0, 1, 0]],  # T-shape
    [[1, 1, 1, 1]],          # I-shape
    [[1, 1], [1, 1]],        # O-shape
    [[0, 1, 1], [1, 1, 0]],  # S-shape
    [[1, 1, 0], [0, 1, 1]],  # Z-shape
    [[1, 1, 1], [1, 0, 0]],  # L-shape
    [[1, 1, 1], [0, 0, 1]]   # J-shape
]

class Tetromino:
    def __init__(self, shape):
        self.shape = shape
        self.color = random.randint(1, len(colors) - 1)
        self.rotation = 0
        self.x = cols // 2 - len(shape[0]) // 2
        self.y = 0

    def rotate(self):
        self.shape = [list(row) for row in zip(*self.shape[::-1])]

class Tetris:
    def __init__(self):
        self.grid = [[0 for _ in range(cols)] for _ in range(rows)]
        self.current_tetromino = Tetromino(random.choice(shapes))
        self.next_tetromino = Tetromino(random.choice(shapes))
        self.score = 0
        self.game_over_flag = False

    def new_tetromino(self):
        self.current_tetromino = self.next_tetromino
        self.next_tetromino = Tetromino(random.choice(shapes))
        if self.check_collision(self.current_tetromino):
            self.game_over_flag = True

    def check_collision(self, tetromino):
        for y, row in enumerate(tetromino.shape):
            for x, cell in enumerate(row):
                if cell and (y + tetromino.y >= rows or
                             x + tetromino.x < 0 or
                             x + tetromino.x >= cols or
                             self.grid[y + tetromino.y][x + tetromino.x]):
                    return True
        return False

    def freeze_tetromino(self):
        for y, row in enumerate(self.current_tetromino.shape):
            for x, cell in enumerate(row):
                if cell:
                    self.grid[y + self.current_tetromino.y][x + self.current_tetromino.x] = self.current_tetromino.color
        self.clear_lines()
        self.new_tetromino()

    def clear_lines(self):
        lines = 0
        for y in range(rows):
            if 0 not in self.grid[y]:
                del self.grid[y]
                self.grid.insert(0, [0 for _ in range(cols)])
                lines += 1
        self.score += lines ** 2

    def move_tetromino(self, dx, dy):
        self.current_tetromino.x += dx
        self.current_tetromino.y += dy
        if self.check_collision(self.current_tetromino):
            self.current_tetromino.x -= dx
            self.current_tetromino.y -= dy
            return False
        return True

    def drop_tetromino(self):
        if not self.move_tetromino(0, 1):
            self.freeze_tetromino()

    def rotate_tetromino(self):
        self.current_tetromino.rotate()
        if self.check_collision(self.current_tetromino):
            for _ in range(3):
                self.current_tetromino.rotate()

def draw_grid(surface, grid):
    for y, row in enumerate(grid):
        for x, cell in enumerate(row):
            pygame.draw.rect(surface, colors[cell], (x * block_size, y * block_size, block_size, block_size))

def draw_tetromino(surface, tetromino):
    for y, row in enumerate(tetromino.shape):
        for x, cell in enumerate(row):
            if cell:
                pygame.draw.rect(surface, colors[tetromino.color], ((tetromino.x + x) * block_size, (tetromino.y + y) * block_size, block_size, block_size))

def draw_text(surface, text, size, color, x, y):
    font = pygame.font.Font(pygame.font.get_default_font(), size)
    text_surface = font.render(text, True, color)
    surface.blit(text_surface, (x, y))

def main():
    pygame.init()
    screen = pygame.display.set_mode((cols * block_size, rows * block_size))
    clock = pygame.time.Clock()
    tetris = Tetris()
    fall_time = 0
    fall_speed = 0.3

    start_game = False
    running = True

    while running:
        screen.fill((0, 0, 0))
        fall_time += clock.get_rawtime()
        clock.tick()

        if not start_game:
            draw_text(screen, "TETRIS", 50, (255, 255, 255), screen_width // 3, screen_height // 4)
            draw_text(screen, "Press any key to start", 30, (255, 255, 255), screen_width // 4, screen_height // 2)
            pygame.display.flip()
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    running = False
                elif event.type == pygame.KEYDOWN:
                    start_game = True
        else:
            if fall_time / 1000 >= fall_speed:
                fall_time = 0
                tetris.drop_tetromino()

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    running = False
                elif event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_LEFT:
                        tetris.move_tetromino(-1, 0)
                    elif event.key == pygame.K_RIGHT:
                        tetris.move_tetromino(1, 0)
                    elif event.key == pygame.K_DOWN:
                        tetris.drop_tetromino()
                    elif event.key == pygame.K_UP:
                        tetris.rotate_tetromino()

            if tetris.game_over_flag:
                draw_text(screen, "GAME OVER", 50, (255, 0, 0), screen_width // 3, screen_height // 3)
                draw_text(screen, f"Score: {tetris.score}", 30, (255, 255, 255), screen_width // 3, screen_height // 2)
                draw_text(screen, "Press any key to restart", 30, (255, 255, 255), screen_width // 4, screen_height // 1.5)
                pygame.display.flip()
                for event in pygame.event.get():
                    if event.type == pygame.QUIT:
                        running = False
                    elif event.type == pygame.KEYDOWN:
                        tetris = Tetris()
            else:
                draw_grid(screen, tetris.grid)
                draw_tetromino(screen, tetris.current_tetromino)
                draw_text(screen, f"Score: {tetris.score}", 30, (255, 255, 255), 10, 10)
                pygame.display.flip()

    pygame.quit()

if __name__ == "__main__":
    main()
