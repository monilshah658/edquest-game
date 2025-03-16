this is the code for the game
import pygame
import random

# Initialize Pygame
pygame.init()

# Constants
SCREEN_WIDTH = 600
SCREEN_HEIGHT = 400
SNAKE_SIZE = 10
SNAKE_SPEED = 15

# Colors
BLACK = (0, 0, 0)        # Black background
GREEN = (0, 255, 0)      # Vibrant green for the snake
BLUE = (0, 0, 255)       # Blue color for the coin

# Create the screen
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Snake Game")

# Snake class
class Snake:
    def __init__(self):
        self.body = [[100, 50], [90, 50], [80, 50]]  # Initial snake body
        self.direction = 'RIGHT'
        self.grow = False
        self.score = 0  # Initialize score

    def update(self):
        head = self.body[0]
        if self.direction == 'UP':
            new_head = [head[0], head[1] - SNAKE_SIZE]
        elif self.direction == 'DOWN':
            new_head = [head[0], head[1] + SNAKE_SIZE]
        elif self.direction == 'LEFT':
            new_head = [head[0] - SNAKE_SIZE, head[1]]
        elif self.direction == 'RIGHT':
            new_head = [head[0] + SNAKE_SIZE, head[1]]

        self.body.insert(0, new_head)
        if not self.grow:
            self.body.pop()  # Remove the last segment if not growing
        else:
            self.grow = False  # Reset grow flag

    def change_direction(self, new_direction):
        # Prevent the snake from reversing
        if (new_direction == 'UP' and not self.direction == 'DOWN') or \
           (new_direction == 'DOWN' and not self.direction == 'UP') or \
           (new_direction == 'LEFT' and not self.direction == 'RIGHT') or \
           (new_direction == 'RIGHT' and not self.direction == 'LEFT'):
            self.direction = new_direction

    def grow_snake(self):
        self.grow = True

    def draw(self):
        # Draw the head of the snake with eyes
        pygame.draw.rect(screen, GREEN, (self.body[0][0], self.body[0][1], SNAKE_SIZE, SNAKE_SIZE))
        eye_offset = SNAKE_SIZE // 4
        pygame.draw.circle(screen, BLACK, (self.body[0][0] + eye_offset, self.body[0][1] + eye_offset), 2)
        pygame.draw.circle(screen, BLACK, (self.body[0][0] + SNAKE_SIZE - eye_offset, self.body[0][1] + eye_offset), 2)

        # Draw the body segments
        for i, segment in enumerate(self.body[1:]):
            if i == len(self.body) - 2:  # Tail
                pygame.draw.polygon(screen, GREEN, [(segment[0], segment[1]), 
                                                     (segment[0] + SNAKE_SIZE, segment[1] + SNAKE_SIZE // 2), 
                                                     (segment[0], segment[1] + SNAKE_SIZE)])
            else:
                pygame.draw.rect(screen, GREEN, (segment[0], segment[1], SNAKE_SIZE, SNAKE_SIZE))

# Coin class
class Coin:
    def __init__(self):
        self.position = [random.randrange(1, (SCREEN_WIDTH // SNAKE_SIZE)) * SNAKE_SIZE,
                         random.randrange(1, (SCREEN_HEIGHT // SNAKE_SIZE)) * SNAKE_SIZE]

    def spawn(self):
        self.position = [random.randrange(1, (SCREEN_WIDTH // SNAKE_SIZE)) * SNAKE_SIZE,
                         random.randrange(1, (SCREEN_HEIGHT // SNAKE_SIZE)) * SNAKE_SIZE]

    def draw(self):
        # Draw the coin as a diamond shape
        points = [
            (self.position[0] + SNAKE_SIZE // 2, self.position[1]),  # Top
            (self.position[0] + SNAKE_SIZE, self.position[1] + SNAKE_SIZE // 2),  # Right
            (self.position[0] + SNAKE_SIZE // 2, self.position[1] + SNAKE_SIZE),  # Bottom
            (self.position[0], self.position[1] + SNAKE_SIZE // 2)  # Left
        ]
        pygame.draw.polygon(screen, BLUE, points)  # Blue diamond

# Game loop
def main(): 
    snake = Snake()
    coin = Coin()
    clock = pygame.time.Clock()
    running = True
    timer = 300  # Timer set for 5 minutes (300 seconds)

    while running: 
        for event in pygame.event.get(): 
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_UP:
                    snake.change_direction('UP')
                elif event.key == pygame.K_DOWN:
                    snake.change_direction('DOWN')
                elif event.key == pygame.K_LEFT:
                    snake.change_direction('LEFT')
                elif event.key == pygame.K_RIGHT:
                    snake.change_direction('RIGHT')

        # Update the timer
        timer -= 1 / SNAKE_SPEED  # Decrease timer based on frame rate
        if timer <= 0:
            running = False  # End the game when time is up

        snake.update()

        # Check for collision with the coin and update score
        if snake.body[0] == coin.position:
            snake.grow_snake()
            coin.spawn()
            snake.score += 1  # Increment score

        # Check for collision with itself
        if snake.body[0] in snake.body[1:] or \
           snake.body[0][0] < 0 or snake.body[0][0] >= SCREEN_WIDTH or \
           snake.body[0][1] < 0 or snake.body[0][1] >= SCREEN_HEIGHT:

            running = False  # End the game on collision

        # Clear the screen and display score
        screen.fill(BLACK)
        font = pygame.font.SysFont(None, 35)
        score_text = font.render(f'Score: {snake.score}', True, GREEN)
        screen.blit(score_text, (10, 10))  # Display score at the top left

        # Display the timer
        timer_text = font.render(f'Time: {int(timer)}', True, GREEN)
        screen.blit(timer_text, (SCREEN_WIDTH - 150, 10))  # Display timer at a slightly left position


        snake.draw()
        coin.draw()

        # Update the display
        pygame.display.flip()
        clock.tick(SNAKE_SPEED)

    pygame.quit()

if __name__ == "__main__":
    main()
