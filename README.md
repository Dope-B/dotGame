# 아두이노용 벽돌깨기 게임

#### 사용 부품
- 8x8 dot matrix
- 택트스위치
- 부저

#### 사용기술: visualMicro

<img width="30%" src="https://user-images.githubusercontent.com/33209821/230914772-debe0e2c-e2f0-4d09-9142-0bd161551320.jpg"/>

#### 설명
- 블럭은 4줄이 생성되고 2차원 배열로 boolean형으로 이뤄진다.
- 게임 클리어 혹은 오버 시 출력 화면도 2차원 배열로 저장했다.
- 버튼으로 최하단 막대를 움직일 수 있고 막대의 위치는 항상 최하단에 있고 길이도 고정되어있으니 막대의 시작점을 기준으로 출력한다.
```C++
    if (pos_bar >= 1) {// 왼쪽 이동 시
			index[7][pos_bar + 2] = 0;
			pos_bar -= 1;
			index[7][pos_bar] = 1;
		}
    
    if (pos_bar <= 4) {// 오른쪽 이동 시
			index[7][pos_bar] = 0;
			pos_bar += 1;
			index[7][pos_bar+2] = 1;
		}
```
- 막대 이동버튼을 누르면 게임이 시작하고 막대의 최초 이동 방향에 따라 공의 최초 이동 방향이 결정된다.
- 공의 이동 방향은 2개의 boolean형 변수로 결정된다.
```C++
boolean left = 1;// 1-> 왼쪽 0-> 오른쪽
boolean up = 1;// 1-> 위쪽 0-> 아래쪽
```
- 공의 충돌 판정은 다음과 같다(ball_col_check())
  - 벽에 부딪혔을 시 x축 방향이 바뀐다.
    - 바로 위에 블럭이 있다면 그 블럭을 지우고 x축, y축 이동방향이 바뀐다.
  - 이동방향에 블럭이 있다면 그 블럭을 지우고 x축 방향은 랜덤하게 결정되고 y축 이동방향은 바뀐다.
- 일정 시간마다 게임의 프레임 업데이트 주기가 빨라지고 상한선은 80ms이다.
```C++
      grade = millis();// grade, curT 둘 중 하나는 필요X
			curT = millis();
			if (grade % 200 == 0) {
				if (tempo > 100) { tempo -= 40; }// 업데이트 주기 빨라짐
				else if (tempo <= 100) { tempo = 80; }// 상한선
			}
			if (curT - preT > tempo) {
				preT = curT;
				ball_col_check();// 업데이트 내용
				ball_move_check();
			}
```

#### 피드백
- 업데이트 주기가 빨라지는 조건을 시간이 아니라 없어진 블럭의 갯수로 하면 더 좋았을 듯 하다.
- 공 충돌판정 알고리즘을 방향에 따른 이동 방향을 변수로 따로 설정했더라면 보기 좋게 구현 가능 했을 듯 하다.
