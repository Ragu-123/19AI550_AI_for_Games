# Ex.No: 5  Implementation of Jumping behavior 
### DATE:13-09-2024                                                                           
### REGISTER NUMBER : 212222240081
### AIM: 
To write a python program to simulate Jumbing behavior. 
### Algorithm:
1. Start the program
2. Import the necessary modules
3. Initiate the pygame engine and window
4. Specify the necessary parameter for player height,depth,gravity,jump power. 
5. Create a game loop to simulate the continuous behavior.
6. If Quit button is pressed then quit the pygame window.
7. Move the player left when left button is pressed
8. Move the player right when right button is pressed
9. If space bar is pressed then enable the jump by increasing y axis value.
10. land the player and display the player at every timestep
11.  Stop the program
 ### Program:
```
import pygame
from pygame.locals import *
from sys import exit

# Path to your image file
sprite_image_filename = r"C:\Users\manoj\Downloads\ball.jpeg"

pygame.init()

# Set up the display window
screen = pygame.display.set_mode((840, 680), 0, 32)

# Load the sprite image
sprite = pygame.image.load(sprite_image_filename)

# Set up the clock for controlling frame rate
clock = pygame.time.Clock()

# The initial position of our sprite
x = 100  # X coordinate (constant)
y = 480  # Y coordinate (starting position)

# Variables for jumping
is_jumping = False
jump_height = 150  # How high the sprite will jump
jump_speed = 300   # Speed of the jump (in pixels per second)
initial_y = y      # Store the original y position

# Main game loop
while True:
   for event in pygame.event.get():
       if event.type == QUIT:
           pygame.quit()
           exit()
       if event.type == MOUSEBUTTONDOWN:
           if not is_jumping:  # Start jump on mouse click
               is_jumping = True

   # Clear the screen
   screen.fill((0, 0, 0))

   if is_jumping:
       # Calculate time passed
       time_passed = clock.tick(30)
       time_passed_seconds = time_passed / 1000.0
       
       # Update y position to simulate jump
       y -= jump_speed * time_passed_seconds
       
       # Check if the sprite has reached the peak of the jump
       if y <= initial_y - jump_height:
           is_jumping = False
   
   else:
       # If not jumping, bring the sprite back down to the initial position
       if y < initial_y:
           time_passed = clock.tick(30)
           time_passed_seconds = time_passed / 1000.0
           y += jump_speed * time_passed_seconds
           
           # Ensure it doesn't go past the original position
           if y > initial_y:
               y = initial_y

   # Draw the sprite at the updated position
   screen.blit(sprite, (x, y))
   
   # Update the display
   pygame.display.update()

   # Control the frame rate
   clock.tick(30)
```



### Output:
![image](https://github.com/user-attachments/assets/dae0afb9-5cb8-4e6f-ae00-60b5753ea9b8)
![image](https://github.com/user-attachments/assets/543439d4-d4b4-4460-b3d9-8eaa8c89c975)



### Result:
Thus the simple jumping behavior  was implemented.
