import pygame
import random
import time

# Inicializa o pygame
pygame.init()

# Configurações da tela
WIDTH, HEIGHT = 1920, 1080
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Animal Fighters")

# Taxa de atualização
clock = pygame.time.Clock()
FPS = 60

# Física
GRAVITY = 1
JUMP_STRENGTH = -25
KNOCKBACK_FORCE = 15

# Cores
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GREEN = (0, 255, 0)

# Fonte do placar
font = pygame.font.Font(None, 74)

# Placar de vitórias
score_p1 = 0
score_p2 = 0
max_score = 3

# Controle de tempo de ataque
attack_cooldown = 0.6  # 0.6 segundos

# Carregar imagens dos personagens e fundos
try:
    hit_effect = pygame.transform.scale(pygame.image.load("hit_effect.png"), (50, 50))
    personagens = {
        "Galinha": pygame.image.load("galinha.png"),
        "Esquilo": pygame.image.load("esquilo.png"),
        "Raposa": pygame.image.load("raposa.png"),
        "Lobo": pygame.image.load("lobo.png"),
        "Urso": pygame.image.load("urso.png"),
        "Guaxinim": pygame.image.load("guaxinim.png"),
        "Jaguatirica": pygame.image.load("jaguatirica.png"),
        "Onça Pintada": pygame.image.load("onca_pintada.png"),
        "Macaco": pygame.image.load("macaco.png")
    }
    for key in personagens:
        personagens[key] = pygame.transform.scale(personagens[key], (200, 200))

    fundos = [
        pygame.image.load("fundo1.png"),
        pygame.image.load("fundo2.png"),
        pygame.image.load("fundo3.png")
    ]
    for i in range(len(fundos)):
        fundos[i] = pygame.transform.scale(fundos[i], (WIDTH, HEIGHT))
except pygame.error:
    print("Erro ao carregar imagens. Certifique-se de que os arquivos PNG estão na mesma pasta que o código.")
    exit()

# Classe do jogador
class Player:
    def __init__(self, x, y, character_name):
        self.x = x
        self.y = y
        self.character_name = character_name
        self.image = personagens[character_name]
        self.speed = 7
        self.health = 100
        self.attack_power = 1
        self.hit_effect_time = 0
        self.hit_effect_size = (100, 100)
        self.width = 200
        self.height = 200
        self.last_attack_time = 0
        self.vel_y = 0
        self.on_ground = False
    
    def move(self, keys, left, right, jump):
        if keys[left] and self.x > 0:
            self.x -= self.speed
        if keys[right] and self.x < WIDTH - self.width:
            self.x += self.speed
        if keys[jump] and self.on_ground:
            self.vel_y = JUMP_STRENGTH
            self.on_ground = False
        
        # Aplicar gravidade
        self.vel_y += GRAVITY
        if self.vel_y > 10:
            self.vel_y = 10  # Limita a velocidade de queda
        
        self.y += self.vel_y
        if self.y >= HEIGHT - self.height - 230:
            self.y = HEIGHT - self.height - 230
            self.vel_y = 0
            self.on_ground = True
    
    def attack(self, opponent):
        global running
        current_time = time.time()
        if current_time - self.last_attack_time >= attack_cooldown:
            print(f'{self.character_name} atacou {opponent.character_name}! Vida restante: {opponent.health - self.attack_power}/20')
            if abs(self.x - opponent.x) <= 200 and abs(self.y - opponent.y) <= 100:
                opponent.health -= 5
                if opponent.health <= 0:
                    check_winner()
                opponent.hit_effect_time = pygame.time.get_ticks()
                if pygame.time.get_ticks() - opponent.hit_effect_time < 200:
                    screen.blit(pygame.transform.scale(hit_effect, (100, 100)), (opponent.x + 75, opponent.y - 20))
                
                # Knockback ao ser atingido
                if self.x < opponent.x:
                    opponent.x += KNOCKBACK_FORCE
                else:
                    opponent.x -= KNOCKBACK_FORCE
                opponent.vel_y = -10  # Pequeno impulso para cima ao ser atingido
                self.last_attack_time = current_time
    
    def draw(self, screen):
        screen.blit(self.image, (self.x, self.y))

# Loop principal do jogo





def select_background():
    global selected_background
    selecting = True
    while selecting:
        screen.fill(WHITE)
        for i, fundo in enumerate(fundos):
            screen.blit(fundo, (i * WIDTH // 3, 0))
            
        pygame.display.flip()
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
            if event.type == pygame.MOUSEBUTTONDOWN:
                x = event.pos[0] // (WIDTH // 3)
                selected_background = fundos[x]
                selecting = False

select_background()
    
def select_characters():
        global p1_character, p2_character
        p1_character = None
        p2_character = None
        selecting = True
        while selecting:
            screen.fill(WHITE)
            screen.blit(selected_background, (0, 0))
            for i, name in enumerate(personagens.keys()):
                x = WIDTH//2 - 250 + (i % 3) * 220  
                y = HEIGHT//2 - 300 + (i // 3) * 220  
                screen.blit(personagens[name], (x, y))
            pygame.display.flip()
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    exit()
                if event.type == pygame.MOUSEBUTTONDOWN:
                    for i, name in enumerate(personagens.keys()):
                        x = WIDTH//2 - 250 + (i % 3) * 220  
                        y = HEIGHT//2 - 300 + (i // 3) * 220  
                        if x < event.pos[0] < x + 200 and y < event.pos[1] < y + 200:
                            if p1_character is None:
                                p1_character = name
                            else:
                                p2_character = name
                                selecting = False
                                break

select_characters()
def check_winner():
    global running, player1, player2
    if player1.health <= 0 or player2.health <= 0:
        winner = player2.character_name if player1.health <= 0 else player1.character_name
        print(f"{winner} venceu! Reiniciando...")
        pygame.display.flip()
        
        select_characters()
        player1 = Player(400, HEIGHT//2, p1_character)
        player2 = Player(WIDTH - 600, HEIGHT//2, p2_character)
        print(f"{player2.character_name} venceu! Reiniciando em 3 segundos...")
        pygame.display.flip()
        
        select_characters()
        player1 = Player(400, HEIGHT//2, p1_character)
        player2 = Player(WIDTH - 600, HEIGHT//2, p2_character)
        running = True
    elif player2.health <= 0:
        print(f"{player1.character_name} venceu! Reiniciando em 3 segundos...")
        pygame.display.flip()
        time.sleep(3)
        select_characters()
        player1 = Player(400, HEIGHT//2, p1_character)
        player2 = Player(WIDTH - 600, HEIGHT//2, p2_character)
        running = True
    elif player2.health <= 0:
        print(f"{player1.character_name} venceu! Reiniciando em 3 segundos...")
        pygame.display.flip()
        time.sleep(3)
        select_characters()
        player1 = Player(400, HEIGHT//2, p1_character)
        player2 = Player(WIDTH - 600, HEIGHT//2, p2_character)
        running = True
    elif player2.health <= 0:
        print(f"{player1.character_name} venceu! Reiniciando...")
        select_characters()
        player1 = Player(400, HEIGHT//2, p1_character)
        player2 = Player(WIDTH - 600, HEIGHT//2, p2_character)
        running = True
    elif player2.health <= 0:
        print("Player 1 venceu! Reiniciando...")
        select_characters()
        player1 = Player(400, HEIGHT//2, p1_character)
        player2 = Player(WIDTH - 600, HEIGHT//2, p2_character)
        running = True
        print("Reiniciando o jogo...")
        select_characters()
        player1 = Player(400, HEIGHT//2, p1_character)
        player2 = Player(WIDTH - 600, HEIGHT//2, p2_character)
        print("Player 2 venceu! Reiniciando...")
        select_characters()
        player1 = Player(400, HEIGHT//2, p1_character)
        player2 = Player(WIDTH - 600, HEIGHT//2, p2_character)
        running = False
    elif player2.health <= 0:
        print("Player 1 venceu! Reiniciando...")
        select_characters()
        player1 = Player(400, HEIGHT//2, p1_character)
        player2 = Player(WIDTH - 600, HEIGHT//2, p2_character)
        running = False
    elif player2.health <= 0:
        print("Player 1 venceu! Reiniciando...")
        select_characters()
        player1 = Player(400, HEIGHT//2, p1_character)
        player2 = Player(WIDTH - 600, HEIGHT//2, p2_character)

player1 = Player(400, HEIGHT//2, p1_character)
player2 = Player(WIDTH - 600, HEIGHT//2, p2_character)

    



running = True
while running:
        clock.tick(FPS)
        screen.fill(WHITE)
        screen.blit(selected_background, (0, 0))
        
        keys = pygame.key.get_pressed()
        player1.move(keys, pygame.K_a, pygame.K_d, pygame.K_w)
        player2.move(keys, pygame.K_LEFT, pygame.K_RIGHT, pygame.K_UP)
        
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_r:
                    player1.attack(player2)
                if event.key == pygame.K_KP2:
                    player2.attack(player1)
        
        pygame.draw.rect(screen, BLACK, (50, 30, 600, 30))
        pygame.draw.rect(screen, RED, (50, 30, 600, 30))
        pygame.draw.rect(screen, GREEN, (50, 30, 600 * (player1.health / 100), 30))
        pygame.draw.rect(screen, BLACK, (WIDTH - 650, 30, 600, 30))
        pygame.draw.rect(screen, RED, (WIDTH - 650, 30, 600, 30))
        pygame.draw.rect(screen, GREEN, (WIDTH - 650, 30, 600 * (player2.health / 100), 30))
        
        player1.draw(screen)
        player2.draw(screen)
        
        pygame.display.flip()
