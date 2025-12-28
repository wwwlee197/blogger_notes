實驗目的： Arduino UNO 連接人體紅外線感測器、繼電器、LED、光敏電阻、七段顯示器，伺服馬達，達成人流計算之功能。

實驗步驟：

1.  先將Arduino連接人體紅外線，由於人體紅外線受到熱源影響，所以盡量遮蔽附近的熱源，測試人手揮動時候是否會有感應到，可以透過串列傳輸列印出結果測試人體紅外線。

2.  將此將人體紅外線當成入口處，有人通過時候室內人數之數值會加一，也LED1(將透過繼電器控制)會閃爍1秒鐘，室內人數之數值將透過串列印出來，沒有人通過就不要印出來。

3.  利用光敏電阻當成另外一個類似人體紅外線感測器(因只有一個人體紅外線感測器)，放在出口處，人出去時候將手放光敏電阻上，造成數值變化，讓LED2(將透過繼電器控制)會閃爍1秒鐘，室內人數之數值會減一，該數值將透過串列印出來，沒有人通過就不要印出來。

4.  室內人數之數值預設為0，室內人數之數值會依照目前人數一直在七段顯示器上顯示(假設小於10人，故0~9)。

5.  為能確保了解繼電器之控制， 兩顆LED 1 和 LED 2將透過繼電器導通，亮一秒鐘。

6.  人數增加時候，控制伺服馬達轉90度。

7.  人數減少時候，控制伺服馬達轉135度。

8.  相關接線如投影片。NO1、COM1為第一組LED 1，NO2、COM2為第二組LED 2。可以發現繼電器之聲音變化了解動作。

9.  上傳程式碼PDF檔案。

10.將實驗結果用手機拍攝成果影片，手機橫向拍攝，先拍成果再解釋程式設計之原理。影片上傳YouTube 將連結繳交即可

```cpp
#include <Servo.h>

  

// 七段顯示器 a~g 對應 pin 2~8

const int segPins[7] = {2, 3, 4, 5, 6, 7, 8};

  

// 七段顯示器數字編碼（共陰極）

const byte digits[10][7] = {

  {1,1,1,1,1,1,0}, // 0

  {0,1,1,0,0,0,0}, // 1

  {1,1,0,1,1,0,1}, // 2

  {1,1,1,1,0,0,1}, // 3

  {0,1,1,0,0,1,1}, // 4

  {1,0,1,1,0,1,1}, // 5

  {1,0,1,1,1,1,1}, // 6

  {1,1,1,0,0,0,0}, // 7

  {1,1,1,1,1,1,1}, // 8

  {1,1,1,1,0,1,1}  // 9

};

  

// 顯示字母 F（Full）

const byte fullDigit[7] = {1, 0, 0, 1, 1, 1, 1};

  

const int pirPin = 11;

const int ldrPin = A0;

const int relay1Pin = 9;  // 進入 LED

const int relay2Pin = 10;  // 離開 LED

const int servoPin = 13;

  

int peopleCount = 0;

Servo myServo;

  

bool pirTriggered = false;

bool ldrTriggered = false;

  

unsigned long lastPrintTime = 0;  // 每秒印光敏電阻用

  

void setup() {

  Serial.begin(9600);

  

  // 設定七段顯示器接腳為輸出

  for (int i = 0; i < 7; i++) {

    pinMode(segPins[i], OUTPUT);

  }

  

  pinMode(pirPin, INPUT);

  pinMode(relay1Pin, OUTPUT);

  pinMode(relay2Pin, OUTPUT);

  

  myServo.attach(servoPin);

  myServo.write(0); // 初始角度

}

  

void loop() {

  int pirState = digitalRead(pirPin); //人體紅外線感測器

  int ldrValue = analogRead(ldrPin);   //光敏電阻

  

  // ======= 進入檢測 (PIR) =======

  if (pirState == HIGH && !pirTriggered) {

    pirTriggered = true;

  

    if (peopleCount < 9) {

      peopleCount++;

      Serial.print("有人進入，室內人數：");

      Serial.println(peopleCount);

  

      digitalWrite(relay1Pin, HIGH);

      delay(1000);

      digitalWrite(relay1Pin, LOW);

  

      myServo.write(90); delay(1000); myServo.write(0);

    } else {

      Serial.println("人數已滿");

    }

  }

  

  if (pirState == LOW) {

    pirTriggered = false;  // 重設 PIR 觸發狀態

  }

  

  // ======= 離開檢測 (光敏電阻) =======

  if (ldrValue > 990 && !ldrTriggered && peopleCount > 0) {

    ldrTriggered = true;

  

    peopleCount--;

    Serial.print("有人離開，室內人數：");

    Serial.println(peopleCount);

  

    digitalWrite(relay2Pin, HIGH);

    delay(1000);

    digitalWrite(relay2Pin, LOW);

  

    myServo.write(135); delay(1000); myServo.write(0);

  }

  

  if (ldrValue > 500) {  // 若光線恢復正常，則重設

    ldrTriggered = false;

  }

  

  // ======= 顯示人數或 "滿" =======

  if (peopleCount <= 9) {

    showDigit(peopleCount);

  } else {

    showFull();  // 顯示 "F"

  }

  

  // ======= 每秒顯示光敏電阻數值 =======

  if (millis() - lastPrintTime >= 1000) {

    Serial.print("光敏電阻數值：");

    Serial.println(ldrValue);

    lastPrintTime = millis();

  }

  

  delay(100);  // 稍作延遲

}

  

// 顯示數字

void showDigit(int num) {

  for (int i = 0; i < 7; i++) {

    digitalWrite(segPins[i], digits[num][i]);

  }

}

  

// 顯示字母 F（人數已滿）

void showFull() {

  for (int i = 0; i < 7; i++) {

    digitalWrite(segPins[i], fullDigit[i]);

  }

}
```