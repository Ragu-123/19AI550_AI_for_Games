# Ex.No: 11  Mini Project - AI enable fighter game 
### DATE:                                                                            
### REGISTER NUMBER : 212222240081
### AIM: 
The aim of the game is to provide an engaging combat experience where players use gestures and AI-driven strategies to outmaneuver and defeat opponents. It focuses on integrating intuitive controls and dynamic interactions for both single-player and multiplayer gameplay.
### Algorithm:
```
Game Initialization: Initialize the Pyxel window and define game resolution and title.
Menu Navigation: Use arrow keys and gestures to select between Test Game, Host Game, Join Game, and Quit.
Network Setup: Detect available network interfaces using netifaces for hosting or joining multiplayer games.
Gesture-Based Controls: Implement gesture recognition for player actions like attacks, movement, and blocking.
Game State Management: Use a GameState class to transition between MENU, PLAYING, and GAME_OVER.
AI Decision-Making: Use distance-based logic for AI actions: approach, retreat, attack, or idle.
Player Interaction: Enable keyboard and gesture inputs to control movement and initiate attacks.
Dynamic Rendering: Continuously update and draw game components (player, AI, menu) based on the current state.
Collision and Cooldowns: Manage attack cooldowns and detect player-AI or projectile collisions.
```

  
### Program:
```
import pyxel
import socket
import threading
import cv2
import mediapipe as mp
import numpy as np
import sys
import os
import netifaces
import time
import random

class GestureDetector:
    def __init__(self):
        self.mp_hands = mp.solutions.hands
        self.hands = self.mp_hands.Hands(
            static_image_mode=False,
            max_num_hands=1,
            min_detection_confidence=0.7
        )
        self.cap = cv2.VideoCapture(0)
        self.detected_gesture = None
        self.running = True
        self.thread = threading.Thread(target=self._process_frames)
        self.thread.start()
        self.last_gesture_time = time.time()
        self.gesture_cooldown = 0.2  # 200ms cooldown between gestures

    def _process_frames(self):
        while self.running:
            success, image = self.cap.read()
            if not success:
                continue

            image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
            results = self.hands.process(image)

            if results.multi_hand_landmarks:
                hand_landmarks = results.multi_hand_landmarks[0]
                
                current_time = time.time()
                if current_time - self.last_gesture_time >= self.gesture_cooldown:
                    # Check gestures in priority order
                    if self._is_block_gesture(hand_landmarks):
                        self.detected_gesture = "block"
                        self.last_gesture_time = current_time
                    elif self._is_power_attack_gesture(hand_landmarks):
                        self.detected_gesture = "power_attack"
                        self.last_gesture_time = current_time
                    elif self._is_punch_gesture(hand_landmarks):
                        self.detected_gesture = "punch"
                        self.last_gesture_time = current_time
                    elif self._is_dash_gesture(hand_landmarks):
                        self.detected_gesture = "dash"
                        self.last_gesture_time = current_time
                    else:
                        self.detected_gesture = None

            time.sleep(0.016)  # Approximately 60 FPS to match game performance

    def _is_punch_gesture(self, landmarks):
        finger_tips = [8, 12, 16, 20]  # Index, Middle, Ring, Pinky
        finger_mids = [6, 10, 14, 18]  # Middle points of fingers
        
        bent_fingers = sum(1 for tip, mid in zip(finger_tips, finger_mids)
                         if landmarks.landmark[tip].y > landmarks.landmark[mid].y)
        
        return bent_fingers >= 3

    def _is_block_gesture(self, landmarks):
        # Open palm facing forward (all fingers extended)
        finger_tips = [8, 12, 16, 20]  # Index, Middle, Ring, Pinky
        finger_mids = [6, 10, 14, 18]  # Middle points of fingers
        
        extended_fingers = sum(1 for tip, mid in zip(finger_tips, finger_mids)
                             if landmarks.landmark[tip].y < landmarks.landmark[mid].y)
        
        # Check if palm is vertical (y-coordinates of finger bases are similar)
        finger_bases = [5, 9, 13, 17]
        base_y_coords = [landmarks.landmark[i].y for i in finger_bases]
        vertical_palm = max(base_y_coords) - min(base_y_coords) < 0.1
        
        return extended_fingers >= 3 and vertical_palm

    def _is_power_attack_gesture(self, landmarks):
        # Thumbs up gesture
        thumb_tip = landmarks.landmark[4]
        thumb_ip = landmarks.landmark[3]
        
        # Other fingers should be folded
        finger_tips = [8, 12, 16, 20]
        finger_mids = [6, 10, 14, 18]
        
        fingers_folded = sum(1 for tip, mid in zip(finger_tips, finger_mids)
                           if landmarks.landmark[tip].y > landmarks.landmark[mid].y)
        
        thumb_up = thumb_tip.y < thumb_ip.y and fingers_folded >= 3
        return thumb_up

    def _is_dash_gesture(self, landmarks):
        # Horizontal swipe gesture (check if hand is moving horizontally)
        wrist = landmarks.landmark[0]
        fingertips = [landmarks.landmark[tip] for tip in [8, 12, 16, 20]]
        
        # Check if fingers are together and extended
        x_spread = max(tip.x for tip in fingertips) - min(tip.x for tip in fingertips)
        y_spread = max(tip.y for tip in fingertips) - min(tip.y for tip in fingertips)
        
        fingers_together = x_spread < 0.1
        fingers_extended = all(tip.y < wrist.y for tip in fingertips)
        
        return fingers_together and fingers_extended

    def get_gesture(self):
        gesture = self.detected_gesture
        self.detected_gesture = None
        return gesture

    def cleanup(self):
        self.running = False
        if hasattr(self, 'thread'):
            self.thread.join()
        self.cap.release()

class Player:
    def __init__(self, x, y, color, is_player_one=True):
        self.x = x
        self.y = y - 70
        self.vy = 0
        self.color = color
        self.is_jumping = False
        self.health = 100
        self.facing_right = True
        self.attacking = False
        self.blocking = False
        self.power_attacking = False
        self.dashing = False
        self.attack_frame = 0
        self.animation_frame = 0
        self.frame_counter = 0
        self.is_player_one = is_player_one
        self.state = "idle"
        self.dash_speed = 5
        self.dash_duration = 10
        self.dash_frame = 0
        self.power_attack_frame = 0
        
    def jump(self):
        if not self.is_jumping:
            self.is_jumping = True
            self.vy = -8

    def attack(self):
        if not self.attacking and not self.blocking:
            self.attacking = True
            self.attack_frame = 0

    def power_attack(self):
        if not self.power_attacking and not self.blocking:
            self.power_attacking = True
            self.power_attack_frame = 0

    def block(self):
        if not self.attacking and not self.power_attacking:
            self.blocking = True

    def stop_block(self):
        self.blocking = False

    def dash(self):
        if not self.dashing:
            self.dashing = True
            self.dash_frame = 0
        
    def update(self):
        self.frame_counter = (self.frame_counter + 1) % 8
        if self.frame_counter == 0:
            self.animation_frame = (self.animation_frame + 1) % 4

        if self.is_jumping:
            self.y += self.vy
            self.vy += 0.5
            self.state = "jump"
            if self.y > 80:
                self.y = 80
                self.is_jumping = False
                self.vy = 0
                self.state = "idle"
                
        if self.attacking:
            self.state = "attack"
            self.attack_frame += 1
            if self.attack_frame >= 10:
                self.attacking = False
                self.attack_frame = 0
                self.state = "idle"

        if self.power_attacking:
            self.state = "power_attack"
            self.power_attack_frame += 1
            if self.power_attack_frame >= 15:
                self.power_attacking = False
                self.power_attack_frame = 0
                self.state = "idle"

        if self.dashing:
            self.state = "dash"
            self.dash_frame += 1
            movement = self.dash_speed * (1 if self.facing_right else -1)
            self.x += movement
            if self.dash_frame >= self.dash_duration:
                self.dashing = False
                self.dash_frame = 0
                self.state = "idle"

    def draw(self):
        flip = 1 if self.facing_right else -1
        base_x = self.x if self.facing_right else self.x + 16
        
        # Draw shadow
        pyxel.rect(self.x + 2, 86, 12, 2, 5)
        
        if self.blocking:
            self._draw_block(base_x, flip)
        elif self.power_attacking:
            self._draw_power_attack(base_x, flip)
        elif self.state == "idle":
            self._draw_idle(base_x, flip)
        elif self.state == "walk":
            self._draw_walk(base_x, flip)
        elif self.state == "jump":
            self._draw_jump(base_x, flip)
        elif self.state == "attack":
            self._draw_attack(base_x, flip)
        elif self.state == "dash":
            self._draw_dash(base_x, flip)

    def _draw_block(self, base_x, flip):
        # Similar to idle but with arms raised in blocking position
        pyxel.rect(base_x - 6 * flip, self.y - 24, 12, 12, 7)
        pyxel.pset(base_x - 3 * flip, self.y - 18, 0)
        pyxel.rect(base_x - 6 * flip, self.y - 12, 12, 18, self.color)
        # Arms raised for blocking
        pyxel.rect(base_x - 9 * flip, self.y - 20, 3, 12, 7)
        pyxel.rect(base_x + 6 * flip, self.y - 20, 3, 12, 7)
        pyxel.rect(base_x - 6 * flip, self.y + 6, 4, 8, 7)
        pyxel.rect(base_x * flip, self.y + 6, 4, 8, 7)

    def _draw_power_attack(self, base_x, flip):
        self._draw_idle(base_x, flip)
        # Extended attacking arm with effect
        pyxel.rect(base_x + (3 if self.facing_right else -15), self.y - 8, 15, 4, 7)
        if self.power_attack_frame < 8:
            effect_x = base_x + (21 if self.facing_right else -33)
            pyxel.circ(effect_x, self.y - 8, 8 - self.power_attack_frame, 10)
            pyxel.circ(effect_x, self.y - 8, 6 - self.power_attack_frame, 8)

    def _draw_dash(self, base_x, flip):
        # Draw character leaning forward while dashing
        pyxel.rect(base_x - 6 * flip, self.y - 24, 12, 12, 7)
        pyxel.pset(base_x - 3 * flip, self.y - 18, 0)
        # Leaning body
        pyxel.rect(base_x - 8 * flip, self.y - 12, 14, 18, self.color)
        # Trailing effect
        for i in range(3):
            offset = (i + 1) * 4 * (-1 if self.facing_right else 1)
            pyxel.rect(base_x - 6 * flip + offset, self.y - 12, 12, 18, self.color - i - 1)

    # Keep existing _draw_idle, _draw_walk, _draw_jump, and _draw_attack methods...
    def _draw_idle(self, base_x, flip):
        pyxel.rect(base_x - 6 * flip, self.y - 24, 12, 12, 7)
        pyxel.pset(base_x - 3 * flip, self.y - 18, 0)
        pyxel.rect(base_x - 6 * flip, self.y - 12, 12, 18, self.color)
        arm_offset = 1 if self.animation_frame % 2 == 0 else 0
        pyxel.rect(base_x - 9 * flip, self.y - 12 + arm_offset, 3, 12, 7)
        leg_offset = 1 if self.animation_frame % 2 == 0 else 0
        pyxel.rect(base_x - 6 * flip, self.y + 6, 4, 8, 7)
        pyxel.rect(base_x * flip, self.y + 6 + leg_offset, 4, 8, 7)

    def _draw_walk(self, base_x, flip):
        self._draw_idle(base_x, flip)
        leg_offset = abs(2 - self.animation_frame) * 2
        pyxel.rect(base_x - 6 * flip, self.y + 6, 4, 8, 7)
        pyxel.rect(base_x * flip, self.y + 6 + leg_offset, 4, 8, 7)

    def _draw_jump(self, base_x, flip):
        self._draw_idle(base_x, flip)
        pyxel.rect(base_x - 6 * flip, self.y + 6, 4, 6, 7)
        pyxel.rect(base_x * flip, self.y + 6, 4, 6, 7)

    def _draw_attack(self, base_x, flip):
        self._draw_idle(base_x, flip)
        pyxel.rect(base_x + (3 if self.facing_right else -15), self.y - 8, 12, 3, 7)
        if self.attack_frame < 5:
            effect_x = base_x + (18 if self.facing_right else -30)
            pyxel.circ(effect_x, self.y - 8, 6 - self.attack_frame, 8 + self.attack_frame)

class TestGame:
    def __init__(self):
        self.player = Player(40, 150, 11)
        self.opponent = Player(120, 150, 8, False)
        self.ai = AIOpponent(self.player)
        self.game_state = GameState()
        self.background_color = 1
        self.floor_color = 3
        self.init_particles()
        self.gesture_detector = GestureDetector()

    def init_particles(self):
        self.particles = []

    def add_hit_particles(self, x, y, power=1):
        num_particles = 5 * power
        for _ in range(num_particles):
            angle = random.uniform(0, 6.28)
            speed = random.uniform(1, 3) * power
            self.particles.append({
                'x': x,
                'y': y,
                'dx': np.cos(angle) * speed,
                'dy': np.sin(angle) * speed,
                'life': 10 * power
            })

    def update_particles(self):
        for particle in self.particles[:]:
            particle['x'] += particle['dx']
            particle['y'] += particle['dy']
            particle['dy'] += 0.2
            particle['life'] -= 1
            if particle['life'] <= 0:
                self.particles.remove(particle)

    def update(self):
        # Check for gesture-based actions
        gesture = self.gesture_detector.get_gesture()
        if gesture:
            if gesture == "punch":
                self.player.attack()
            elif gesture == "block":
                self.player.block()
            elif gesture == "power_attack":
                self.player.power_attack()
            elif gesture == "dash":
                self.player.dash()

        # Regular controls (as fallback)
        if pyxel.btn(pyxel.KEY_LEFT) or pyxel.btn(pyxel.KEY_RIGHT):
            if not self.player.blocking and not self.player.dashing:
                self.player.state = "walk"
        else:
            if not self.player.blocking and not self.player.dashing:
                self.player.state = "idle"

        if pyxel.btn(pyxel.KEY_SPACE):
            self.player.jump()
        if pyxel.btn(pyxel.KEY_LEFT):
            if not self.player.blocking and not self.player.dashing:
                self.player.x -= 2
                self.player.facing_right = False
        if pyxel.btn(pyxel.KEY_RIGHT):
            if not self.player.blocking and not self.player.dashing:
                self.player.x += 2
                self.player.facing_right = True
        if pyxel.btnp(pyxel.KEY_X):  # Keyboard attack as fallback
            self.player.attack()
        if pyxel.btnp(pyxel.KEY_C):  # Keyboard block as fallback
            self.player.block()
        if not pyxel.btn(pyxel.KEY_C) and self.player.blocking:
            self.player.stop_block()

        self.ai.update(self.opponent)
        self.player.update()
        self.opponent.update()
        
        # Combat collision detection
        if abs(self.player.x - self.opponent.x) < 20:
            if self.player.attacking:
                if not self.opponent.blocking:
                    self.opponent.health -= 10
                    self.add_hit_particles(self.opponent.x, self.opponent.y - 8)
                else:
                    # Reduced damage when blocking
                    self.opponent.health -= 3
                    self.add_hit_particles(self.opponent.x, self.opponent.y - 8, 0.5)
                    
            if self.player.power_attacking:
                if not self.opponent.blocking:
                    self.opponent.health -= 20
                    self.add_hit_particles(self.opponent.x, self.opponent.y - 8, 2)
                else:
                    # Still significant damage when blocking power attack
                    self.opponent.health -= 10
                    self.add_hit_particles(self.opponent.x, self.opponent.y - 8, 1)
                    
            if self.opponent.attacking:
                if not self.player.blocking:
                    self.player.health -= 10
                    self.add_hit_particles(self.player.x, self.player.y - 8)
                else:
                    # Reduced damage when blocking
                    self.player.health -= 3
                    self.add_hit_particles(self.player.x, self.player.y - 8, 0.5)

        self.update_particles()
        
        # Game over condition
        if self.game_state.current_state == GameState.PLAYING:
            if self.player.health <= 0 or self.opponent.health <= 0:
                self.game_state.current_state = GameState.GAME_OVER
                self.game_state.winner = "AI Opponent" if self.player.health <= 0 else "You"
                self.game_state.final_score = (self.player.health, self.opponent.health)

    def draw(self):
        pyxel.cls(self.background_color)
        
        # Draw floor with details
        pyxel.rect(0, 88, 160, 120, self.floor_color)
        for i in range(0, 160, 20):
            pyxel.rect(i, 88, 2, 2, self.floor_color + 1)

        # Draw players
        self.player.draw()
        self.opponent.draw()
        
        # Draw particles
        for particle in self.particles:
            pyxel.pset(int(particle['x']), int(particle['y']), 
                      10 if particle['life'] > 5 else 9)
        
        # Draw health bars with fancy border
        def draw_health_bar(x, y, health, color):
            pyxel.rect(x-1, y-1, 52, 7, 1)  # Border
            pyxel.rect(x, y, 50, 5, 0)      # Background
            pyxel.rect(x, y, health // 2, 5, color)  # Health

        draw_health_bar(10, 10, self.player.health, 11)
        draw_health_bar(90, 10, self.opponent.health, 8)
        
        # Draw controls
        pyxel.text(5, 110, "SPACE:Jump X/Punch:Attack C:Block", 7)
        
        # Draw game over screen
        if self.game_state.current_state == GameState.GAME_OVER:
            pyxel.rectb(30, 40, 100, 40, 7)  # Border
            pyxel.rect(31, 41, 98, 38, 0)    # Background
            pyxel.text(45, 50, f"{self.game_state.winner} Won!", 7)
            pyxel.text(45, 60, f"Final Score: {self.game_state.final_score[0]}-{self.game_state.final_score[1]}", 7)
            pyxel.text(45, 70, "Press Q to quit", 7)

    def cleanup(self):
        if hasattr(self, 'gesture_detector'):
            self.gesture_detector.cleanup()

class NetworkUI:
    def __init__(self):
        self.state = "MENU"
        self.selected_option = 0
        self.menu_options = ["Test Game", "Host Game", "Join Game", "Quit"]
        self.ip_address = ""
        self.available_interfaces = self._get_network_interfaces()

    def _get_network_interfaces(self):
        interfaces = []
        try:
            for interface in netifaces.interfaces():
                try:
                    addrs = netifaces.ifaddresses(interface)
                    if netifaces.AF_INET in addrs:
                        for addr in addrs[netifaces.AF_INET]:
                            if 'addr' in addr:
                                interfaces.append((interface, addr['addr']))
                except Exception:
                    continue
        except Exception:
            interfaces.append(('localhost', '127.0.0.1'))
        return interfaces

    def update(self):
        if self.state == "MENU":
            if pyxel.btnp(pyxel.KEY_UP):
                self.selected_option = (self.selected_option - 1) % len(self.menu_options)
            if pyxel.btnp(pyxel.KEY_DOWN):
                self.selected_option = (self.selected_option + 1) % len(self.menu_options)
            
            if pyxel.btnp(pyxel.KEY_RETURN):
                if self.selected_option == 0:
                    return "TEST"
                elif self.selected_option == 1:
                    self.state = "HOST_SETUP"
                elif self.selected_option == 2:
                    self.state = "JOIN_SETUP"
                elif self.selected_option == 3:
                    return "QUIT"

        elif self.state == "HOST_SETUP":
            if pyxel.btnp(pyxel.KEY_ESCAPE):
                self.state = "MENU"
            return None

        elif self.state == "JOIN_SETUP":
            if pyxel.btnp(pyxel.KEY_ESCAPE):
                self.state = "MENU"
            return None

        return None

    def draw(self):
        pyxel.cls(0)
        
        if self.state == "MENU":
            pyxel.text(60, 20, "GESTURE COMBAT", pyxel.COLOR_WHITE)
            
            for i, option in enumerate(self.menu_options):
                color = pyxel.COLOR_YELLOW if i == self.selected_option else pyxel.COLOR_WHITE
                pyxel.text(60, 40 + i * 10, option, color)

        elif self.state == "HOST_SETUP":
            pyxel.text(40, 20, "HOST GAME", pyxel.COLOR_WHITE)
            pyxel.text(40, 40, "Available interfaces:", pyxel.COLOR_WHITE)
            
            for i, (interface, ip) in enumerate(self.available_interfaces):
                pyxel.text(40, 60 + i * 10, f"{interface}: {ip}", pyxel.COLOR_WHITE)
            
            pyxel.text(40, 100, "Press ESC to return", pyxel.COLOR_WHITE)

        elif self.state == "JOIN_SETUP":
            pyxel.text(40, 20, "JOIN GAME", pyxel.COLOR_WHITE)
            pyxel.text(40, 40, "Enter IP address:", pyxel.COLOR_WHITE)
            pyxel.text(40, 60, self.ip_address + "_", pyxel.COLOR_WHITE)
            pyxel.text(40, 100, "Press ESC to return", pyxel.COLOR_WHITE)

class GameState:
    MENU = "MENU"
    PLAYING = "PLAYING"
    GAME_OVER = "GAME_OVER"
    
    def __init__(self):
        self.current_state = self.PLAYING
        self.winner = None
        self.final_score = (0, 0)

class AIOpponent:
    def __init__(self, player):
        self.player = player
        self.attack_cooldown = 0
        self.decision_cooldown = 0
        self.current_action = None
        
    def update(self, opponent):
        if self.decision_cooldown > 0:
            self.decision_cooldown -= 1
        else:
            self.decision_cooldown = 30
            self.decide_action(opponent)
            
        if self.attack_cooldown > 0:
            self.attack_cooldown -= 1
            
        self.execute_action(opponent)
    
    def decide_action(self, opponent):
        distance = abs(self.player.x - opponent.x)
        
        if distance > 40:
            self.current_action = "approach"
        elif distance < 20:
            self.current_action = "retreat"
        elif self.attack_cooldown == 0:
            self.current_action = "attack"
        else:
            self.current_action = "idle"
    
    def execute_action(self, opponent):
        if self.current_action == "approach":
            if opponent.x < self.player.x:
                opponent.x += 1
                opponent.facing_right = False
            else:
                opponent.x -= 1
                opponent.facing_right = True
            opponent.state = "walk"
            
        elif self.current_action == "retreat":
            if opponent.x < self.player.x:
                opponent.x -= 1
                opponent.facing_right = True
            else:
                opponent.x += 1
                opponent.facing_right = False
            opponent.state = "walk"
            
        elif self.current_action == "attack" and self.attack_cooldown == 0:
            opponent.attack()
            self.attack_cooldown = 60
            
        else:
            opponent.state = "idle"

def main():
    pyxel.init(160, 120, title="Gesture Combat")
    
    net_ui = NetworkUI()
    current_game = None
    
    try:
        while True:
            if current_game is None:
                result = net_ui.update()
                net_ui.draw()
                
                if result == "TEST":
                    current_game = TestGame()
                elif result == "HOST":
                    pass
                elif result == "QUIT":
                    break
                elif isinstance(result, tuple) and result[0] == "JOIN":
                    pass
            else:
                current_game.update()
                current_game.draw()
                
                if pyxel.btnp(pyxel.KEY_Q):
                    if isinstance(current_game, TestGame):
                        current_game.cleanup()
                        break
                    else:
                        current_game = None
                    
            pyxel.flip()
    finally:
        if current_game and isinstance(current_game, TestGame):
            current_game.cleanup()

if __name__ == "__main__":
    try:
        main()
    except Exception as e:
        print(f"Fatal error: {e}")
        sys.exit(1)

'''

### Output:
![WhatsApp Image 2024-11-17 at 09 52 50_957a4df4](https://github.com/user-attachments/assets/8619b3e4-fdbb-4d37-87ce-0ef23f567cff)
![WhatsApp Image 2024-11-17 at 09 53 53_47416230](https://github.com/user-attachments/assets/144fe182-fc84-4dc7-a86f-fe95a412be58)
![WhatsApp Image 2024-11-17 at 09 55 12_75d7e201](https://github.com/user-attachments/assets/8e4d4c29-7554-4203-86ec-284cc9f2570f)
![WhatsApp Image 2024-11-17 at 09 55 39_61ce7c68](https://github.com/user-attachments/assets/8e6874a4-624a-4b13-a22f-f5fd0fd85fea)



### Result:
The 2D fighting game was successfully implemented, featuring gesture-based controls and AI-driven opponents. Players can interact with AI enemies through dynamic animations, health tracking, and scoring mechanisms. The game concludes with a "Game Over" screen showcasing the final score, delivering an engaging and polished gameplay experience.
