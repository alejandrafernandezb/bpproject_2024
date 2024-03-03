#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <Wire.h>

#define JOYSTICK_X A1
#define JOYSTICK_Y A2
#define JOYSTICK_BTN 7
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET    -1
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

#define SNAKE_LENGTH 50
#define FOOD_COUNT 5
int SNAKE_SPEED=25;

struct Point {
  int x;
  int y;
};

Point snake[SNAKE_LENGTH];
Point food[FOOD_COUNT];
int snakeLength = 1;
int direction = 1;
int score = 0;

void setup() {
  pinMode(JOYSTICK_X, INPUT);
  pinMode(JOYSTICK_Y, INPUT);
  pinMode(JOYSTICK_BTN, INPUT_PULLUP);
  Serial.begin(9600);
  
  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println(F("SSD1306 allocation failed"));
    for(;;);
  }

  display.clearDisplay();
  randomSeed(analogRead(0));
  resetGame();
}

void loop() {
  display.clearDisplay();
  drawFood();
  moveSnake();
  drawSnake();
  display.display();
  delay(SNAKE_SPEED);
}

void resetGame() {
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(WHITE);
  display.setCursor(0,10);
  display.println("GameOver!");
  display.display();
  Serial.println("end");
  delay(5000);
  score = 0;
  snakeLength = 1;
  snake[0].x = SCREEN_WIDTH / 2;
  snake[0].y = SCREEN_HEIGHT / 2;
  generateFood();
}

void generateFood() {
  for (int i = 0; i < FOOD_COUNT; i++) {
    food[i].x = random(0, SCREEN_WIDTH);
    food[i].y = random(0, SCREEN_HEIGHT);
  }
}

void drawFood() {
  for (int i = 0; i < FOOD_COUNT; i++) {
    display.fillRect(food[i].x, food[i].y, 4, 4, WHITE);
  }
}

void drawSnake() {
  for (int i = 0; i < snakeLength; i++) {
    display.fillRect(snake[i].x, snake[i].y, 4, 4, WHITE);
  }
}

int prev_direction = 1;
//returns direction
int detectJoystick(){
  int x = analogRead(JOYSTICK_X);
  int y = analogRead(JOYSTICK_Y);
  if (x>650){
    //move right
    Serial.println(String(x)+"right");
    return 1;
    //Keyboard.press(97)
  } else if (x<400){
    //move left
    Serial.println(String(x)+"left");
    return 3;
  } else if (y<400){
    //down
    Serial.println(String(y)+"up");
    return 0;
  } else if (y>650){
    //move up
    Serial.println(String(y)+"down");
    return 2;
  } else {
    return prev_direction;
  }
}

void moveSnake() {
  Point next;
  next.x = snake[0].x;
  next.y = snake[0].y;
  
  prev_direction = direction;
  int direction = detectJoystick();
  switch(direction) {
    case 0: next.y--; break; // Up
    case 1: next.x++; break; // Right
    case 2: next.y++; break; // Down
    case 3: next.x--; break; // Left
  }

  if (next.x >= SCREEN_WIDTH || next.x < 0 || next.y >= SCREEN_HEIGHT || next.y < 0) {
    resetGame();
    return;
  }
  
  for (int i = snakeLength - 1; i > 0; i--) {
    snake[i] = snake[i - 1];
    if (snake[i].x == next.x && snake[i].y == next.y) {
      //resetGame();
      return;
    }
  }

  snake[0] = next;

  for (int i = 0; i < FOOD_COUNT; i++) {
    if (next.x == food[i].x && next.y == food[i].y) {
      score += 10;
      snakeLength++;
      SNAKE_SPEED--;
      if (snakeLength >= SNAKE_LENGTH) {
        snakeLength = SNAKE_LENGTH;
      }
      food[i].x = random(0, SCREEN_WIDTH);
      food[i].y = random(0, SCREEN_HEIGHT);
    }
  }
}
