#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <string.h>

#ifdef _WIN32
#include <conio.h>
#include <windows.h>
#else
#include <unistd.h>
#include <termios.h>
#endif

#define LENGTH 20 // Width of the game board
#define BREADTH 20 // Height of the game board
#define SNAKE_MAX_LENGTH 20 // Maximum length of the snake
int move =1;
int level = 1;
int playy = 0;
char playagain;
int k = 1, r = 0, l = 0, u = 0, d = 0;
int snakeX[SNAKE_MAX_LENGTH], snakeY[SNAKE_MAX_LENGTH]; // Snake positions
int fruitX, fruitY; // Fruit position
int snakeLength = 1; // Initial length of the snake
int score = 0; // Score
int gameover = 0; // Game over flag
char direction = 'd'; // Initial direction of the snake
char username[20];

void fruiteat() {
    int dd, yy;
    do {
        dd = rand() % (LENGTH - 2) + 1; // Generate a random position within the game board, excluding the borders
        yy = rand() % (BREADTH - 2) + 1;
    } while (dd == 0 || yy == 0);

    fruitX = dd;
    fruitY = yy;
}

void setup() {
    srand(time(NULL)); // Seed for randomization
    snakeX[0] = LENGTH / 2; // Initial snake position
    snakeY[0] = BREADTH / 2;
     // Generate initial fruit position
   fruiteat();
}

void draw() {
    system("cls || clear"); // Clear screen
    printf("Score: %d\n", score);
    printf("Level %d\n",level);
    printf("Use 'w', 'a', 's', 'd' or 'W', 'A', 'S', 'D' to move the snake. Press 'x' to quit.\n");
if(move == 1)
{
     snakeX[0] = 10; // Initial snake position
    snakeY[0] = 10;
}
    // Loop through the game board
    for (int i = 0; i < BREADTH; i++) {
        for (int j = 0; j < LENGTH; j++) {
            if (i == 0 || i == BREADTH - 1)
                printf("-");
            else if (j == 0 || j == LENGTH - 1)
                printf("|");
            else if (i == fruitY && j == fruitX)
                printf("*"); // Draw fruit
            else {
                int flag = 0;
                for (int k = 0; k < snakeLength; k++) {
                    if (snakeX[k] == j && snakeY[k] == i) {
                        if(move==1)
                        {
                         printf("^");
                         move=0;
                        }
                        if (k == 0) {
                            if (r == 1)
                                printf(">");
                            else if (l == 1)
                                printf("<");
                            else if (u == 1)
                                printf("^");
                            else if (d == 1)
                                printf("v");
                        } else if (level > 1) {
                            printf("x"); // Draw snake body
                        }
                        flag = 1;
                    }
                }
                if (flag == 0)
                    printf(" "); // Draw empty space
            }
        }
        printf("\n");
    }
}

void input() {
#ifdef _WIN32
    if (_kbhit()) {
        char ch = _getch();
        switch (ch) {
#else
    struct termios oldt, newt;
    tcgetattr(STDIN_FILENO, &oldt);
    newt = oldt;
    newt.c_lflag &= ~(ICANON | ECHO);
    tcsetattr(STDIN_FILENO, TCSANOW, &newt);
    int ch = getchar();
    tcsetattr(STDIN_FILENO, TCSANOW, &oldt);
#endif

    switch (ch) {
        case 'w':
        case 'W':
            if (direction != 's') {
                direction = 'w';
                u = 1;
                k = 0;
                r = l = d = 0;
            }
            break;
        case 'a':
        case 'A':
            if (direction != 'd') {
                direction = 'a';
                l = 1;
                k = 0;
                r = u = d = 0;
            }
            break;
        case 's':
        case 'S':
            if (direction != 'w') {
                direction = 's';
                d = 1;
                k = 0;
                r = l = u = 0;
            }
            break;
        case 'd':
        case 'D':
            if (direction != 'a') {
                direction = 'd';
                r = 1;
                k = 0;
                l = u = d = 0;
            }
            break;
        case 'x':
            gameover = 1;
            break;
        }
    }

void run() {
    int prevX = snakeX[0]; // Store the previous position of the head
    int prevY = snakeY[0];

    // Move the head of the snake
    switch (direction) {
        case 'w':
            if (snakeY[0] == 0) // If snake reaches top boundary
                snakeY[0] = BREADTH - 2; // Set y-coordinate to bottom boundary
            else
                snakeY[0]--;
            break;
        case 'a':
            if (snakeX[0] == 0) // If snake reaches left boundary
                snakeX[0] = LENGTH - 2; // Set x-coordinate to right boundary
            else
                snakeX[0]--;
            break;
        case 's':
            if (snakeY[0] == BREADTH - 2) // If snake reaches bottom boundary
                snakeY[0] = 0; // Set y-coordinate to top boundary
            else
                snakeY[0]++;
            break;
        case 'd':
            if (snakeX[0] == LENGTH - 2) // If snake reaches right boundary
                snakeX[0] = 0; // Set x-coordinate to left boundary
            else
                snakeX[0]++;
            break;
    }

    // Move the body segments
    for (int i = 1; i < snakeLength; i++) {
        int tempX = snakeX[i];
        int tempY = snakeY[i];
        snakeX[i] = prevX;
        snakeY[i] = prevY;
        prevX = tempX;
        prevY = tempY;
    }

   if (level >= 2) {
        // Check collision with boundaries
        if (snakeX[0] == 0 || snakeX[0] == LENGTH - 1 || snakeY[0] == 0 || snakeY[0] == BREADTH - 1)
            gameover = 1;
    }

    if (level >= 2) {
        for (int i = 1; i < snakeLength; i++) {
            if (snakeX[i] == snakeX[0] && snakeY[i] == snakeY[0])
                gameover = 1;
        }
    }

    // Check if fruit is eaten
    if (snakeX[0] == fruitX && snakeY[0] == fruitY) {
        // for score increment as per level
        if (level == 1) {
            score++;
        } else if (level == 2) {
            score += 5;
        } else if (level == 3) {
            score += 10;
        }
        // for game over in level 1 
        if (level == 1 && score == 10) {
            gameover = 1;
        } else // GAME OVER FOR LEVEL 2
        if (level == 2 && score == 50) {
            gameover = 1;
        }

         // Increase snake length by one and add new body segment
        if (level > 1) {
            if (snakeLength < SNAKE_MAX_LENGTH) {
                snakeLength++;
                snakeX[snakeLength - 1] = prevX; // Add new body segment at previous tail position
                snakeY[snakeLength - 1] = prevY;
            }
        }


   fruiteat();
}
}

void delay(unsigned int milliseconds) {
#ifdef _WIN32
    Sleep(milliseconds);
#else
    usleep(milliseconds * 1000);
#endif
}

void play() {
    setup(); // Initialize the game
    while (!gameover) {
        draw(); // Draw the game board
        input(); // Handle user input
        run();   // Update the game state
        delay(400); // Control speed of snake (400 milliseconds)
        
        if (level == 1 && score == 10) {
            printf("Congratulations %s, Level 1 is Complete. New Feature unlocked for Level 2\n", username);
            printf("Your score: %d\n", score);
            level++;
            gameover = 0;
            playy = 0;
            break;
        }
        else
        if (level == 2 && score == 50) {
            printf("Congratulations %s, Level %d is Complete. New Feature unlocked for Level %d\n", username, level, level + 1);
            printf("Your score: %d\n", score);
            level++;
            gameover = 0;
            playy = 0;
            break;
        }
        if(level==3&& score==150)
        {
            gameover =0 ;
            break;
        }
    }
}

int main() {
    printf("Welcome to Snake Game Level %d\n", level);
    printf("Enter your username: ");
    scanf("%s", username);
    getchar(); // Consume the newline character left in the input buffer

    play();

    if (level == 2){
        printf("Rules For Level %d are :\n", level);
        printf("1. Boundary collision start\n2. Score is Increased by +5\n3. Max score 50\n");
        printf("** If You read all rules press 1 :");
        scanf("%d", &playy);
        if (playy == 1) {
            gameover = 0;
            play();
        }
    }
  
  
   if (level == 3) {
        printf("Rules For Level %d are :\n", level);
        printf("1. Boundary collision start\n2. Score is Increased by +10\n3. Max score 100\n4. Self-collision start\n");
        printf("** If You read all rules press 1 :");
        scanf("%d", &playy);
        if (playy == 1) {
            gameover = 0;
            play();
        }
    }
 printf("Congratulations %s, You Complete the all 3 level of game\n", username);
    printf("Game Over! Your score: %d\n", score);
    return 0;
}