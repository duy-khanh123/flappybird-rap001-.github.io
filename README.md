# flappybird-rap001-.github.io
# PROJECT PYTHON :FLAPPY BIRD

import pygame
from pygame.locals import *
import random

pygame.init()

clock = pygame.time.Clock()
fps = 60

screen_width = 864
screen_height = 756

screen = pygame.display.set_mode((screen_width, screen_height))
pygame.display.set_caption('Flappy Bird')

#define font
font = pygame.font.SysFont('Bauhaus 93', 60)
font2 = pygame.font.SysFont('Bauhaus 93', 30)

#define colours
white = (255, 255, 255)

#define game variables
ground_scroll = 0
scroll_speed = 4
flying = False
game_over = False
pipe_gap = 150
pipe_frequency = 1500 #milliseconds
last_pipe = pygame.time.get_ticks() - pipe_frequency
score = 0
pass_pipe = False
food_appear = False
waitting_food = 0
max_hp = 5

#load max score
file = open('max_score.txt', 'r')
max_score = int(file.readline())
file.close()

#load images
bg = pygame.image.load('img/bg.png')
ground_img = pygame.image.load('img/ground.png')
button_img = pygame.image.load('img/restart.png')


#function for outputting text onto the screen
def draw_text(text, font, text_col, x, y):
	img = font.render(text, True, text_col)
	screen.blit(img, (x, y))

def reset_game():
	pipe_group.empty()
	flappy.rect.x = 80
	flappy.rect.y = int(screen_height / 2)
	flappy.hp = 1
	hp_bar.slg = 1
	food_group.empty()
	return score

def reset_game2():
	global flying
	pipe_group.empty()
	flappy.rect.x = 80
	flappy.rect.y = int(screen_height / 2)
	flying = False
	food_group.empty()

class Bird(pygame.sprite.Sprite):

	def __init__(self, x, y):
		pygame.sprite.Sprite.__init__(self)
		self.images = []
		self.index = 0
		self.counter = 0
		for num in range (1, 4):
			img = pygame.image.load(f"img/bird{num}.png")
			self.images.append(img)
		self.image = self.images[self.index]
		self.rect = self.image.get_rect()
		self.rect.center = [x, y]
		self.vel = 0
		self.clicked = False
		self.hp = 1
		

	def update(self):

		if flying == True:
			#apply gravity
			self.vel += 0.5
			if self.vel > 8:
				self.vel = 8
			if self.rect.bottom < 768:
				self.rect.y += int(self.vel)

		if game_over == False:
			#jump
			if pygame.mouse.get_pressed()[0] == 1 and self.clicked == False:
				self.clicked = True
				self.vel = -10
			if pygame.mouse.get_pressed()[0] == 0:
				self.clicked = False

			#handle the animation
			flap_cooldown = 5
			self.counter += 1
			
			if self.counter > flap_cooldown:
				self.counter = 0
				self.index += 1
				if self.index >= len(self.images):
					self.index = 0
				self.image = self.images[self.index]

			#rotate the bird
			self.image = pygame.transform.rotate(self.images[self.index], self.vel * -2)
		else:
			#point the bird at the ground
			self.image = pygame.transform.rotate(self.images[self.index], -90)


class Food(pygame.sprite.Sprite):
	
	def __init__(self, x, y):
		super().__init__()
		self.images = []
		for num in range(17):
			img = pygame.image.load(f"img/apple/{num}.png")
			img = pygame.transform.scale(img, (80,80))
			self.images.append(img)			
		self.id_img = 0
		self.image = self.images[0]
		self.rect = self.image.get_rect(topleft = (x, y))
		self.cnt = 0
		self.time_cnt = 3

	def update(self):
		self.cnt += 1
		if self.cnt >= self.time_cnt:
			self.id_img += 1
			if self.id_img > 16:
				self.id_img = 0
			self.cnt = 0
			self.image = self.images[self.id_img]
		
		self.rect.x -= scroll_speed
		if self.rect.right < 0:
			self.kill()

class HP_bar:
	def __init__(self):
		self.img = pygame.transform.scale(pygame.image.load("img/hp.png"), (60, 60))
		self.slg = 1
	
	def add(self):
		self.slg = min(self.slg + 1, max_hp)
	def sub(self):
		self.slg -= 1
	
	def draw(self):
		draw_text('HP: ', font2, white, 20, 70)
		for i in range(self.slg):
			dx = 70 + 30 * i
			screen.blit(self.img, (dx, 60))
	
	
		
class Pipe(pygame.sprite.Sprite):

	def __init__(self, x, y, position):
		pygame.sprite.Sprite.__init__(self)
		self.image = pygame.image.load("img/pipe.png")
		self.rect = self.image.get_rect()
		#position variable determines if the pipe is coming from the bottom or top
		#position 1 is from the top, -1 is from the bottom
		if position == 1:
			self.image = pygame.transform.flip(self.image, False, True)
			self.rect.bottomleft = [x, y - int(pipe_gap / 2)]
		elif position == -1:
			self.rect.topleft = [x, y + int(pipe_gap / 2)]


	def update(self):
		self.rect.x -= scroll_speed
		if self.rect.right < 0:
			self.kill()



class Button():
	def __init__(self, x, y, image):
		self.image = image
		self.rect = self.image.get_rect()
		self.rect.topleft = (x, y)

	def draw(self):
		action = False

		#get mouse position
		pos = pygame.mouse.get_pos()

		#check mouseover and clicked conditions
		if self.rect.collidepoint(pos):
			if pygame.mouse.get_pressed()[0] == 1:
				action = True

		#draw button
		screen.blit(self.image, (self.rect.x, self.rect.y))

		return action


pipe_group = pygame.sprite.Group()
bird_group = pygame.sprite.GroupSingle()
food_group = pygame.sprite.Group()
flappy = Bird(100, int(screen_height / 2))

hp_bar = HP_bar()

bird_group.add(flappy)

#create restart button instance
button = Button(screen_width // 2 - 50, screen_height // 2 - 100, button_img)


run = True
while run:
	clock.tick(fps)

	#draw background
	screen.blit(bg, (0,0))
	
	pipe_group.draw(screen)
	bird_group.draw(screen)
	bird_group.update()
	hp_bar.draw()
	
	#draw and scroll the ground
	screen.blit(ground_img, (ground_scroll, 600))

	#check the score
	if len(pipe_group) > 0:
		if bird_group.sprites()[0].rect.left > pipe_group.sprites()[0].rect.left\
			and bird_group.sprites()[0].rect.right < pipe_group.sprites()[0].rect.right\
			and pass_pipe == False:
			pass_pipe = True
		if pass_pipe == True:
			if bird_group.sprites()[0].rect.left > pipe_group.sprites()[0].rect.right:
				score += 1
				pass_pipe = False


	draw_text(str(score), font, white, int(screen_width / 2), 20)
	draw_text('MAX SCORE: {}'.format(max_score), font2, (110,10,0), 20, 30)


	#look for collision
	if pygame.sprite.groupcollide(bird_group, pipe_group, False, False) or flappy.rect.top < 0:
		flappy.hp -= 1
		if flappy.hp <= 0:
			game_over = True
		else:
			hp_bar.sub()
			reset_game2()
	if pygame.sprite.groupcollide(food_group, pipe_group, True, False):
		pass
	if pygame.sprite.groupcollide(bird_group, food_group, False, True):
		flappy.hp = min(flappy.hp + 1, max_hp)
		hp_bar.add()

	#once the bird has hit the ground it's game over and no longer flying
	if flappy.rect.bottom >= 700:
		game_over = True
		flying = False


	if flying == True and game_over == False:
		#generate new pipes
		time_now = pygame.time.get_ticks()
		if time_now - last_pipe > pipe_frequency:
			pipe_height = random.randint(-80, 100)
			btm_pipe = Pipe(screen_width, int(screen_height / 2) - 150 + pipe_height, -1)
			top_pipe = Pipe(screen_width, int(screen_height / 2) - 150 + pipe_height, 1)
			pipe_group.add(btm_pipe)
			pipe_group.add(top_pipe)
			last_pipe = time_now
		pipe_group.update()


		# generate new food 
		if game_over == False:
			if food_appear == False:
				waitting_food = pygame.time.get_ticks() + random.randint(2000, 3500)
				food_appear = True
			else:
				if pygame.time.get_ticks() >= waitting_food:
					food_appear = False
					food = Food(random.randint(0,300) + screen_width, random.randint(200, 500))
					food_group.add(food)

		food_group.update()
		food_group.draw(screen)

		ground_scroll -= scroll_speed
		if abs(ground_scroll) > 35:
			ground_scroll = 0
	

	#check for game over and reset
	if game_over == True:
		draw_text('GAME OVER!!', font, white, 270, 200)
		max_score = max(max_score, score)
		food_appear = False
		if button.draw():
			game_over = False
			score = reset_game()


	for event in pygame.event.get():
		if event.type == pygame.QUIT:
			# update max_score
			filew = open('max_score.txt', 'w')
			filew.write(str(max_score))
			filew.close()

			run = False

		elif event.type == pygame.MOUSEBUTTONDOWN:
			flying = True
			food_appear = True

	pygame.display.update()

pygame.quit()
