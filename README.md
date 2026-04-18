# pong_simple.py — упрощенная версия
import pygame, random, sys

pygame.init()
pygame.mixer.init()

W, H = 800, 600
screen = pygame.display.set_mode((W, H))
pygame.display.set_caption("Pong")
clock = pygame.time.Clock()
font = pygame.font.Font(None, 74)

WHITE, BLACK = (255, 255, 255), (0, 0, 0)

# Простой звук (короткий писк)
bounce_sound = pygame.mixer.Sound(buffer=bytes([int(x*127+128) for x in [0]*1000 + [1]*500 + [0]*1000]))

# Простое изображение шарика (белый круг)
ball_img = pygame.Surface((25, 25))
ball_img.fill(BLACK)
pygame.draw.circle(ball_img, WHITE, (12, 12), 12)

# Ракетки
paddle_w, paddle_h = 15, 100
left = pygame.Rect(20, H//2-50, paddle_w, paddle_h)
right = pygame.Rect(W-35, H//2-50, paddle_w, paddle_h)
ball = pygame.Rect(W//2-12, H//2-12, 25, 25)

# Параметры
ball_vx, ball_vy = 5, 5
paddle_speed = 7
score_l = score_r = 0
game_mode = None  # "2p" или "bot"

def reset_ball():
    global ball_vx, ball_vy
    ball.center = (W//2, H//2)
    ball_vx = -ball_vx
    ball_vy = random.choice([-5, 5])

# Меню
in_menu = True
while in_menu:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_1:
                game_mode, in_menu = "2p", False
            elif event.key == pygame.K_2:
                game_mode, in_menu = "bot", False
    
    screen.fill(BLACK)
    screen.blit(font.render("PONG", True, WHITE), (W//2-70, 150))
    screen.blit(font.render("1 - 2 Players", True, WHITE), (W//2-120, 300))
    screen.blit(font.render("2 - vs Bot", True, WHITE), (W//2-100, 380))
    pygame.display.flip()
    clock.tick(60)

# Основная игра
while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
    
    # Управление
    keys = pygame.key.get_pressed()
    if keys[pygame.K_w] and left.top > 0: left.y -= paddle_speed
    if keys[pygame.K_s] and left.bottom < H: left.y += paddle_speed
    
    if game_mode == "2p":
        if keys[pygame.K_UP] and right.top > 0: right.y -= paddle_speed
        if keys[pygame.K_DOWN] and right.bottom < H: right.y += paddle_speed
    else:  # Bot
        if right.centery < ball.centery: right.y += 5
        if right.centery > ball.centery: right.y -= 5
        right.y = max(0, min(H - paddle_h, right.y))
    
    # Движение шарика
    ball.x += ball_vx
    ball.y += ball_vy
    
    # Отскоки
    if ball.top <= 0 or ball.bottom >= H:
        ball_vy = -ball_vy
        bounce_sound.play()
    
    if ball.colliderect(left) or ball.colliderect(right):
        ball_vx = -ball_vx
        bounce_sound.play()
    
    # Голы
    if ball.left <= 0:
        score_r += 1
        reset_ball()
    if ball.right >= W:
        score_l += 1
        reset_ball()
    
    # Отрисовка
    screen.fill(BLACK)
    pygame.draw.rect(screen, WHITE, left)
    pygame.draw.rect(screen, WHITE, right)
    screen.blit(ball_img, (ball.x, ball.y))
    
    # Линия и счет
    for y in range(0, H, 40):
        pygame.draw.line(screen, WHITE, (W//2, y), (W//2, y+20))
    screen.blit(font.render(str(score_l), True, WHITE), (W//4, 20))
    screen.blit(font.render(str(score_r), True, WHITE), (3*W//4, 20))
    
    pygame.display.flip()
    clock.tick(60)
