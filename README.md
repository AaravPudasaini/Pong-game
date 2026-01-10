# Pong-game
I made this Pong game with python and pygame


[Pong game by Aarav Pudasaini.py](https://github.com/user-attachments/files/24544270/Pong.game.by.Aarav.Pudasaini.py)
import pygame
import sys
import math
import os
import turtle

pygame.init()

# screen
WIDTH, HEIGHT = 800, 500
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Pong")

# colour
GREEN = (0, 120, 0)
WHITE = (255, 255, 255)
BLUE = (0, 0, 255)
RED = (255, 0, 0)
BLACK = (0, 0, 0)

# font
title_font = pygame.font.SysFont("arial", 40)
text_font = pygame.font.SysFont("arial", 22)
score_font = pygame.font.SysFont("arial", 26)

# clock
clock = pygame.time.Clock()

# game state
INTRO = "intro"
NAME = "name"
GAME = "game"
END = "end"
state = INTRO

# player
player_name = ""

# score
player_score = 0
cpu_score = 0
WIN_SCORE = 10
winner = ""

# paddle
PADDLE_WIDTH, PADDLE_HEIGHT = 15, 100
left_paddle = pygame.Rect(40, HEIGHT//2 - PADDLE_HEIGHT//2, PADDLE_WIDTH, PADDLE_HEIGHT)
right_paddle = pygame.Rect(WIDTH - 55, HEIGHT//2 - PADDLE_HEIGHT//2, PADDLE_WIDTH, PADDLE_HEIGHT)
PADDLE_SPEED = 5
CPU_SPEED = 5

# ball
BALL_RADIUS = 10
ball_x, ball_y = WIDTH // 2, HEIGHT // 2
BALL_SPEED_X = 5
BALL_SPEED_Y = 5


NET_WIDTH = 10

# button
yes_button = pygame.Rect(WIDTH//2 - 120, 280, 100, 50)
no_button = pygame.Rect(WIDTH//2 + 20, 280, 100, 50)

# funtion
def reset_game():
    global player_score, cpu_score, ball_x, ball_y, BALL_SPEED_X, BALL_SPEED_Y
    player_score = 0
    cpu_score = 0
    ball_x, ball_y = WIDTH // 2, HEIGHT // 2
    BALL_SPEED_X = 5
    BALL_SPEED_Y = 5

def draw_intro():
    screen.fill(GREEN)
    lines = [
        "Welcome to Pong",
        "by Aarav Pudasaini",
        "You will be playing against a CPU.",
        "Use the'W' key to go up and 'S' keyto go down",
        "First to 10 points wins.",
        "Press SPACE to continue"
    ]
    y = 100
    for i, text in enumerate(lines):
        font = title_font if i == 0 else text_font
        rendered = font.render(text, True, WHITE)
        screen.blit(rendered, (WIDTH//2 - rendered.get_width()//2, y))
        y += 45

def draw_name_screen():
    screen.fill(GREEN)
    prompt = text_font.render("Enter your name and press ENTER:", True, WHITE)
    name_text = title_font.render(player_name, True, WHITE)
    screen.blit(prompt, (WIDTH//2 - prompt.get_width()//2, 180))
    screen.blit(name_text, (WIDTH//2 - name_text.get_width()//2, 230))

def draw_scoreboard():
    score_text = score_font.render(
        f"{player_name}: {player_score}  |  CPU: {cpu_score}",
        True,
        WHITE
    )
    screen.blit(score_text, (WIDTH - score_text.get_width() - 20, 20))

def draw_end_screen():
    screen.fill(GREEN)

    win_text = title_font.render(winner, True, WHITE)
    question = text_font.render("Would you like to try again?", True, WHITE)

    screen.blit(win_text, (WIDTH//2 - win_text.get_width()//2, 150))
    screen.blit(question, (WIDTH//2 - question.get_width()//2, 210))

    pygame.draw.rect(screen, BLUE, yes_button)
    pygame.draw.rect(screen, RED, no_button)

    yes_text = text_font.render("YES", True, WHITE)
    no_text = text_font.render("NO", True, WHITE)

    screen.blit(yes_text, (yes_button.centerx - yes_text.get_width()//2,
                            yes_button.centery - yes_text.get_height()//2))
    screen.blit(no_text, (no_button.centerx - no_text.get_width()//2,
                           no_button.centery - no_text.get_height()//2))

# loop
while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

        if event.type == pygame.KEYDOWN:
            if state == INTRO and event.key == pygame.K_SPACE:
                state = NAME
            elif state == NAME:
                if event.key == pygame.K_RETURN and player_name != "":
                    state = GAME
                elif event.key == pygame.K_BACKSPACE:
                    player_name = player_name[:-1]
                else:
                    if len(player_name) < 12:
                        player_name += event.unicode

        if event.type == pygame.MOUSEBUTTONDOWN and state == END:
            if yes_button.collidepoint(event.pos):
                reset_game()
                state = GAME
            elif no_button.collidepoint(event.pos):
                pygame.quit()
                sys.exit()

    
    if state == GAME:
        keys = pygame.key.get_pressed()

        if keys[pygame.K_w] and left_paddle.top > 0:
            left_paddle.y -= PADDLE_SPEED
        if keys[pygame.K_s] and left_paddle.bottom < HEIGHT:
            left_paddle.y += PADDLE_SPEED

        if ball_y < right_paddle.centery and right_paddle.top > 0:
            right_paddle.y -= CPU_SPEED
        elif ball_y > right_paddle.centery and right_paddle.bottom < HEIGHT:
            right_paddle.y += CPU_SPEED

        ball_x += BALL_SPEED_X
        ball_y += BALL_SPEED_Y

        if ball_y - BALL_RADIUS <= 0 or ball_y + BALL_RADIUS >= HEIGHT:
            BALL_SPEED_Y *= -1

        if left_paddle.collidepoint(ball_x - BALL_RADIUS, ball_y):
            BALL_SPEED_X *= -1
        if right_paddle.collidepoint(ball_x + BALL_RADIUS, ball_y):
            BALL_SPEED_X *= -1

        if ball_x < 0:
            cpu_score += 1
            ball_x, ball_y = WIDTH // 2, HEIGHT // 2
        elif ball_x > WIDTH:
            player_score += 1
            ball_x, ball_y = WIDTH // 2, HEIGHT // 2

        if player_score == WIN_SCORE:
            winner = f"{player_name} Wins!"
            state = END
        elif cpu_score == WIN_SCORE:
            winner = "CPU Wins!"
            state = END

    
    if state == INTRO:
        draw_intro()
    elif state == NAME:
        draw_name_screen()
    elif state == GAME:
        screen.fill(GREEN)
        pygame.draw.rect(screen, WHITE, (WIDTH//2 - NET_WIDTH//2, 0, NET_WIDTH, HEIGHT))
        pygame.draw.rect(screen, BLUE, left_paddle)
        pygame.draw.rect(screen, RED, right_paddle)
        pygame.draw.circle(screen, WHITE, (ball_x, ball_y), BALL_RADIUS)
        draw_scoreboard()
    elif state == END:
        draw_end_screen()

    pygame.display.flip()
    clock.tick(60)
