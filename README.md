# Greedy_Python
 #a gluttonous snake game by using python's pygame library
import pygame                     #pygame library
import random

SCREEN_SIZE = [1000, 800]         #constant: ez change for other monitor size
BACKGROUND_COLOR = (255,255,255)  #Background color of the game interface, :rgb = white


CELL_RADIUS = 20                  #Snake radius and color
SNAKE_COLOR = (156,231,100)

FOOD_COLOR = (255,0,0)
FOOD_RADIUS = 10


UPDATE = pygame.USEREVENT + 1
FOOD = pygame.USEREVENT + 2

FINAL_WORD_COLOR = (0,100,151)
MSG_POSITION = (200, 350)

def init_game():
    pygame.init()
    pygame.display.set_caption("Gluttonous Snake 贪吃蛇")  #window title
    pygame.time.set_timer(UPDATE,500)                      # The snake updates its own position every half second
    pygame.time.set_timer(FOOD,1000)                       # Food updates its location every second
    
class Game:
    def __init__(self):
        self.running = True                                 # Game starts
        self.screen = pygame.display.set_mode(SCREEN_SIZE)  # Set the window size when the game is running, you can change
        self.snake = Snake()                                # Create our snake
        self.food = None
        self.message = None
  
        
class Cell:                                                 #Our snake and food is writed by circiles so we need a class to statement
    def __init__(self,x,y):                                 
        self.x = x
        self.y = y
        
    def to_tuple(self):                                   #Coordinations
        return self.x, self.y             
    
    def copy(self):
        return Cell(self.x, self.y)
    
    def update(self, deriction):                        #Changes in body coordinates when the snake is asked to change direction
        if deriction == "U":
            self.y -= CELL_RADIUS * 2
        if deriction == "D":
            self.y += CELL_RADIUS * 2  
        if deriction == "L":
            self.x -= CELL_RADIUS * 2 
        if deriction == "R":
            self.x += CELL_RADIUS * 2        
  
    
        
class Snake:
    def __init__(self):
        cell_diameter = CELL_RADIUS * 2
        x = (SCREEN_SIZE[0] / cell_diameter //2) * cell_diameter
        y = (SCREEN_SIZE[1] / cell_diameter //2) * cell_diameter  #snake start position at middle of screen
        
        self.body = [Cell(x,y)]
        self.cell_size = CELL_RADIUS
        self.color = SNAKE_COLOR
        self.direction = 'L'                                      # Initial direction of motion


        
    def update(self):                                             # In order to create a visual change when the snake is moving in one direction,                                                         
        head = self.body[0].copy()                                #I used to remove the last joint of the snake and add the new joint to the forefront of the body in the direction of the snake's movement
        self.body.pop()
        head.update(self.direction)
        self.body.insert(0,head)
        
        
        
        
#**********************************************************************************
def update():                                                                  #Parts to check at all times
    check_snake_dir()                                                          #Change the direction of the snake's movement by pressing the arrow keys from the keyboard                                                  
    check_food()                                                               #Check if the food is generated
    check_head_body_collision()                                                #Check if the snake is touching its body
    check_out_boundry()                                                        #Check if the snake's movement is beyond the screen boundary
    check_win()                                                                #Check the victory conditions
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            game.running = False
        if event.type == UPDATE:
            game.snake.update()
        if event.type == FOOD:
            generate_food()
            
            
def check_snake_dir():
    pressed_keys = pygame.key.get_pressed()
    if pressed_keys[pygame.K_UP] and not game.snake.direction == "D":
        game.snake.direction = "U"
    if pressed_keys[pygame.K_DOWN] and not game.snake.direction == "U":
        game.snake.direction = "D"
    if pressed_keys[pygame.K_LEFT] and not game.snake.direction == "R":
        game.snake.direction = "L"
    if pressed_keys[pygame.K_RIGHT] and not game.snake.direction == "L":
        game.snake.direction = "R"                


def check_food():
    if game.food is not None and is_snake_food_collide():                           #When the food is eaten by the snake, 
        cell = Cell(game.food[0], game.food[1])                                     #the coordinates of the food will become the coordinates of the snake's new body, 
        game.snake.body.insert(0,cell)                                              #and this new part of the body is inserted into the very front of the body，
        game.food = None                                                            #And the food is disappeared.
        
def is_snake_food_collide():                                                       
    head = game.snake.body[0].copy()
    head.update(game.snake.direction)
    return (game.food[0] - head.x) == 0 and (game.food[1] - head.y) == 0           #Check if the coordinates of the food and the first part of the snake's body exactly coincide to determine if the snake has got the food


def check_head_body_collision():                                                  #Check whether the first part of the snake's body, 
    body = game.snake.body                                                        #that is, the head, touches the other parts of the body, and if it does, 
    try:                                                                          #then the body that was touched and all subsequent parts disappear and the snake's body is shortened
        index = [cell.to_tuple() for cell in body[1:]].index(head.to_tuple())
    except:
        index = -1
    if len(body) > 1 and index > -1:
        game.snake.body = game.snake.body[:index]

        
#**************************************************************************************        
        
               
def generate_food():
    while game.food is None:                                                                             #When the food is not generated or eaten by the snake, 
        cell_diameter = CELL_RADIUS *2                                                                   #I need to generate new food in the interface, the location of the food generation is random,
        rand_x = random.randint(1,(SCREEN_SIZE[0]/ cell_diameter) -1) *cell_diameter                     # but the food can not be generated in the body of the snake, also cannot go beyond the screen.
        rand_y = random.randint(1,(SCREEN_SIZE[1]/ cell_diameter) -1) *cell_diameter                     
        if (rand_x, rand_y) not in [cell.to_tuple() for cell in game.snake.body]:
            game.food = (rand_x, rand_y)

def draw():
    game.screen.fill(BACKGROUND_COLOR)
    if game.running:
        draw_snake()
        draw_food()
    else:
        draw_restart()
    pygame.display.flip()
    
def draw_restart():
    font = pygame.font.Font(None,30)
    restart_msg = font.render(game.message, True, FINAL_WORD_COLOR)
    game.screen.blit(restart_msg, MSG_POSITION)
    
def draw_food():
    if game.food is not None:
        pygame.draw.circle(game.screen, FOOD_COLOR, game.food, FOOD_RADIUS)
    
def draw_snake():
    for cell in game.snake.body:
        pygame.draw.circle(game.screen, game.snake.color,cell.to_tuple(), game.snake.cell_size)    
        
#*************************************************************************  distingush win/lose     
def check_out_boundry(): #可改
    cell_diameter = 2*CELL_RADIUS                                                                                    #Determine the trajectory of the snake
    if  game.snake.body[0].x < cell_diameter or game.snake.body[0].x > SCREEN_SIZE[0] - cell_diameter:               #If the x-coordinate of the first part of the snake's body is smaller than the radius of the snake's body, 
        game.running = False                                                                                           #then the snake has moved out of the left boundary of the screen
        game.message = "Snake is out of boundry. You wanna try again?"+" Press 'y' or 'n'"                         #If the x-coordinate of the first part of the snake's body is greater than the border of the snake's game screen, 
    if game.snake.body[0].y < cell_diameter or game.snake.body[0].y > SCREEN_SIZE[1] - cell_diameter:                  #then the snake has moved out of the right border of the screen
        game.running = False                                                                                        #y coordinate is the same
        game.message = "Snake is out of boundry. You wanna try again?"+" Press 'y' or 'n'"      
        
def check_win():
    length = len(game.snake.body)                                                                                  #Judgment of victory based on the length of the snake's body can be changed
    if length >= 4:
        game.running = False
        game.message = "Congrats, You WIN! You wanna try again?"+" Press 'y' or 'n'"
        
def check_restart():                                                                                              #When winning or losing the user can choose whether to restart, y for restart, n for closing the game
    while not game.running:
        for event in pygame.event.get():
            if event.type == pygame.KEYUP and event.key == pygame.K_y:
                game.snake = Snake()
                game.running = True
            if event.type == pygame.KEYUP and event.key == pygame.K_n:
                return  
        

    
if __name__ == '__main__':
    init_game()
    game = Game()
    while game.running:
        update()
        draw()
        check_restart()
    pygame.quit()
        
