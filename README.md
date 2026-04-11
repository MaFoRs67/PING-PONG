# PING-PONG
full code 
import pygame
import sys
import random

# Инициализация pygame
pygame.init()

# Константы
WIDTH, HEIGHT = 800, 600
PADDLE_W, PADDLE_H = 15, 100
BALL_SIZE = 15
PADDLE_SPEED = 7
BALL_SPEED = 5
BOT_SPEED = 5.5  # Скорость бота (чуть ниже игрока для баланса)
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
GRAY = (120, 120, 120)

# Создание окна
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Пинг-Понг")
clock = pygame.time.Clock()
font = pygame.font.SysFont(None, 60)
small_font = pygame.font.SysFont(None, 40)

# Инициализация объектов
left_paddle = pygame.Rect(20, HEIGHT // 2 - PADDLE_H // 2, PADDLE_W, PADDLE_H)
right_paddle = pygame.Rect(WIDTH - 20 - PADDLE_W, HEIGHT // 2 - PADDLE_H // 2, PADDLE_W, PADDLE_H)
ball = pygame.Rect(WIDTH // 2 - BALL_SIZE // 2, HEIGHT // 2 - BALL_SIZE // 2, BALL_SIZE, BALL_SIZE)

ball_dx = BALL_SPEED
ball_dy = BALL_SPEED
left_score = 0
right_score = 0

game_mode = None  # "2p" или "bot"
menu_active = True

def reset_ball():
    global ball_dx, ball_dy
    ball.center = (WIDTH // 2, HEIGHT // 2)
    ball_dx = -ball_dx  # Меняем направление подачи
    ball_dy = BALL_SPEED * (1 if random.random() > 0.5 else -1)

# Игровой цикл
while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

        # Обработка ввода в меню
        if menu_active and event.type == pygame.KEYDOWN:
            if event.key == pygame.K_1:
                game_mode = "2p"
                menu_active = False
            elif event.key == pygame.K_2:
                game_mode = "bot"
                menu_active = False
            elif event.key == pygame.K_ESCAPE:
                pygame.quit()
                sys.exit()

    # === МЕНЮ ===
    if menu_active:
        screen.fill(BLACK)
        title = font.render("ПИНГ-ПОНГ", True, WHITE)
        opt1 = small_font.render("Нажми 1 - Игра вдвоем", True, WHITE)
        opt2 = small_font.render("Нажми 2 - Игра с ботом", True, GRAY)
        esc_text = small_font.render("ESC - Выход", True, GRAY)
        
        screen.blit(title, (WIDTH//2 - title.get_width()//2, HEIGHT//3))
        screen.blit(opt1, (WIDTH//2 - opt1.get_width()//2, HEIGHT//2))
        screen.blit(opt2, (WIDTH//2 - opt2.get_width()//2, HEIGHT//2 + 50))
        screen.blit(esc_text, (WIDTH//2 - esc_text.get_width()//2, HEIGHT - 80))
        
        pygame.display.flip()
        clock.tick(60)
        continue

    # === ИГРА ===
    keys = pygame.key.get_pressed()

    # Левый игрок (всегда управляется клавиатурой)
    if keys[pygame.K_w] and left_paddle.top > 0:
        left_paddle.y -= PADDLE_SPEED
    if keys[pygame.K_s] and left_paddle.bottom < HEIGHT:
        left_paddle.y += PADDLE_SPEED

    # Правая ракетка: игрок 2 или бот
    if game_mode == "2p":
        if keys[pygame.K_UP] and right_paddle.top > 0:
            right_paddle.y -= PADDLE_SPEED
        if keys[pygame.K_DOWN] and right_paddle.bottom < HEIGHT:
            right_paddle.y += PADDLE_SPEED
    elif game_mode == "bot":
        # Логика бота
        if right_paddle.centery < ball.centery - 10:
            right_paddle.y += BOT_SPEED
        elif right_paddle.centery > ball.centery + 10:
            right_paddle.y -= BOT_SPEED
        
        # Ограничение выхода за экран
        right_paddle.clamp_ip(pygame.Rect(0, 0, WIDTH, HEIGHT))

    # Движение мяча
    ball.x += ball_dx
    ball.y += ball_dy

    # Отскок от верха/низа
    if ball.top <= 0 or ball.bottom >= HEIGHT:
        ball_dy *= -1

    # Отскок от ракеток
    if ball.colliderect(left_paddle) or ball.colliderect(right_paddle):
        ball_dx *= -1
        # Коррекция позиции, чтобы мяч не "застревал"
        if ball_dx > 0:
            ball.left = right_paddle.left - ball.width
        else:
            ball.right = left_paddle.right

    # Голы
    if ball.left <= 0:
        right_score += 1
        reset_ball()
    if ball.right >= WIDTH:
        left_score += 1
        reset_ball()

    # Отрисовка
    screen.fill(BLACK)
    pygame.draw.rect(screen, WHITE, left_paddle)
    pygame.draw.rect(screen, WHITE, right_paddle)
    pygame.draw.ellipse(screen, WHITE, ball)
    
    # Центральная линия
    for y in range(0, HEIGHT, 30):
        pygame.draw.line(screen, WHITE, (WIDTH // 2, y), (WIDTH // 2, y + 15))

    # Счёт
    left_text = font.render(str(left_score), True, WHITE)
    right_text = font.render(str(right_score), True, WHITE)
    screen.blit(left_text, (WIDTH // 4 - left_text.get_width() // 2, 20))
    screen.blit(right_text, (3 * WIDTH // 4 - right_text.get_width() // 2, 20))

    # Индикатор режима
    mode_str = "Режим: 2 Игрока (W/S + ↑/↓)" if game_mode == "2p" else "Режим: vs Бот (W/S)"
    mode_text = small_font.render(mode_str, True, GRAY)
    screen.blit(mode_text, (WIDTH//2 - mode_text.get_width()//2, HEIGHT - 40))

    pygame.display.flip()
    clock.tick(60)
