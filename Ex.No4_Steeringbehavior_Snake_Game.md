# Ex.No: 4  Implementation of Snake game using Steering behaviors
### DATE:13-09-2024                                                                           
### REGISTER NUMBER :212222240081
### AIM: 
To write a python program to simulate the snake game using steering behaviors
### Algorithm:
1. Start the program
2. Import the necessary modules
3. Initiate the pygame engine and window
4. Specify the necessary parameter for background,snake and food
5. Create a function for seeking behavior towards the target
6.  Move the snake towards the target by move function
7.  Increase the size of snake by wrap around function
8.  create a food at location randomly
9.  In main, create a game loop, move the snake towards the food,check the collision and increase the size
10.  Update the display
11.  Stop the program
 ### Program:
```
import pygame
import random

# Initialize Pygame
pygame.init()

# Screen dimensions
width, height = 800, 600
screen = pygame.display.set_mode((width, height))
pygame.display.set_caption("Wandering Behavior")

# Colors
black = (0, 0, 0)
white = (255, 255, 255)

# Character properties
character_size = 20
x = width // 2
y = height // 2
velocity = 2

# Movement directions
direction_x = random.choice([-1, 1])
direction_y = random.choice([-1, 1])

# Time to change direction
change_direction_time = 1000  # in milliseconds
last_change_time = pygame.time.get_ticks()

# Game loop
running = True
while running:
   for event in pygame.event.get():
       if event.type == pygame.QUIT:
           running = False

   # Check if it's time to change direction
   current_time = pygame.time.get_ticks()
   if current_time - last_change_time > change_direction_time:
       direction_x = random.choice([-1, 1])
       direction_y = random.choice([-1, 1])
       last_change_time = current_time

   # Update character position
   x += direction_x * velocity
   y += direction_y * velocity

   # Boundaries check
   if x < 0 or x > width - character_size:
       direction_x *= -1
   if y < 0 or y > height - character_size:
       direction_y *= -1

   # Clear screen
   screen.fill(black)

   # Draw character
   pygame.draw.rect(screen, white, (x, y, character_size, character_size))

   # Update display
   pygame.display.flip()

   # Frame rate
   pygame.time.delay(30)

# Quit Pygame
pygame.quit()
```


### Output:
![image](https://github.com/user-attachments/assets/15901e92-24f3-4bd8-9c10-72e2a9cd0402)



### Result:
Thus the simple snake game was implemented.
