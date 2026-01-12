# PygameCode

# link: https://youtu.be/gW_IIJXU2dc 

# Import necessary modules from the pygame library
from pygame import *
import random, sys

# Define a snowflake class that represents falling snowflakes in the game
class Flake(sprite.Sprite):
    def _init_(self):
        sprite.Sprite._init_(self)  # Initialize the Sprite base class
        self.image = Surface((11, 11))  # Create a surface for the snowflake
        self.rect = self.image.get_rect()  # Get the rectangular boundary of the surface
        
# Draw the snowflake shape using lines
        draw.line(self.image, Color("white"), (5, 0), (5, 10))
        draw.line(self.image, Color("white"), (0, 2), (10, 8))
        draw.line(self.image, Color("white"), (0, 8), (10, 2))
        
        self.vy = 1  # Set the vertical speed of the snowflake
        flakes.add(self)  # Add the snowflake to the flakes group

    def update(self):
# Update the snowflake's position
        self.rect.y += self.vy  # Move the snowflake down
        self.rect.x = (self.rect.x + self.vx) % width  # Handle horizontal wrapping

# Remove the snowflake if it reaches the ground
        if self.rect.y > height - 50:
            flakes.remove(self)

# Check for collision with the angel and reduce lives if hit
        if angel.lives > 0 and self.rect.colliderect(angel.rect):
            angel.lives -= 1
            startFloor()

# Define the Angel class, representing the player's character
class Angel(sprite.Sprite):
    def _init_(self):
        sprite.Sprite._init_(self)  # Initialize the Sprite base class
        self.image = Surface((11, 17))  # Create a surface for the angel
        self.rect = self.image.get_rect()  # Get the rectangular boundary of the surface
        
# Draw the angel shape using a polygon and circle
        poly = ((0, 1), (2, 4), (8, 4), (10, 1), (10, 8), (7, 10), 
                (10, 16), (0, 16), (3, 10), (0, 8))
        draw.polygon(self.image, Color("white"), poly)
        draw.circle(self.image, Color("white"), (5, 2), 2)
        
        self.lives = 3  # Initialize the angel's lives

    def update(self):
# Handle angel movement based on keyboard input
        keys = key.get_pressed()  # Get the current state of the keyboard
        if keys[K_UP]:
            self.rect.y -= 3  # Move up
        if keys[K_DOWN] and self.rect.bottom < height:
            self.rect.y += 3  # Move down
        if keys[K_LEFT] and self.rect.x > 0:
            self.rect.x -= 3  # Move left
        if keys[K_RIGHT] and self.rect.right < width:
            self.rect.x += 3  # Move right

# Check if the angel reaches the next floor or specific keys are pressed
        if (keys[K_l] and self.rect.bottom < height) or self.rect.top < 25:
            startFloor(min(floor + 1, len(vxRanges)))

# Initialize game variables
flakes = sprite.RenderPlain() 
init() 
width, height = 800, 600  
window = display.set_mode((width, height))  
screen = display.get_surface()  
fnt = font.Font(None, 24)  

# List of multiple-choice questions for the game
questions = [
    {"q": "What does 'TLS' stand for in the context of secure communication?", "choices": [
        "A. Transport Layer Security",
        "B. Transmission Line Standard",
        "C. Trusted Link Service",
        "D. Transport Level Signaling"], "answer": "A"},
    {"q": "Which of the following is a primary function of a TLS client?", "choices": [
        "A. Encrypt data between the client and server",
        "B. Generate the server's private key",
        "C. Open unencrypted communication channels",
        "D. Validate the server's IP address"], "answer": "A"},
    {"q": "What is the main role of the TLS handshake process?", "choices": [
        "A. To perform encryption of data",
        "B. To authenticate and establish a secure connection",
        "C. To compress data for faster transmission",
        "D. To initiate a TCP connection"], "answer": "B"},
    {"q": "Which of these is a common certificate validation step performed by a TLS client?", "choices": [
        "A. Verifying the certificate's expiration date",
        "B. Encrypting the server's certificate",
        "C. Sending an HTTP GET request to validate the certificate",
        "D. Skipping the validation process"], "answer": "A"},
    {"q": "In TLS communication, what does the client use to verify the server's identity?", "choices": [
        "A. The server's private key",
        "B. The server's public certificate",
        "C. The server's MAC address",
        "D. The client's pre-shared key"], "answer": "B"}
]
user_answers = []  # List to store the user's answers

# Function to display a multiple-choice question
def ask_question(question):
    screen.fill((0, 0, 0))  # Clear the screen
    write(question["q"], width // 2, 100, center=1)  # Display the question
    for i, choice in enumerate(question["choices"]):
        write(choice, width // 2, 150 + i * 30, center=1)  # Display the choices
    display.flip()  # Update the display

    selected = None
    while selected is None:  # Wait for the user to select an answer
        for ev in event.get():
            if ev.type == constants.QUIT:  # Exit if the user closes the window
                exit(0)
            if ev.type == constants.KEYDOWN:  # Check for key press
                if ev.unicode.upper() in ["A", "B", "C", "D"]:  # Validate input
                    selected = ev.unicode.upper()
    return selected  # Return the selected answer

# Function to render text on the screen
def write(s, x, y, center=0):
    text = fnt.render(s, True, Color("white"))  # Render text with the font
    screen.blit(text, (x - center * text.get_width() / 2, y))  # Draw text on the screen

# Function to display text input on the screen
def get_input(prompt, x, y):
    input_active = True
    user_text = ""  # Store user input
    while input_active:
        for ev in event.get():
            if ev.type == constants.QUIT:  # Exit if the user closes the window
                exit(0)
            if ev.type == constants.KEYDOWN:
                if ev.key == K_RETURN:  # Finish input on Enter key
                    input_active = False
                elif ev.key == K_BACKSPACE:  # Remove the last character
                    user_text = user_text[:-1]
                else:
                    user_text += ev.unicode  # Add the typed character
        screen.fill((0, 0, 0))  # Clear the screen
        write(prompt, x, y)  # Display the prompt
        write(user_text, x, y + 40)  # Display the current input
        display.flip()  # Update the display
    return user_text  # Return the user input

# Prompt the user for their name and matric number
name = get_input("Enter your name:", width // 2, height // 2 - 40)
matric_no = get_input("Enter your matric number:", width // 2, height // 2)

# Ask the first question
user_answers.append(ask_question(questions[0]))

# Ranges for flake velocities and spawn frequencies
vxRanges = ((0, 0), (-1, 1), (1, 2), (-2, 2), (-3, 3))
vyRanges = ((1, 1), (1, 2), (1, 2), (1, 3), (1, 3))
freqs = (0.1, 0.15, 0.2, 0.2, 0.15)

clock = time.Clock()  # Create a clock to control frame rate
angel = Angel()  # Create the angel sprite

# Function to update and spawn snowflakes
def updateFlakes():
    if random.random() < freqs[floor - 1]:  # Spawn a new flake based on the frequency
        f = Flake()
        f.rect.x = random.randint(0, 3 * width)  # Set a random horizontal position
        f.rect.bottom = 0  # Start at the top of the screen
        f.vx = random.randint(*vxRanges[floor - 1])  # Set horizontal speed
        f.vy = random.randint(*vyRanges[floor - 1])  # Set vertical speed
    flakes.update()  # Update all flakes

floor = 1  # Initialize the starting floor

# Function to reset the game state for a new floor
def startFloor(flr=0):
    global floor
    angel.rect.x = width / 2  # Center the angel horizontally
    angel.rect.bottom = height  # Place the angel at the bottom of the screen
    if flr:
        floor = flr  # Set the current floor
        for i in range(800):
            updateFlakes()  # Pre-spawn flakes for the new floor

startFloor(1)  # Start the first floor

# Main game loop
while True:
    for ev in event.get():
        if ev.type == constants.QUIT:  # Exit if the user closes the window
            exit(0)

    screen.fill((0, 0, 0))  # Clear the screen
    write("Lives: %d" % angel.lives, width - 120, 20)  # Display the number of lives

    # Ask subsequent questions based on the floor level
    if floor == 2 and len(user_answers) == 1:
        user_answers.append(ask_question(questions[1]))

    if floor == 3 and len(user_answers) == 2:
        user_answers.append(ask_question(questions[2]))

    if floor == 4 and len(user_answers) == 3:
        user_answers.append(ask_question(questions[3]))

    if floor == len(vxRanges):  # Check if the player reached the final floor
        if len(user_answers) == 4:
            user_answers.append(ask_question(questions[4]))  # Ask the final question
        score = 100 * floor  # Calculate the score
        correct = sum(1 for i, ans in enumerate(user_answers) if ans == questions[i]["answer"])  # Count correct answers

        # Display winning message and stats
        write("Congratulations, you have won!", width / 2, height / 2 - 60, 1)
        write(f"Name: {name}", width / 2, height / 2 - 20, 1)
        write(f"Matric No: {matric_no}", width / 2, height / 2, 1)
        write(f"Score: {score}", width / 2, height / 2 + 20, 1)
        write(f"Correct Answers: {correct}/{len(questions)}", width / 2, height / 2 + 40, 1)
        write("Press 'N' to start a new game.", width / 2, height / 2 + 60, 1)

        if key.get_pressed()[K_n]:  # Restart the game if 'N' is pressed
            angel.lives = 3
            user_answers = []
            name = get_input("Enter your name:", width // 2, height // 2 - 40)  # Prompt for name again
            matric_no = get_input("Enter your matric number:", width // 2, height // 2)  # Prompt for matric number again
            startFloor(1)  # Reset the game

        display.flip()  # Update the display
        clock.tick(40)  # Control the frame rate
        continue

    write("Floor: %d/%d" % (floor, len(vxRanges)), width - 120, 40)  # Display the current floor
    updateFlakes()  # Update and spawn flakes
    flakes.draw(screen)  # Draw the flakes on the screen

    if angel.lives > 0:
        angel.update()  # Update the angel's position
        screen.blit(angel.image, angel.rect.topleft)  # Draw the angel on the screen
    else:
        score = 100 * (floor - 1)  # Calculate the score based on the floor reached
        correct = sum(1 for i, ans in enumerate(user_answers) if ans == questions[i]["answer"])  # Count correct answers

        # Display game over message and stats
        write("Game Over!", width / 2, height / 2 - 60, 1)
        write(f"Name: {name}", width / 2, height / 2 - 20, 1)
        write(f"Matric No: {matric_no}", width / 2, height / 2, 1)
        write(f"Score: {score}", width / 2, height / 2 + 20, 1)
        write(f"Correct Answers: {correct}/{len(questions)}", width / 2, height / 2 + 40, 1)
        write("Press 'N' to start a new game.", width / 2, height / 2 + 60, 1)

        if key.get_pressed()[K_n]:  # Restart the game if 'N' is pressed
            angel.lives = 3
            user_answers = []
            name = get_input("Enter your name:", width // 2, height // 2 - 40)  # Prompt for name again
            matric_no = get_input("Enter your matric number:", width // 2, height // 2)  # Prompt for matric number again
            startFloor(1)  # Reset the game

        display.flip()  # Update the display
        clock.tick(40)  # Control the frame rate
        continue

    if floor == 1 and angel.lives == 3 and angel.rect.bottom == height:
        # Display instructions on the first floor
        write("You are a little angel, that must go back to the sky.", width / 2, height / 2 - 20, 1)
        write("Use arrow keys to fly while avoiding snowflakes.", width / 2, height / 2, 1)
        write("Reach the next floor at the top of the screen.", width / 2, height / 2 + 20, 1)

    display.flip()  # Update the display
    clock.tick(40)  # Control the frame rate

