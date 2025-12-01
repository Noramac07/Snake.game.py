# My Snake Game - In Python
For my computer science final project I chose to recreate the game Snake. In this game, the snake eats the food and grows as it eats, aswell the sanke has to avoid hitting the barriar and itself so it does not die and thus end the game. This game also includes things like speed levels, in game score and highscore tracking, random spawn, and arrow key conrol with space bar pause feature.
# How to run the game
Start by running the code, once the blue screen pops up choose a level based of what speed difficulty you want to use. Once the level is selected, press any arrow key to start the game, then using the arrow keys to move, try to eat the food and play until you die. At any point yu can pause, and once you die you can choose to try again with the same level or go and choose a new speed level.
# Required libraries to install
This uses pyton libraries, which do not require instalation. For extrenal libraries Tkinter and os were importated to create the game messages, peramiters, and files storage for highscores.
# Features
Light blue background, 3 speed level selection, random food spawn, game ends by hitting borders or the snakes body, top 5 highscores tracked, in game score displayed, and title displayed.
# Docstrings for all functions
__init__()
load_hs()
save_hs()
draw()
move()
paused()
startover()
go_back()
level()
change_direction()
# Full Code
#importing needed libraries
import tkinter
import random
import os

#game varibles values and functions
ROWS = 20
COLLUMS = 20
TILE_SIZE = 20 
WIDTH = 400 #overall width of game peramiters rows x collums
HEIGHT = 400 #overall height of game peramiters rows x collums

#game peramiters 
game = tkinter.Tk()
game.title("Nora's Snacky Snake Game")
game.resizable(False,False)

#game background and borders
BACKGROUND = tkinter.Canvas(game, bg = "LightBlue", width = WIDTH, height = HEIGHT, borderwidth = 2, highlightthickness = 2)
BACKGROUND.pack() #putting this within the window/ game peramiters
game.update() #add multiple items at once

#restart and back to menu messages
restart = tkinter.Button(game, text = "Try again?", font = "Arial 10", command = lambda: start_over()) 
restart.place_forget() #used to hide the message
Go_Back = tkinter.Button(game, text = "Level Selection", font = "Arial 10", command = lambda: go_back())
Go_Back.place_forget()

class TILE: #x and y grid
    def __init__(self,x,y):
        self.x = x
        self.y = y

#Snake, Food, and velocity peramiters
Snake = TILE(200,300) #snake's head start position
Food = TILE(200,100) #food's starting position
Snake_Body = [] #making multiple snake tiles as a list
Gameover = False #at start game isnt over
Pause = False
Score = 0 #starting score
Highscore = [] 
Speed = 100
VelX = 0 #starting velocity of both varibles before any keys are pressed
VelY = 0 
loop = None
Menu = None

Hsfile = "highscores.txt"

def load_hs():
    global Highscore
    if os.path.exists(Hsfile):
        with open(Hsfile, "r") as f:
            for line in f:
                line = line.strip()
                if line.isdigit():
                    Highscore.append(int(line))
        Highscore.sort(reverse=True)

def save_hs():
    with open(Hsfile, "w") as f:
        for score in Highscore:
            f.write(str(score) + "\n") #new line 
load_hs() #calling load highscore

def draw(): #draw function

    #defining caribles as global
    global Snake, Food, Snake_Body, Gameover, Score, loop, Pause

    BACKGROUND.delete("all")  # clear everything first

    if Pause:  # handle pause first
        BACKGROUND.create_text(200, 200, font = "Arial 30", text = "Paused, Continue?", fill = "black")
        BACKGROUND.create_text(40, 20, font = "Arial 10", text = f"Score: {Score}", fill = "Black")
        return  # stop here, don’t move or redraw snake

    # Game over screen
    if Gameover:
        BACKGROUND.create_text(200, 200, font = "Arial 30", text = f"Game Over!: {Score}", fill = "Black")
        restart.place(relx = 0.5, rely = 0.6, anchor = "center")
        Go_Back.place(relx = 0.5, rely = 0.7, anchor = "center")
        # Add highscore logic
        if Score not in Highscore:
            Highscore.append(Score)
            Highscore.sort(reverse=True)
            if len(Highscore) > 5:
                Highscore.pop()
            save_hs()
        BACKGROUND.create_text(200, 60, font = "Arial 15", text = f"New High Score!", fill = "Black")
        for i, hs in enumerate(Highscore):
            BACKGROUND.create_text(200, 85 + i*20, font = "Arial 10", text = f"{i+1}. {hs}", fill = "Black")
        return
    
    move() #actions within move function

    BACKGROUND.create_rectangle(Food.x, Food.y, Food.x + 20, Food.y + 20, fill = "#FF3366") #defining color and placement of food (food drawn first to have snake go above)
    BACKGROUND.create_rectangle(Snake.x, Snake.y, Snake.x + 20, Snake.y + 20, fill = "#3300FF") #defining color and placement of snake head
    for Tile in Snake_Body:
        BACKGROUND.create_rectangle(Tile.x, Tile.y, Tile.x + 20, Tile.y + 20, fill = "#3300FF") #defining tiles adding as snake eats and color

    BACKGROUND.create_text(40, 20, font = "Arial 10", text = f"Score: {Score}", fill = "Black")

    loop = game.after(Speed, draw)

def paused():
    global Pause, loop
    Pause = not Pause
    if not Pause:
        draw()  
game.bind("<space>", lambda e: paused())

def move(): #move function

    #define varbiles as global
    global Snake, Food, Snake_Body, Gameover, Score

    #Gameover - How to loose (wall and body)
    if (Gameover):
        return
    
    #Change length of snake body
    for i in range(len(Snake_Body)-1,-1,-1):
        Tile = Snake_Body[i]
        if (i == 0):  
            Tile.x = Snake.x
            Tile.y = Snake.y
        else: 
            PreviousTile = Snake_Body[i-1] #adding one to snake body every time a tile is consumed
            Tile.x = PreviousTile.x
            Tile.y = PreviousTile.y

    #move over one pixel each time + Re draw/ print pixels
    Snake.x += VelX * 20 
    Snake.y += VelY * 20 

    if (Snake.x < 0 or Snake.x >= WIDTH or Snake.y < 0 or Snake.y >= HEIGHT): #How to lose by hitting the wall
        Gameover = True
        return
    for Tile in Snake_Body: #How to lose by hitting the body of the snake
        if (Snake.x == Tile.x and Snake.y == Tile.y):
            Gameover = True
            return
        
    #Snake into Food collison
    if (Snake.x == Food.x and Snake.y == Food.y): #Once food is eaten, adds to snake body 
        Snake_Body.append(TILE(Food.x,Food.y))
        Food.x = random.randint(0,COLLUMS-1) * 20 #Once food is eaten, it moves to a random place 
        Food.y = random.randint(0,ROWS-1) * 20
        Score += 1 #adds one to the score each time food collison occures
        
#Restart - how to try again   
def start_over():
    global loop, Snake, Food, Snake_Body, VelX, VelY, Gameover, Score

    if loop is not None:
        game.after_cancel(loop) #stop previous loop for running

    Snake.x = 200 #redefining peramiters for these functions
    Snake.y = 300
    Food.x = 200
    Food.y = 100
    Snake_Body = []
    VelX = 0
    VelY = 0
    Gameover = False
    Score = 0
        
    restart.place_forget() #used to hide the message
    Go_Back.place_forget()

    draw()

#return to menu function
def go_back():
    global loop, Snake, Snake_Body, VelX, VelY, Gameover, Score, Menu

    if loop is not None:
        game.after_cancel(loop) #redefing peramiters for these functions

    Snake.x = 200 #redefining peramiters for these functions
    Snake.y = 300
    Food.x = 200
    Food.y = 100
    Snake_Body = []
    VelX = 0
    VelY = 0
    Gameover = False
    Score = 0
        
    restart.place_forget() #used to hide the message
    Go_Back.place_forget()

    Menu = tkinter.Frame(game, bg = "Black") #back to menu message
    Menu.place(relx = 0.5, rely = 0.5, anchor = "center") #placing message

    message = tkinter.Label(Menu, text = "Select Difficulty", fg = "white", bg = "black", font = "Arial 25") #level selection message
    message.pack(pady = 10) #spacing

    B1 = tkinter.Button(Menu, text = "Easy", width = 25, font = "Arial 20", command = lambda: level(150)) #easy level option button
    B1.pack(pady = 5)
    B2 = tkinter.Button(Menu, text = "Medium", width = 25, font = "Arial 20", command = lambda: level(100)) #medium level button
    B2.pack(pady = 5)
    B3 = tkinter.Button(Menu, text = "Difficult", width = 25, font = "Arial 20", command = lambda: level(50)) #hard level button
    B3.pack(pady = 5)

#Different speed levels
def level(level_speed):
    global Menu, Speed
    Speed = level_speed
    Menu.destroy()
    draw()

Menu = tkinter.Frame(game, bg = "Black")
Menu.place(relx = 0.5, rely = 0.5, anchor = "center")

message = tkinter.Label(Menu, text = "Select Difficulty", fg = "white", bg = "black", font = "Arial 30")
message.pack(pady = 10)

B1 = tkinter.Button(Menu, text = "Easy", width = 25, font = "Arial 25", command = lambda: level(150))
B1.pack(pady = 5)
B2 = tkinter.Button(Menu, text = "Medium", width = 25, font = "Arial 25", command = lambda: level(100))
B2.pack(pady = 5)
B3 = tkinter.Button(Menu, text = "Difficult", width = 25, font = "Arial 25", command = lambda: level(50))
B3.pack(pady = 5)

#Movemnt using arrow keys
def change_direction(e): #e means event
    global VelX, VelY
    
    if (e.keysym == "Up" and VelY != 1): #defining the direction of movement for each arrow key
        VelX = 0
        VelY = -1
    elif (e.keysym == "Down" and VelY != -1):
        VelX = 0
        VelY = 1
    elif (e.keysym == "Right" and VelX != -1):
        VelX = 1
        VelY = 0
    elif (e.keysym == "Left" and VelX != 1):
        VelX = -1
        VelY = 0
    if loop is not None:
        game.after_cancel(loop)
    loop = game.after(Speed, draw)

game.bind("<KeyRelease>", change_direction) #once a key is pressed, it releases and movement starts
game.mainloop()
