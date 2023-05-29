# shooting_game6
import pygame
import random

from pygame import draw
from pygame.color import Color
from pygame.sprite import Sprite
from pygame.surface import Surface

from player import Player
from enemy import Enemy
from bullet import Bullet
from boss import Boss
from item import Item
from shield import Shield
from plus import Plus
from minus import Minus
from animation import Animation

pygame.init()

# 게임 화면 설정
image_list = ['background.png','backgrounds.png'] #배경회면 2개 더 늘려주기
size = (1000, 986)
screen = pygame.display.set_mode(size, pygame.FULLSCREEN)
pygame.display.set_caption("슈팅 게임")

#배경화면 클래스
class Background(Sprite):
    def __init__(self):
        self.i = 0
        self.sprite_image = image_list[self.i]
        self.image = pygame.image.load(self.sprite_image).convert()
        self.rect = self.image.get_rect()
        self.rect.y = 0
        self.dy = 1
        self.count = 0
        
        Sprite.__init__(self)

    def update(self):
        self.rect.y -= self.dy
        if self.rect.y == -400:
            self.rect.y = 0
            self.count += 1
            
        if self.count == 5:
            self.i = self.i + 1
            self.sprite_image = image_list[self.i]
            self.image = pygame.image.load(self.sprite_image).convert()
            self.rect = self.image.get_rect()
            self.count = 0

#이미지 그룹
background = Background()
background_group = pygame.sprite.Group()
background_group.add(background)

player = Player()
player.rect.x = 450
player.rect.y = 900
player_group = pygame.sprite.Group()
player_group.add(player)

boss = Boss()
boss.rect.x = 500
boss.rect.y = -5000
boss_group = pygame.sprite.Group()
boss_group.add(boss)

shield = Shield()
shield.rect.x = -5000
shield.rect.y = -5000
shield_group = pygame.sprite.Group()
shield_group.add(shield)

plus = Plus()
plus.rect.x = -4000
plus.rect.y = -4000
plus_group = pygame.sprite.Group()
plus_group.add(plus)

minus = Minus()
minus.rect.x = - 3000
minus.rect.y = - 3000
minus_group = pygame.sprite.Group()
minus_group.add(minus)

bullet_group = pygame.sprite.Group()

#변수
bullets = 0
enemy_count = 0
boss_count = 0
shields = 0
item_li = 0
count = 0
space_bar = 0
shield_score = 0
boss_score = 0
blt_count = 0
bullet_count = 0
blt_rect = 0

count_li = [0, 1]

#클래스 리스트
cls_list = [shield, plus, minus]

for i in range(10000):
    b = Bullet()
    b.rect.x = -1000
    b.rect.y = -1000
    bullet_group.add(b)

#적의 좌표 생성
enemys_x = []
enemys_y = []

for i in range(10):
    enemys_x.append(random.randint(0, 884))
    enemys_y.append(random.randint(-400, 0))

enemy_group = pygame.sprite.Group()

for k in range(10):
    enemy = Enemy(enemys_x[k], enemys_y[k])
    enemy_group.add(enemy)

#아이템의 좌표 생성
items_x = []
items_y = []

for j in range(10):
    items_x.append(random.randint(0, 950))
    items_y.append(random.randint(-400, 0))

item_group = pygame.sprite.Group()

# 게임 루프
running = True
clock = pygame.time.Clock()

while running:
    # 이벤트 처리
    for event in pygame.event.get(): 
        if event.type == pygame.QUIT:
            running = False
        elif event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE:
                #player.rect.x + 67 / 2을 기준점으로 시작 홀수면 -10 짝수면 +10
                bullet_group.sprites()[bullets].shoot(player.rect.x + 67 / 2, player.rect.y)
                bullets += 1
                if bullets == 10000:
                    running = False
                if blt_count % 2 == 1 or blt_count % 2 == 0 and blt_count > 1: #홀수 -10 짝수 +10
                    odd = blt_count//2+1
                    even = blt_count//2
                    blt_rect = 10
                    for k in range(odd):
                        bullet_group.sprites()[bullets].shoot(player.rect.x + 67 / 2 - blt_rect, player.rect.y)
                        bullets += blt_count
                        blt_rect+=10
                    blt_rect = 10
                    for h in range(even):
                        bullet_group.sprites()[bullets].shoot(player.rect.x + 67 / 2 + blt_rect, player.rect.y)
                        bullets += blt_count
                        blt_rect += 10


    #스코어 점수가 400이 되면 보스 출현
    if player.score % 400 == 0:
        boss.rect.x = 250
        boss.rect.y = 100
        boss_group.add(boss)

    #스코어가 200씩 늘어날 때 마다 쏜다.
    if player.score % 200 == 0:
        items = Item(items_x[item_li], items_y[item_li])
        item_group.add(items)
        item_li += 1
        player.score += 10

    if item_li == 10:
        item_li = 0

    # 게임 상태 업데이트
    player_group.update()
    bullet_group.update()
    enemy_group.update()
    boss_group.update()
    item_group.update()
    shield_group.update()
    plus_group.update()
    minus_group.update()

    # 충돌 처리
    if enemy.alive():
        hits = pygame.sprite.groupcollide(enemy_group, bullet_group, True, True)
        if hits:
            player.score += 10
            t = list(hits.keys())[0]
            t.rect.y = -200
            enemy_group.add(t)

    if boss.alive():
        hitss = pygame.sprite.groupcollide(boss_group, bullet_group, False, True)
        if hitss:
            boss_count += 1
            if boss_count == 30:
                boss_count = 0
                boss.kill()
                boss_score += 1

    if boss_score == 1:
        boss.rect.y = -5000
        boss_group.add(boss)
        boss_score = 0

    if player.alive():
        hit = pygame.sprite.groupcollide(item_group, player_group, True, False)
        if hit:
            shields += 1
            vle = random.randint(0, 2)
            cls_list[vle].rect.x = (player.rect.x + player.rect.width/2) - cls_list[vle].rect.width/2
            cls_list[vle].rect.y = player.rect.y
            count += 1
            if vle == 0:
                shield_score += 1
            if vle == 1:
                blt_count += 1
            if vle == 2:
                blt_count -= 1

    if shield_score == 1:
        shield_group.add(shield)
        shield_score = 0

    if count != 0:
        cls_list[vle].rect.x = (player.rect.x + player.rect.width/2) - cls_list[vle].rect.width/2
        cls_list[vle].rect.y = player.rect.y
            
    if shield.alive():
        hitt = pygame.sprite.groupcollide(shield_group, enemy_group, True, True)
        if hitt:
            count = 0
            
    if player.alive():
        hitse = pygame.sprite.groupcollide(player_group, enemy_group, True, True)
        if hitse:
            running = False

    # 화면 그리기
    background_group.update()
    background_group.draw(screen)
    player_group.draw(screen)
    bullet_group.draw(screen)
    enemy_group.draw(screen)
    boss_group.draw(screen)
    item_group.draw(screen)
    shield_group.draw(screen)
    plus_group.draw(screen)
    minus_group.draw(screen)

    # 점수 출력
    font = pygame.font.Font(None, 30)
    score_text = font.render('Score: {}'.format(player.score), True, (0, 0, 0))
    screen.blit(score_text, (10, 10))

    font = pygame.font.Font(None, 30)
    score_text = font.render('boss: {}'.format(boss_count), True, (0, 0, 0))
    screen.blit(score_text, (10, 30))

    # 화면 업데이트
    pygame.display.update()

    # 프레임 설정
    clock.tick(60)

pygame.quit()
