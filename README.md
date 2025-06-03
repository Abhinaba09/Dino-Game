import pygame
import random
import os

# Initialize Pygame
pygame.init()

# Screen
WIDTH, HEIGHT = 800, 400
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Chrome Dino Simulation")

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
BLUE = (0, 100, 255)

# Fonts
FONT = pygame.font.SysFont("Arial", 24)
BIG_FONT = pygame.font.SysFont("Arial", 36)

# Clock
clock = pygame.time.Clock()

# Sounds
jump_sound = pygame.mixer.Sound("jump.wav")
gameover_sound = pygame.mixer.Sound("gameover.wav")

# Dino
class Dino(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = pygame.Surface((50, 50))
        self.image.fill(BLACK)
        self.rect = self.image.get_rect(midbottom=(80, HEIGHT - 20))
        self.gravity = 0
        self.is_jumping = False

    def jump(self):
        if not self.is_jumping:
            jump_sound.play()
            self.gravity = -15
            self.is_jumping = True

    def update(self):
        self.gravity += 1
        self.rect.y += self.gravity
        if self.rect.bottom >= HEIGHT - 20:
            self.rect.bottom = HEIGHT - 20
            self.is_jumping = False

# Obstacle
class Obstacle(pygame.sprite.Sprite):
    def __init__(self, speed):
        super().__init__()
        self.image = pygame.Surface((30, 60))
        self.image.fill((255, 0, 0))
        self.rect = self.image.get_rect(midbottom=(WIDTH + 30, HEIGHT - 20))
        self.speed = speed

    def update(self):
        self.rect.x -= self.speed
        if self.rect.right < 0:
            self.kill()

# PowerUp
class PowerUp(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = pygame.Surface((25, 25))
        self.image.fill((0, 255, 255))
        self.rect = self.image.get_rect(center=(WIDTH + 30, HEIGHT - 80))
        self.speed = 5

    def update(self):
        self.rect.x -= self.speed
        if self.rect.right < 0:
            self.kill()

# Text
def draw_text(text, font, color, surface, x, y):
    render = font.render(text, True, color)
    surface.blit(render, (x, y))

# Save score
def save_score(score):
    with open("leaderboard.txt", "a") as f:
        f.write(str(score) + "\n")

# Load top scores
def load_leaderboard():
    if not os.path.exists("leaderboard.txt"):
        return []
    with open("leaderboard.txt", "r") as f:
        scores = [int(line.strip()) for line in f if line.strip().isdigit()]
    return sorted(scores, reverse=True)[:5]

# Show leaderboard
def show_leaderboard():
    screen.fill(WHITE)
    draw_text("LEADERBOARD", BIG_FONT, BLACK, screen, WIDTH//2 - 100, 50)
    scores = load_leaderboard()
    for i, s in enumerate(scores):
        draw_text(f"{i+1}. {s}", FONT, BLACK, screen, WIDTH//2 - 40, 100 + i*30)
    draw_text("Press SPACE to go back", FONT, BLACK, screen, WIDTH//2 - 100, HEIGHT - 50)
    pygame.display.flip()
    wait = True
    while wait:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
            if event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE:
                wait = False

# Start menu
def show_start_menu():
    screen.fill(WHITE)
    draw_text("CHROME DINO", BIG_FONT, BLACK, screen, WIDTH//2 - 100, 50)
    draw_text("SPACE: Start  |  L: Leaderboard", FONT, BLACK, screen, WIDTH//2 - 150, 120)
    draw_text("Jump over obstacles. Collect power-ups!", FONT, BLUE, screen, WIDTH//2 - 180, 180)
    pygame.display.flip()

# Game loop
def run_game():
    dino = Dino()
    all_sprites = pygame.sprite.Group(dino)
    obstacles = pygame.sprite.Group()
    powerups = pygame.sprite.Group()

    score = 0
    obstacle_speed = 7
    slow_timer = 0

    SPAWN_OBSTACLE = pygame.USEREVENT + 1
    SPAWN_POWERUP = pygame.USEREVENT + 2
    pygame.time.set_timer(SPAWN_OBSTACLE, 1500)
    pygame.time.set_timer(SPAWN_POWERUP, 7000)

    running = True
    while running:
        clock.tick(60)
        print("Game running...")  # Debug line
        screen.fill(WHITE)        # Clear screen

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
            if event.type == SPAWN_OBSTACLE:
                o = Obstacle(obstacle_speed if not slow_timer else 3)
                all_sprites.add(o)
                obstacles.add(o)
            if event.type == SPAWN_POWERUP:
                p = PowerUp()
                all_sprites.add(p)
                powerups.add(p)
            if event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE:
                dino.jump()

        # Speed increases over time
        if score % 500 == 0 and score != 0 and not slow_timer:
            obstacle_speed += 0.5

        all_sprites.update()
        all_sprites.draw(screen)

        # Collision check
        if pygame.sprite.spritecollideany(dino, obstacles):
            gameover_sound.play()
            pygame.time.delay(1000)
            save_score(score // 10)
            running = False

        # Power-up
        if pygame.sprite.spritecollideany(dino, powerups):
            for p in powerups:
                p.kill()
            slow_timer = 180

        if slow_timer > 0:
            slow_timer -= 1

        # Score display
        score += 1
        draw_text(f"Score: {score // 10}", FONT, BLACK, screen, 10, 10)
        if slow_timer:
            draw_text("Slow-Mo!", FONT, BLUE, screen, WIDTH - 140, 10)

        pygame.display.flip()

# Main loop
while True:
    show_start_menu()
    waiting = True
    while waiting:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE:
                    run_game()
                    waiting = False
                elif event.key == pygame.K_l:
                    show_leaderboard()
