# Game bắn súng đơn giản - Pygame
# Chạy được trên Python có cài pygame (pip install pygame)

import pygame
import random
import math

# Khởi tạo pygame
pygame.init()

# Cài đặt màn hình
WIDTH = 800
HEIGHT = 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Game Bắn Súng Siêu Đơn Giản")

clock = pygame.time.Clock()
FPS = 60

# Màu sắc
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (220, 20, 60)
BLUE = (0, 120, 255)
YELLOW = (255, 215, 0)

# Font
font = pygame.font.SysFont("Arial", 36, bold=True)
small_font = pygame.font.SysFont("Arial", 24)

# Lớp Player (người chơi)
class Player(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = pygame.Surface((50, 50), pygame.SRCALPHA)
        pygame.draw.circle(self.image, BLUE, (25, 25), 25)
        pygame.draw.polygon(self.image, YELLOW, [(25, 10), (40, 25), (25, 40)])
        self.rect = self.image.get_rect(center=(WIDTH//2, HEIGHT//2))
        self.speed = 5

    def update(self):
        keys = pygame.key.get_pressed()
        if keys[pygame.K_a] or keys[pygame.K_LEFT]:
            self.rect.x -= self.speed
        if keys[pygame.K_d] or keys[pygame.K_RIGHT]:
            self.rect.x += self.speed
        if keys[pygame.K_w] or keys[pygame.K_UP]:
            self.rect.y -= self.speed
        if keys[pygame.K_s] or keys[pygame.K_DOWN]:
            self.rect.y += self.speed

        # Giới hạn không ra ngoài màn hình
        self.rect.clamp_ip(screen.get_rect())

# Lớp Đạn
class Bullet(pygame.sprite.Sprite):
    def __init__(self, x, y, direction):
        super().__init__()
        self.image = pygame.Surface((12, 12), pygame.SRCALPHA)
        pygame.draw.circle(self.image, YELLOW, (6, 6), 6)
        self.rect = self.image.get_rect(center=(x, y))
        self.speed = 12
        self.vel_x = math.cos(math.radians(direction)) * self.speed
        self.vel_y = -math.sin(math.radians(direction)) * self.speed  # góc 0 là bên phải

    def update(self):
        self.rect.x += self.vel_x
        self.rect.y += self.vel_y
        if not screen.get_rect().colliderect(self.rect):
            self.kill()

# Lớp Enemy (kẻ địch)
class Enemy(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = pygame.Surface((40, 40), pygame.SRCALPHA)
        pygame.draw.circle(self.image, RED, (20, 20), 20)
        self.rect = self.image.get_rect(center=(random.randint(50, WIDTH-50), random.randint(-100, -40)))
        self.speed = random.uniform(1.8, 3.2)

    def update(self):
        # Di chuyển xuống dưới
        self.rect.y += self.speed
        if self.rect.top > HEIGHT + 20:
            self.kill()

# Nhóm sprite
all_sprites = pygame.sprite.Group()
bullets = pygame.sprite.Group()
enemies = pygame.sprite.Group()

player = Player()
all_sprites.add(player)

# Biến game
score = 0
running = True
shoot_cooldown = 0
spawn_timer = 0

print("Điều khiển:")
print("  WASD / mũi tên → di chuyển")
print("  Chuột trái → bắn")
print("  Nhấn ESC hoặc đóng cửa sổ → thoát\n")

while running:
    dt = clock.tick(FPS)
    # ---------------- EVENT ----------------
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_ESCAPE:
                running = False

    # Bắn khi giữ chuột trái
    mouse_pressed = pygame.mouse.get_pressed()
    if mouse_pressed[0] and shoot_cooldown <= 0:
        # Tính góc từ player đến chuột
        mx, my = pygame.mouse.get_pos()
        dx = mx - player.rect.centerx
        dy = my - player.rect.centery
        angle = math.degrees(math.atan2(-dy, dx))   # góc theo hệ độ

        bullet = Bullet(player.rect.centerx, player.rect.centery, angle)
        all_sprites.add(bullet)
        bullets.add(bullet)
        shoot_cooldown = 12   # frames cooldown (~0.2 giây)

    if shoot_cooldown > 0:
        shoot_cooldown -= 1

    # Sinh enemy
    spawn_timer += 1
    if spawn_timer > 45:   # khoảng 0.75 giây sinh 1 con
        enemy = Enemy()
        all_sprites.add(enemy)
        enemies.add(enemy)
        spawn_timer = random.randint(-15, 10)  # tạo chút ngẫu nhiên

    # ---------------- UPDATE ----------------
    all_sprites.update()

    # Va chạm đạn - enemy
    hits = pygame.sprite.groupcollide(enemies, bullets, True, True)
    for hit in hits:
        score += 10

    # Va chạm player - enemy → game over
    if pygame.sprite.spritecollideany(player, enemies):
        running = False
        print(f"\nGAME OVER!   Điểm của bạn: {score}\n")

    # ---------------- DRAW ----------------
    screen.fill(BLACK)

    all_sprites.draw(screen)

    # Hiển thị điểm
    score_text = font.render(f"SCORE: {score}", True, WHITE)
    screen.blit(score_text, (20, 15))

    # Hướng dẫn nhỏ
    tip = small_font.render("Chuột trái = bắn   WASD = di chuyển", True, (180,180,180))
    screen.blit(tip, (WIDTH - tip.get_width() - 20, HEIGHT - 40))

    pygame.display.flip()

pygame.quit()
print("Cảm ơn bạn đã chơi!")
