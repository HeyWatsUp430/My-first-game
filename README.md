# My-first-game
This is my first game :) 

#Import the modules needed
import os
import sys
import time
import random
import pygame

#Initializing the font module
pygame.font.init()

#Setting up the window
W, H = 600, 600
window = pygame.display.set_mode((W, H))
pygame.display.set_caption('Space Explorer')

#Setting up the fonts needed for the game
font1 = pygame.font.SysFont('Arial', 50)
font2 = pygame.font.SysFont('comicsans', 50)
font3 = pygame.font.SysFont('Bahnschrift', 30)

#---------------------------------------------------
#Setting up the images of the sprites for the game
#---------------------------------------------------

#The Background
BG = pygame.transform.scale(pygame.image.load(os.path.join('pic/background4.png')),(W,H))

# The sprites for the enemies and their lasers
ENEMY_SHIP_ONE = pygame.transform.scale(pygame.image.load(os.path.join('pic/enemyship3.png')),(70,50))
ENEMY_SHIP_TWO = pygame.transform.scale(pygame.image.load(os.path.join('pic/enemyship2.png')),(70,50))
ENEMY_SHIP_THREE = pygame.transform.scale(pygame.image.load(os.path.join('pic/enemyship4.png')),(70,50))
ENEMY_SHIP_FOUR = pygame.transform.scale(pygame.image.load(os.path.join('pic/enemyship1.png')),(70,50))

ENEMY_LASER_ONE = pygame.transform.scale(pygame.image.load(os.path.join('pic/enemy_laser1.png')),(50,90))
ENEMY_LASER_TWO = pygame.transform.scale(pygame.image.load(os.path.join('pic/enemy_laser2.png')),(50,90))
ENEMY_LASER_THREE = pygame.transform.scale(pygame.image.load(os.path.join('pic/enemy_laser3.png')),(50,90))
ENEMY_LASER_FOUR = pygame.transform.scale(pygame.image.load(os.path.join('pic/enemy_laser4.png')),(50,90))

#The sprites for the player and laser
PLAYER_SPACE_SHIP = pygame.transform.scale(pygame.image.load(os.path.join('pic/player_spaceship.png')), (150,150))
PLAYER_LASER = pygame.transform.scale(pygame.image.load(os.path.join('pic/laser_player.png')), (90,100))

class Laser:
    '''
    Creating the laser class
    '''
    def __init__(self, x, y, img):
        self.x = x
        self.y = y
        self.img = img
        self.mask = pygame.mask.from_surface(self.img)

    def draw_laser(self):
        '''
        Draws the laser in the screen
        '''
        window.blit(self.img, (self.x, self.y))

    def collision(self, obj):
        '''
        Returns True if obj collides
        '''
        return collide(self, obj)        

    def offscreen(self):
        '''
        Checks if the laser is of the screen
        '''
        return not(self.y <= H and self.y >= 0)

    def move_laser(self, vel):
        '''
        Moves the laser
        '''
        self.y += vel
    
class Ship:
    '''
    Creating an abstract class 
    for the player ship class and the enemy ship class
    '''

    def __init__(self, x, y, health):
        self.x = x
        self.y = y
        self.image = None
        self.laser_image = None
        self.laser_list = []
        self.health = health
        self.max_health = 100
        self.cooldown = 6
        self.cool_down_counter = 0

    def cool_down(self):
        '''
        Cools down the laser after shooting
        '''
        if self.cool_down_counter == self.cooldown:
            self.cool_down_counter = 0
        elif self.cool_down_counter < self.cooldown:
            self.cool_down_counter += 1

    def shoot(self):
        '''
        Shoots the laser
        '''
        if self.cool_down_counter == 0:
            laser = Laser(self.x + 30, self.y, self.laser_image)
            self.laser_list.append(laser)
            self.cool_down_counter = 1

    def move_lasers(self, vel, objs):
        '''
        Moves the lasers with the vel param
        '''
        self.cool_down()
        for laser in self.laser_list:
            laser.move_laser(vel)
            if laser.offscreen():
                self.laser_list.remove(laser)
            elif laser.collision(objs):
                objs.health -= 10  
                self.laser_list.remove(laser) 
            
    def draw(self):
        '''
        Draws the image of the ship and the laser
        '''
        window.blit(self.image, (self.x, self.y))
        for laser in self.laser_list:
            laser.draw_laser()

    def width(self):
        '''
        Returns the width of the ship
        '''
        return self.image.get_width()

    def height(self):
        '''
        Returns the height of the ship
        '''
        return self.image.get_height()

class Player(Ship):
    '''
    Creating the player class
    '''
    def __init__(self, x, y, health = 100):
        super().__init__(x, y, health = 100)
        self.image = PLAYER_SPACE_SHIP
        self.laser_image = PLAYER_LASER
        self.points = 0
        self.mask = pygame.mask.from_surface(self.image)

    def move_lasers(self, vel, objs):
        '''
        Overwriting the move_lasers method for the player class
        because the original method only lessens the health of the player
        so if laser collides with enemy obj, enemy obj gets removed.
        '''
        self.cool_down()
        for laser in self.laser_list:
            laser.move_laser(vel)
            if laser.offscreen():
                self.laser_list.remove(laser)
            else:
                for obj in objs:
                    try:
                        if laser.collision(obj):
                            self.points += 10
                            objs.remove(obj)
                            self.laser_list.remove(laser)
                    except ValueError:
                        continue

    def draw(self):
        '''
        Overwriting draw method to draw the healthbar at the same timeS
        '''
        super().draw()
        self.healthbar()
    
    def healthbar(self):
        '''
        Drawing the health bar
        '''
        pygame.draw.rect(window, (255,0,0), (0,590, 600, 30))
        pygame.draw.rect(window, (0,255,0), (0,590, 600 * (self.health/self.max_health), 30))
            
#Creating the Enemy class
class Enemy(Ship):
    '''
    Creating the Enemy class
    '''

    images = {
            'ship1': (ENEMY_SHIP_ONE, ENEMY_LASER_ONE),
            'ship2': (ENEMY_SHIP_TWO, ENEMY_LASER_TWO),
            'ship3': (ENEMY_SHIP_THREE, ENEMY_LASER_THREE),
            'ship4': (ENEMY_SHIP_FOUR, ENEMY_LASER_FOUR),
            }

    def __init__(self, x, y, color):
        super().__init__(x, y, health=100)
        self.image, self.laser_image = self.images[color]
        self.mask = pygame.mask.from_surface(self.image)

    def move(self, vel):
        '''
        Moves the enemy ship
        '''
        self.y += vel

def collide(obj1, obj2):
    '''
    This checks if obj1 has collided with obj2
    '''
    x = obj2.x - obj1.x
    y = obj2.y - obj1.y
    return obj1.mask.overlap(obj2.mask, (x, y)) != None

def draw(level, score, lives, enemies):
    '''
    This function places the texts, sprites of the game
    '''

    for enemy in enemies:
        enemy.draw()

    level_label = font3.render(f'level: {level}',1,(255,0,0))
    lives_label = font3.render(f'Lives: {lives}',1,(0,255,0))
    scoretext = font3.render('Score',1,(200,0,255))
    scorevalue = font3.render(f'{score}',1,(200,0,255))

    window.blit(lives_label, (10,10))
    window.blit(level_label, (W - level_label.get_width() - 10, 10))
    window.blit(scoretext, (W/2 - 40, 10))
    window.blit(scorevalue, (W/2 - scorevalue.get_width()/2, 50))

    pygame.display.update()

def maingame():
    '''
    This is where the main function of the game comes in
    '''
    clock = pygame.time.Clock()
    FPS = 60
    
    level = 0
    lives = 3

    player_health = 100
    player = Player(223, 468, player_health)
    player_vel = 10

    enemy_obj = []
    enemy_vel = 3
    wave = 0

    laser_vel = 30
    enemy_shot = 100

    run = True

    def lose(score):
        '''
        This function tells us if it is game over 
        '''
        window.fill((0,0,0))

        lose_font = font2.render('Game over :(',1, (255,0,0))
        score_font = font2.render('Your score',1,(0,255,0))
        wait_font = font2.render('Wait for the screen to go back',1,(0,0,255))
        wait_font2 = font2.render('to go back to the main menu',1,(0,0,255))
        score_value = font2.render(f'{score}',1,(0,255,0))

        window.blit(lose_font, (W/2 - lose_font.get_width()/2, 100))
        window.blit(score_font, (W/2 - score_font.get_width()/2, 200))
        window.blit(score_value, (W/2 - score_value.get_width()/2, 300))
        window.blit(wait_font, (W/2 - wait_font.get_width()/2, 400))
        window.blit(wait_font2, (W/2 - wait_font2.get_width()/2, 450))

        pygame.display.update()
        
    while run:
        clock.tick(FPS)
        window.blit(BG, (0,0))
        player.draw()
        draw(level,player.points,lives, enemy_obj)

        if len(enemy_obj) == 0:
            enemy_shot -= 5
            wave += 5
            level += 1
            player_vel += 2
            
            for _ in range(wave):
                enemy = Enemy(random.randrange(50,W - 100), random.randrange(-1500, -100), random.choice(['ship1','ship2','ship3', 'ship4']))
                enemy_obj.append(enemy) 
    
        if lives == 0:
            lose(player.points)
            time.sleep(2)
            main_menu()

        if player.health == 0:
            lives -= 1
            player.health = 100

        for event in pygame.event.get():
            if event.type == pygame.QUIT: 
                sys.exit()

        keys = pygame.key.get_pressed()

        if keys[pygame.K_a] and player.x - player_vel + 39 > 0:
            player.x -= player_vel
        if keys[pygame.K_d] and player.x + player_vel + player.width() - 38 < W:
            player.x += player_vel
        if keys[pygame.K_w] and player.y - player_vel  + 40 > 0:
            player.y -= player_vel
        if keys[pygame.K_s] and player.y + player_vel + player.height() - 30 < H:
            player.y += player_vel
        if keys[pygame.K_SPACE]:
            player.shoot()
        
        player.move_lasers(-laser_vel, enemy_obj)

        for enemy in enemy_obj[:]:
            enemy.move(enemy_vel)
            enemy.move_lasers(laser_vel, player)

            if enemy.y + enemy.height() > H:
                lives -= 1
                enemy_obj.remove(enemy)

            if collide(enemy, player):
                player.health -= 20
                enemy_obj.remove(enemy)
            
            if random.randint(1,enemy_shot) == 1:
                enemy.shoot()

def main_menu():
    '''
    This function places the main menu for us
    '''
    title = font1.render('Space Explorer',10,(0,255,0))
    title2 = font2.render('Press the mouse button to play',1,(255,0,0))

    mission_color = (250,50,255)

    story = font3.render('Story',1,(240,190,125))
    mission1 = font3.render('You are a space explorer, eager to explore',1,mission_color)
    mission2 = font3.render('the vast regions of space. However as you',1,mission_color)
    mission3 = font3.render('progress your journey, aliens interfere and',1,mission_color)
    mission4 = font3.render('start attacking your ship, you must survive.',1,mission_color)

    window.blit(BG, (0,0))
    window.blit(title, (W/2 - title.get_width() + 150, 100))
    window.blit(title2, (W/2 - title.get_width() + 30 , 200))
    window.blit(story, (W/2 - story.get_width()/2, 300))
    window.blit(mission1, (10,350))
    window.blit(mission2, (10,400))
    window.blit(mission3, (10,450))
    window.blit(mission4, (10,500))
    pygame.display.update()

    running = True
    while running:

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                sys.exit()
            if event.type == pygame.MOUSEBUTTONDOWN:
                maingame()

    pygame.display.update()

main_menu()
