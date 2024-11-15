# Ex.No: 11  Mini Project - AI enable fighter game 
### DATE:                                                                            
### REGISTER NUMBER : 212222240081
### AIM: 
To develop a 2D fighting game using Python and Pygame, where a player character battles AI-controlled enemies with animations, health management, scoring, and respawn mechanics. The game will feature smooth character movement, attack animations, and an engaging gameplay experience with a final score display on game over.
### Algorithm:
**1.Game Setup:**

  Initialize Pygame and set up the game window.

  Load background, spritesheet, and sound effects.

  Define colors and fonts for health bars and score.

**2. Define Fighter Class:**
   
 Initialize position, health, and score.
 
 Load and organize sprite animations for actions (idle, move, attack).
 
 Implement movement, boundary checks, and a jump mechanism.
 
 Handle attacks, health reduction on hit, and display health bars.
  
**3. Main Game Loop:**
   
  Initialize player and enemies.
  
Loop while player’s health > 0:

  Handle player input for movement and attacks.
 
   Move enemies towards the player and trigger attacks if in range.
 
   Check for collisions, apply damage, and play sound effects.
 
   Respawn defeated enemies and increase player score.
  
**4. Display Info & Game Over:**

  Display player’s score and health bars.
  
  If player health reaches zero, show “Game Over” and final score, then close the game.
  
### Program:
```
import pygame
from pygame import mixer
from fighter import Fighter

mixer.init()
pygame.init()

#Create game window
SCREEN_WIDTH = 1000
SCREEN_HEIGHT = 600

screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Brawler")

#Set framerate
clock = pygame.time.Clock()
FPS = 60

#Define colours
RED = (255, 0, 0)
YELLOW = (255, 255, 0)
WHITE = (255, 255, 255)

#Define game variables
intro_count = 3
last_count_update = pygame.time.get_ticks()
score = [0, 0] #Player scores [P1, P2]
round_over = False
ROUND_OVER_COOLDOWN = 2000

#Define fighter variables
HERO1_SIZE = 200.25
HERO1_SCALE = 3.4
HERO1_OFFSET = [87, 70]
HERO1_DATA = [HERO1_SIZE, HERO1_SCALE, HERO1_OFFSET]
HERO2_SIZE = 200.25
HERO2_SCALE = 3.2
HERO2_OFFSET = [84, 72]
HERO2_DATA = [HERO2_SIZE, HERO2_SCALE, HERO2_OFFSET]

#Load music and sounds
pygame.mixer.music.load("assets/audio/music.mp3")
pygame.mixer.music.set_volume(0.1)
pygame.mixer.music.play(-1, 0.0, 5000)
sword_fx = pygame.mixer.Sound("assets/audio/sword.mp3")
sword_fx.set_volume(0.5)
hit_fx = pygame.mixer.Sound("assets/audio/hit.mp3")
hit_fx.set_volume(0.5)

#Load background
bg_image = pygame.image.load("assets/images/background/background.png").convert_alpha()

#Load spritesheets
hero1_sheet = pygame.image.load("assets/images/characters/hero1/Sprites/hero1.png").convert_alpha()
hero2_sheet = pygame.image.load("assets/images/characters/hero2/Sprites/hero2.png").convert_alpha()

#Define number of frames of each animation
HERO1_ANIMATION_FRAMES = [8, 8, 2, 6, 6, 4, 6]
HERO2_ANIMATION_FRAMES = [4, 8, 2, 4, 4, 3, 7]

#Define font
count_font = pygame.font.Font("assets/fonts/turok.ttf", 80)
score_font = pygame.font.Font("assets/fonts/turok.ttf", 30)
victory_font = pygame.font.Font("assets/fonts/turok.ttf", 70)

#Function to draw text
def draw_text(text, font, text_col, x, y):
    img = font.render(text, True, text_col) #Convert text to image
    screen.blit(img, (x, y))

#Function to draw background
def draw_bg():
    #Scale original background image to game window size
    scaled_bg = pygame.transform.scale(bg_image, (SCREEN_WIDTH, SCREEN_HEIGHT))
    screen.blit(scaled_bg, (0, 0))

#Function for drawing fighter health bars
def draw_health_bar(health, x, y):
    ratio = health / 100
    pygame.draw.rect(screen, WHITE, (x - 2, y - 2, 404, 34))
    pygame.draw.rect(screen, RED, (x, y, 400, 30))
    pygame.draw.rect(screen, YELLOW, (x, y, 400 * ratio, 30)) #Adjust health bar based on ratio of health

#Create two instances of fighters
fighter_1 = Fighter(1, 200, 370, False, HERO1_DATA, hero1_sheet, HERO1_ANIMATION_FRAMES, sword_fx, hit_fx)
fighter_2 = Fighter(2, 700, 370, True, HERO2_DATA, hero2_sheet, HERO2_ANIMATION_FRAMES, sword_fx, hit_fx)

#Game loop
run = True
while run:

    clock.tick(FPS)

    #Draw background
    draw_bg()

    #Show player stats
    draw_health_bar(fighter_1.health, 20, 20)
    draw_health_bar(fighter_2.health, 580, 20)
    draw_text("P1: " + str(score[0]) + " rounds won", score_font, RED, 20, 60)
    draw_text("P2: " + str(score[1]) + " rounds won", score_font, RED, 580, 60)

    #Update countdown
    if intro_count <= 0:
        #Move fighters
        fighter_1.move(SCREEN_WIDTH, SCREEN_HEIGHT, screen, fighter_2, round_over)
        fighter_2.move(SCREEN_WIDTH, SCREEN_HEIGHT, screen, fighter_1, round_over)
    else:
        #Display countdown timer
        draw_text(str(intro_count), count_font, RED, SCREEN_WIDTH / 2, SCREEN_HEIGHT / 4)
        
        #Update countdown timer
        if (pygame.time.get_ticks() - last_count_update) >= 1000:
            intro_count -= 1
            last_count_update = pygame.time.get_ticks()
            print(intro_count)

    #Update fighters
    fighter_1.update()
    fighter_2.update()

    #Draw fighters
    fighter_1.draw(screen)
    fighter_2.draw(screen)

    #Check for player defeat
    if round_over == False:
        if fighter_1.alive == False:
            score[1] += 1
            round_over = True
            round_over_time = pygame.time.get_ticks()
        elif fighter_2.alive == False:
            score[0] += 1
            round_over = True
            round_over_time = pygame.time.get_ticks()
    else:
        #Display win text
        draw_text("Player {} Wins!".format(1 if fighter_1.alive else 2), victory_font, RED, SCREEN_WIDTH / 3.5, SCREEN_HEIGHT / 5)
        #Additional logic for handling end of the round
        if pygame.time.get_ticks() - round_over_time > ROUND_OVER_COOLDOWN:
            round_over = False
            intro_count = 3
            #Recreate instances of fighters
            fighter_1 = Fighter(1, 200, 370, False, HERO1_DATA, hero1_sheet, HERO1_ANIMATION_FRAMES, sword_fx, hit_fx)
            fighter_2 = Fighter(2, 700, 370, True, HERO2_DATA, hero2_sheet, HERO2_ANIMATION_FRAMES, sword_fx, hit_fx)


    #Event handler
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            run = False


    #Update display
    pygame.display.update()

#Exit pygame
pygame.quit()

import pygame

class Fighter():
    #Constructor
    def __init__(self, player, x, y, flip, data, sprite_sheet, animation_frames, attack_sound, hit_sound):
        #Player hitbox
        self.player = player
        self.size = data[0]
        self.image_scale = data[1]
        self.offset = data[2]
        self.custom_offset = [87, 76]
        self.hit_offset = [85, 81.5]
        self.death_offset = [85, 83]
        self.flip = flip

        #Player animation
        self.animation_list = self.load_images(sprite_sheet, animation_frames)
        self.action = 0 #0:idle #1:run #2:jump #3:attack1 #4:attack2 #5:hit #6:death
        self.frame_index = 0
        self.image = self.animation_list[self.action][self.frame_index] #Assign first frame of fighter's current action animation 
        self.update_time = pygame.time.get_ticks()
        self.rect = pygame.Rect((x, y, 80, 180))
        self.vel_y = 0

        #Player status and actions
        self.running = False
        self.jump = False
        self.attacking = False
        self.attack_type = 0
        self.attack_cooldown = 0
        self.last_attack_time = 0
        self.attack_sound = attack_sound
        self.hit = False
        self.hit_sound = hit_sound
        self.health = 100
        self.alive = True
    
    def load_images(self, sprite_sheet, animation_frames):
        #Extract images from spritesheet
        animation_list = []
        #y = 0
        for y, animation in enumerate(animation_frames): #Track each frame
            temp_img_list = []
            #Create temp list of frames from each row of spritesheet to add to main list
            for x in range(animation):
                temp_img = sprite_sheet.subsurface(x * self.size, y * self.size, self.size, self.size)
                pygame.transform.scale(temp_img, (self.size * self.image_scale, self.size * self.image_scale))
                temp_img_list.append(pygame.transform.scale(temp_img, (self.size * self.image_scale, self.size * self.image_scale)))
            #y += 1
            animation_list.append(temp_img_list)
        #print(animation_list)
        return animation_list

    #Method to move fighters
    def move(self, screen_width, screen_height, surface, target, round_over):
        SPEED = 10
        GRAVITY = 2
        FLOOR = 50
        dx = 0
        dy = 0
        self.running = False
        self.attack_type = 0

        #Get keypress
        key = pygame.key.get_pressed() #Register key pressed into key variable

        #Can only perform other actions if not currently attacking
        if self.attacking == False and self.alive == True and round_over == False:
            #Check player 1 controls
            if self.player == 1:
                #Movement keys
                #Move left
                if key[pygame.K_a]: 
                    dx = -SPEED
                    self.running = True
                #Move right
                if key[pygame.K_d]:
                    dx = SPEED
                    self.running = True
                #Jump
                if key[pygame.K_w] and self.jump == False: #If not currently jumping
                    self.vel_y = -30
                    self.jump = True
                #Attack
                if key[pygame.K_r] or key[pygame.K_t]:
                    self.attack(target)
                    #Determine which attack type was used
                    if key[pygame.K_r]:
                        self.attack_type = 1
                    if key[pygame.K_t]:
                        self.attack_type = 2

            #Check player 2 controls
            if self.player == 2:
                #Movement keys
                #Move left
                if key[pygame.K_LEFT]: 
                    dx = -SPEED
                    self.running = True
                #Move right
                if key[pygame.K_RIGHT]:
                    dx = SPEED
                    self.running = True
                #Jump
                if key[pygame.K_UP] and self.jump == False: #If not currently jumping
                    self.vel_y = -30
                    self.jump = True
                #Attack
                if key[pygame.K_KP1] or key[pygame.K_KP2]:
                    self.attack(target)
                    #Determine which attack type was used
                    if key[pygame.K_KP1]:
                        self.attack_type = 1
                    if key[pygame.K_KP2]:
                        self.attack_type = 2

        #Apply gravity
        self.vel_y += GRAVITY
        dy += self.vel_y

        #Ensure players stays on screen
        if self.rect.left + dx < 0: #If player is going off left side of screen (x position is negative)
            dx = 0 - self.rect.left

        if self.rect.right + dx > screen_width: #If player is going off right side of screen
            dx = screen_width - self.rect.right

        if self.rect.bottom + dy > screen_height - FLOOR: 
            self.vel_y = 0
            self.jump = False #Reset player status to not jumping
            dy = screen_height - FLOOR - self.rect.bottom
        
        #Ensure players face each other
        if target.rect.centerx > self.rect.centerx:
            self.flip = False
        else:
            self.flip = True
        
        #Apply attack cooldown
        if self.attack_cooldown > 0:
            self.attack_cooldown -= 1

        #Update player position
        self.rect.x += dx
        self.rect.y += dy

    #Handle animation updates
    def update(self):
        #Check what action the player is performing
        if self.health <= 0:
            self.health = 0
            self.alive = False
            self.update_action(6) #6:death
        elif self.hit == True:
            self.update_action(5) #5:hit
        elif self.attacking == True: 
            if self.attack_type == 1:
                self.update_action(3) #3:attack1
            elif self.attack_type == 2:
                self.update_action(4) #4:attack2
        elif self.jump == True:
            self.update_action(2) #2:jump
        elif self.running == True:
            self.update_action(1) #1:run
        else:
            self.update_action(0) #0:idle     

        animation_cooldown = 45

        #Update image
        self.image = self.animation_list[self.action][self.frame_index]

        #Check if enough time has passed since last update
        if pygame.time.get_ticks() - self.update_time > animation_cooldown:
            self.frame_index += 1
            self.update_time = pygame.time.get_ticks()

        #Check if animation has finished
        if self.frame_index >= len(self.animation_list[self.action]): #If next animation index is out of range

            #If player is dead, end animation
            if self.alive == False:
                self.frame_index = len(self.animation_list[self.action]) - 1
            else:
                self.frame_index = 0

                #Check if an attack was performed
                if self.action == 3 or self.action == 4:
                    self.attacking = False
                    self.attack_cooldown = 20

                #Check if damage was taken
                if self.action == 5:
                    self.hit = False
                    #If the player was in the middle of an attack, stop attack animation
                    self.attacking = False
                    self.attack_cooldown = 20

    #Method to attack
    def attack(self, target):
        #Add a delay before allowing another attack
        ATTACK_DELAY = 300

        if self.attack_cooldown == 0 and pygame.time.get_ticks() - self.last_attack_time > ATTACK_DELAY:
            self.attacking = True
            self.attack_sound.play()
            
            #Create attack hitbox facing target, from center of player
            attack_width = 3 * self.rect.width
            attack_height = self.rect.height
            if self.flip:
                attacking_rect = pygame.Rect(self.rect.left - attack_width, self.rect.y, attack_width, attack_height)
            else:
                attacking_rect = pygame.Rect(self.rect.right, self.rect.y, attack_width, attack_height)
            
            #Check for collision with target
            if attacking_rect.colliderect(target.rect):
                target.health -= 34
                target.hit = True
                self.hit_sound.play()

            #Set the attack cooldown
            self.attack_cooldown = 20
            self.last_attack_time = pygame.time.get_ticks()
            #pygame.draw.rect(surface, (0, 255, 0), attacking_rect)



    def update_action(self, new_action):
        #Check if the new action is different to previous one
        if new_action != self.action:
            self.action = new_action

            #Update animation settings
            self.frame_index = 0
            self.update_time = pygame.time.get_ticks()

    #Method to draw fighters    
    def draw(self, surface):
        img = pygame.transform.flip(self.image, self.flip, False)
        #pygame.draw.rect(surface, (255, 0, 0), self.rect)

        # Check the current action and apply the appropriate offset
        if self.action == 5:  # Hit animation
            surface.blit(img, (self.rect.x - (self.hit_offset[0] * self.image_scale), self.rect.y - (self.hit_offset[1] * self.image_scale)))
        elif self.action == 6:  # Death animation
            surface.blit(img, (self.rect.x - (self.death_offset[0] * self.image_scale), self.rect.y - (self.death_offset[1] * self.image_scale)))
        elif self.action not in [0, 1, 2]:  # Custom offset for animations other than idle, run, and jump
            surface.blit(img, (self.rect.x - (self.custom_offset[0] * self.image_scale), self.rect.y - (self.custom_offset[1] * self.image_scale)))
        else:
            # Apply regular offset for idle, run, and jump actions
            surface.blit(img, (self.rect.x - (self.offset[0] * self.image_scale), self.rect.y - (self.offset[1] * self.image_scale)))



```
### Output:
![WhatsApp Image 2024-11-12 at 13 20 14_35cffa6c](https://github.com/user-attachments/assets/9d7e90bf-99be-4d1b-8147-ac8751b8dda4)
![WhatsApp Image 2024-11-12 at 13 20 15_64010c38](https://github.com/user-attachments/assets/d4b75f05-96d0-4f55-a592-8cf93c8392dc)
![WhatsApp Image 2024-11-12 at 13 21 21_e08494d7](https://github.com/user-attachments/assets/9f6e097c-2b4e-4d73-8d90-3aed6fe5e186)



### Result:

The 2D fighting game was successfully developed using Python and Pygame. The game allows the player character to engage AI-controlled enemies with animated actions, health bars, scoring, and respawn mechanics. The game ends with a "Game Over" screen displaying the final score, providing an interactive and visually appealing gameplay experience.
