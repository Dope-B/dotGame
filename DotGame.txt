#include <LedControl.h>
LedControl lc = LedControl(12, 11, 10, 1);
boolean index[8][8] = {  {0,0,0,0,0,0,0,0},
						 {0,0,0,0,0,0,0,0}, 
						 {0,0,0,0,0,0,0,0}, 
						 {0,0,0,0,0,0,0,0}, 
						 {0,0,0,0,0,0,0,0}, 
						 {0,0,0,0,0,0,0,0}, 
						 {0,0,0,0,0,0,0,0}, 
						 {0,0,0,0,0,0,0,0}};
boolean block[4][8] = { {1,1,1,1,1,1,1,1},
						{1,1,1,1,1,1,1,1},
						{1,1,1,1,1,1,1,1},
						{1,1,1,1,1,1,1,1} };
boolean smile[8][8] = { {0,0,0,0,0,0,0,0},
						{0,0,0,0,0,0,0,0},
						{0,0,1,0,0,1,0,0},
						{0,0,1,0,0,1,0,0},
						{0,0,1,0,0,1,0,0},
						{0,0,0,0,0,0,0,0},
						{0,1,0,0,0,0,1,0},
						{0,0,1,1,1,1,0,0} };
boolean smile2[8][8] = {{0,0,0,0,0,0,0,0},
						{0,0,0,0,0,0,0,0},
						{0,0,1,0,0,0,1,0},
						{0,0,1,0,0,1,0,0},
						{0,0,1,0,0,0,1,0},
						{0,0,0,0,0,0,0,0},
						{0,1,0,0,0,0,1,0},
						{0,0,1,1,1,1,0,0}};
boolean sad[8][8] = {{0,0,0,0,0,0,0,0},
					 {0,0,0,0,0,0,0,0},
					 {0,0,0,0,0,0,0,0},
				 	 {0,1,1,0,0,1,1,0},
				 	 {0,0,0,0,0,0,0,0},
				 	 {0,0,0,0,0,0,0,0},
					 {0,0,1,1,1,1,0,0},
					 {0,1,0,0,0,0,1,0}};
boolean sad2[8][8] = {{0,0,0,0,0,0,0,0},
					  {0,0,0,0,0,0,0,0},
					  {0,0,0,0,0,0,0,0},
					  {0,1,1,0,0,1,1,0},
					  {0,1,0,0,0,0,1,0},
					  {0,0,0,0,0,0,0,0},
					  {0,0,1,1,1,1,0,0},
					  {0,1,0,0,0,0,1,0}};
boolean left = 1;
boolean up = 1;
boolean isOver = 0;
boolean isClear = 0;
boolean restart = 0;
boolean start = 0;
boolean isBounce = 0;
boolean LBP = 0;//left button pressed
boolean RBP = 0;//right button pressed
boolean icon = 0;
int var = 0;
int pos_bar = 0;
int pos_ball_x = 0;
int pos_ball_y = 0;
int tempo=700;
unsigned long curT = 0;
unsigned long preT = 0;
unsigned long curT2 = 0;//for restart
unsigned long preT2 = 0;
unsigned int grade = 0;
void setup()
{
	pinMode(9, INPUT);//LEFT Button
	pinMode(8, INPUT);//RIGHT Button
	pinMode(7, OUTPUT);//speaker
	lc.shutdown(0, false);
	lc.setIntensity(0, 8);
	lc.clearDisplay(0);
	setStage();
	Serial.begin(9600);
}

// Add the main program code into the continuous loop() function
void loop()
{
	if (!isOver && !isClear) {
		if (start) {
			grade = millis();
			curT = millis();
			if (grade % 200 == 0) {
				if (tempo > 100) { tempo -= 40; }
				else if (tempo <= 100) { tempo = 80; }
			}
			if (curT - preT > tempo) {
				preT = curT;
				ball_col_check();
				ball_move_check();
			}
		}
		else {
			start_check();
		}
		button_check();
		clear_check();
		print();
	}
	if (isOver) { over(); }
	else if (isClear) { clear(); }
}
void start_check() {
	if (digitalRead(9) == 1) {
		start = true;
		left = 1;
	}
	else if (digitalRead(8) == 1) {
		left = 0;
		start = true;
	}
}
void restart_check() {
	if (!restart) {
		if (digitalRead(9) == 1|| digitalRead(8) == 1) {
			restart = true;
			isOver = false;
			isClear = false;
			for (int i = 0; i < 8; i++) {
				for (int j = 0; j < 8; j++) {
					index[i][j] = 0;
				}
			}
		}
	}
}
void button_check() {
	if (!LBP&&digitalRead(9) == 1) {
		LBP = true;
		if (pos_bar >= 1) {
			index[7][pos_bar + 2] = 0;
			pos_bar -= 1;
			index[7][pos_bar] = 1;
		}
	}
	else if (!RBP&&digitalRead(8) == 1) {
		RBP = true;
		if (pos_bar <= 4) {
			index[7][pos_bar] = 0;
			pos_bar += 1;
			index[7][pos_bar+2] = 1;
		}
	}
	if (digitalRead(9) == 0) {
		LBP = false;
	}
	if (digitalRead(8) == 0) {
		RBP = false;
	}
}
void setStage() {
	for (int i = 0; i < 4; i++) {
		for (int j = 0; j < 8; j++) {
			index[i][j] = 1;
		}
	}


	pos_bar = 2;
	index[7][pos_bar] = 1;
	index[7][pos_bar+1] = 1;
	index[7][pos_bar+2] = 1;

	pos_ball_x = pos_bar + 1;
	pos_ball_y = 6;
	index[pos_ball_y][pos_ball_x] = 1;
	tempo = 700;
	restart = false;
}
void ball_col_check() {
	switch (up) {
	case 1:
		isBounce = false;
		if (left) {
			if (pos_ball_x == 0) {//wall
				if (index[pos_ball_y - 1][pos_ball_x] == 0) {
					if (index[pos_ball_y - 1][pos_ball_x + 1] == 0) {
						left = 0;
					}
					else {
						index[pos_ball_y - 1][pos_ball_x + 1] = 0;
						tone(7, 262, 10);
						up = 0;
						left = 0;
					}
				}
				else {
					index[pos_ball_y - 1][pos_ball_x] = 0;
					tone(7, 262, 10);
					up = 0;
					left = 0;
				}
			}
			else if (index[pos_ball_y-1][pos_ball_x] == 1) {//block
				up = 0;
				index[pos_ball_y-1][pos_ball_x] = 0;
				tone(7, 262, 10);
			}
			else if (index[pos_ball_y - 1][pos_ball_x] == 0 && index[pos_ball_y][pos_ball_x - 1] == 1) {
				left = 0;
			}
			else if (index[pos_ball_y - 1][pos_ball_x] == 0 && index[pos_ball_y][pos_ball_x - 1] == 0&& index[pos_ball_y - 1][pos_ball_x-1] == 1) {
				index[pos_ball_y - 1][pos_ball_x-1] = 0;
				tone(7, 262, 10);
				up = 0;
				var = random(2);
				if (var == 0) {
					left = 0;
				}
				else {
					left = 1;
				}
			}
			if (pos_ball_y == 0) {
				if (index[pos_ball_y + 1][pos_ball_x - 1] == 1) { up = 0; left = 0; index[pos_ball_y + 1][pos_ball_x - 1] = 0; }
				else { up = 0; }
			}
		}
		else {
			if (pos_ball_x == 7) {//wall
				if (index[pos_ball_y - 1][pos_ball_x] == 0) {
					if (index[pos_ball_y - 1][pos_ball_x - 1] == 0) {
						left = 1;
					}
					else {
						index[pos_ball_y - 1][pos_ball_x - 1] = 0;
						tone(7, 262, 10);
						up = 0;
						left = 1;
					}
				}
				else {
					index[pos_ball_y - 1][pos_ball_x] = 0;
					tone(7, 262, 10);
					up = 0;
					left = 1;
				}
			}
			else if (index[pos_ball_y-1][pos_ball_x] == 1) {//block
				up = 0;
				index[pos_ball_y-1][pos_ball_x] = 0;
				tone(7, 262, 10);
			}
			else if (index[pos_ball_y - 1][pos_ball_x] == 0&& index[pos_ball_y][pos_ball_x+1]==1) {
				left = 1;
			}
			else if (index[pos_ball_y - 1][pos_ball_x] == 0 && index[pos_ball_y][pos_ball_x + 1] == 0&& index[pos_ball_y - 1][pos_ball_x+1] == 1) {
				index[pos_ball_y-1][pos_ball_x+1] = 0;
				tone(7, 262, 10);
				up = 0;
				var = random(2);
				if (var == 0) {
					left = 0;
				}
				else {
					left = 1;
				}
			}
			if (pos_ball_y == 0) {
				if (index[pos_ball_y + 1][pos_ball_x + 1] == 1) { up = 0; left = 1; index[pos_ball_y + 1][pos_ball_x + 1] = 0; }
				else { up = 0; }
			}
		}
		break;
	default:
		if (pos_ball_y == 7) {
			Serial.println("end");
			isOver = true;
		}
		if (left) {
			if (pos_ball_x == 0) {//wall
				if (index[pos_ball_y + 1][pos_ball_x] == 1 && !isBounce&&pos_ball_y==6) {
					isBounce = true;
					up = 1;
					tone(7, 262, 10);
				}
				else if (index[pos_ball_y + 1][pos_ball_x] == 1 && pos_ball_y == 6) {
					up = 1;
					tone(7, 262, 10);
					index[pos_ball_y + 1][pos_ball_x] == 0;
				}
				left = 0;
			}
			else if (index[pos_ball_y + 1][pos_ball_x] == 1 && pos_ball_y != 6) {
				up = 1;
				tone(7, 262, 10);
				index[pos_ball_y + 1][pos_ball_x] == 0;
			}
			else if (index[pos_ball_y + 1][pos_ball_x] == 0 && index[pos_ball_y + 1][pos_ball_x - 1] == 1 && pos_ball_y != 6) {
				left = 0;
				tone(7, 262, 10);
				index[pos_ball_y + 1][pos_ball_x-1] == 0;
			}
			else if (index[pos_ball_y+1][pos_ball_x] == 1&&!isBounce&&pos_ball_y == 6) {//bar
				isBounce = true;
				up = 1;
				tone(7, 262, 10);
			}
		}
		else {
			if (pos_ball_x == 7) {//wall
				if (index[pos_ball_y + 1][pos_ball_x] == 1 && !isBounce&&pos_ball_y==6) {
					isBounce = true;
					up = 1;
					tone(7, 262, 10);
				}
				else if (index[pos_ball_y + 1][pos_ball_x] == 1&&pos_ball_y==6) {
					up = 1;
					tone(7, 262, 10);
					index[pos_ball_y + 1][pos_ball_x] == 0;
				}
				left = 1;
			}
			else if (index[pos_ball_y + 1][pos_ball_x] == 1 && pos_ball_y != 6) {
				up = 1;
				tone(7, 262, 10);
				index[pos_ball_y + 1][pos_ball_x] == 0;
			}
			else if (index[pos_ball_y + 1][pos_ball_x] == 0 && index[pos_ball_y + 1][pos_ball_x + 1] == 1&& pos_ball_y != 6) {
				left = 1;
				tone(7, 262, 10);
				index[pos_ball_y + 1][pos_ball_x + 1] == 0;
			}
			else if (index[pos_ball_y+1][pos_ball_x] == 1&&!isBounce&&pos_ball_y==6) {//bar
				isBounce = true;
				up = 1;
				tone(7, 262, 10);
			}
		}
		break;
	}
}
void ball_move_check() {
	switch (up) {
	case 1:
		if (left) {
			index[pos_ball_y][pos_ball_x] = 0;
			pos_ball_x -= 1;
			pos_ball_y -= 1;
			index[pos_ball_y][pos_ball_x] = 1;
		}
		else {
			index[pos_ball_y][pos_ball_x] = 0;
			pos_ball_x += 1;
			pos_ball_y -= 1;
			index[pos_ball_y][pos_ball_x] = 1;
		}
		break;
	default:
		if (left) {
			index[pos_ball_y][pos_ball_x] = 0;
			pos_ball_x -= 1;
			pos_ball_y += 1;
			index[pos_ball_y][pos_ball_x] = 1;
		}
		else {
			index[pos_ball_y][pos_ball_x] = 0;
			pos_ball_x += 1;
			pos_ball_y += 1;
			index[pos_ball_y][pos_ball_x] = 1;
		}
		break;
	}
}
void print() {
	for (int i = 0; i < 8; i++) {
		for (int j = 0; j < 8; j++) {
			lc.setLed(0, i, j, index[i][j]);
		}
	}
}
void clear_check() {
	for (int i = 0; i < 4; i++) {
		for (int j = 0; j < 8; j++) {
			if (index[i][j] == 1) {
				return;
			}
		}
		if (i == 3) {
			isClear = true;
		}
	}
}
void clear() {
		for (int i = 0; i < 8; i++) {
			for (int j = 0; j < 8; j++) {
				lc.setLed(0, i, j, 1);
			}
			delay(50);
		}
		for (int i = 0; i < 8; i++) {
			for (int j = 0; j < 8; j++) {
				lc.setLed(0, i, j, 0);
			}
			delay(50);
		}
		start = 0;
		while (!restart) {
			curT2 = millis();
			if (curT2 - preT2 > 700) {
				preT2 = curT2;
				if (icon == 0) {
					for (int i = 0; i < 8; i++) {
						for (int j = 0; j < 8; j++) {
							lc.setLed(0, i, j, smile[i][j]);
						}
					}
					icon = 1;
				}
				else {
					for (int i = 0; i < 8; i++) {
						for (int j = 0; j < 8; j++) {
							lc.setLed(0, i, j, smile2[i][j]);
						}
					}
					icon = 0;
				}
			}
			restart_check();
		}
}
void over() {
		for (int i = 0; i < 8; i++) {
			for (int j = 0; j < 8; j++) {
				lc.setLed(0, i, j, 1);
			}
			delay(50);
		}
		for (int i = 0; i < 8; i++) {
			for (int j = 0; j < 8; j++) {
				lc.setLed(0, i, j, 0);
			}
			delay(50);
		}
		start = 0;
		while (!restart) {
			curT2 = millis();
			if (curT2 - preT2 > 700) {
				preT2 = curT2;
				if (icon == 0) {
					for (int i = 0; i < 8; i++) {
						for (int j = 0; j < 8; j++) {
							lc.setLed(0, i, j, sad[i][j]);
						}
					}
					icon = 1;
				}
				else {
					for (int i = 0; i < 8; i++) {
						for (int j = 0; j < 8; j++) {
							lc.setLed(0, i, j, sad2[i][j]);
						}
					}
					icon = 0;
				}
			}
			restart_check();
	    }
		setStage();
		delay(200);
}