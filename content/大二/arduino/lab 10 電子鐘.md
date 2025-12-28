實驗目的： Arduino UNO利用計時器顯示電子鐘於四合一七節顯示器與OLED

上，並且能整合開關調整時間顯示模式。

實驗步驟：

1.  使用四合一七節顯示器來顯示時間。

2.  程式執行後七節顯示器顯示"00:00"，**開關1壓下放開**，顯示“**時時:分分**”，**再壓一次放開顯示** “**分分:秒秒**”。預設一開始顯示“分分:秒秒”。(請自行計時，勿使用time.h的參數，hour(), second()等，自訂變數，在flash中計數，5ms*200=1s，一秒鐘時候，秒的變數自動加1)

3.  請將四合一七節顯示器所顯示的時間於OLED上顯示。

4.  將程式碼PDF 上傳。

5.  將實驗結果用手機拍攝成果影片，手機橫向拍攝，先成果在拍解釋程式碼設計之原理。

6.  影片上傳YouTube 將超連結繳交即可(需要開放給老師看到)。
```java
#include <Wire.h>

#include <Adafruit_GFX.h>

#include <Adafruit_SH1106.h>

#include <FlexiTimer2.h>

  

// 七段顯示器 a~g 對應腳位

const int segPins[7] = {13, 12, 11, 10, 9, 8, 7};

// 四位數位選

const int digitPins[4] = {5, 4, 3, 2};

// 開關腳位

const int buttonPin = 6;

  

// OLED 設定：使用 I2C、RST 設為 -1

Adafruit_SH1106 display(-1);

  

// 七段顯示數字對應

const byte segCode[10] = {

  // gfedcba

  0b00111111, // 0

  0b00000110, // 1

  0b01011011, // 2

  0b01001111, // 3

  0b01100110, // 4

  0b01101101, // 5

  0b01111101, // 6

  0b00000111, // 7

  0b01111111, // 8

  0b01101111  // 9

};

  

int hour = 11, minute = 15, second = 0;      // 時、分、秒的計時變數

int mode = 0; // 0: 分秒, 1: 時分        // 顯示模式（0=分秒，1=時分）

  

bool lastButtonState = HIGH;               // 上一次按鈕的狀態（用於去彈跳）

unsigned long lastDebounceTime = 0;        // 上一次按鈕狀態改變的時間（去彈跳用）

  

unsigned long lastTick = 0;                // 上一次計時器觸發的時間

int tickCount = 0;                         // 計時器累積次數（用於計算一秒）

  

void setup() {

  FlexiTimer2::set(2, scan7SegISR); // 每2ms刷新一位

  FlexiTimer2::start();

  Serial.begin(9600);

  

  // 初始化七段腳位

  for (int i = 0; i < 7; i++) pinMode(segPins[i], OUTPUT);

  for (int i = 0; i < 4; i++) pinMode(digitPins[i], OUTPUT);

  pinMode(buttonPin, INPUT_PULLUP);

  

  // 初始化 OLED

  display.begin(SH1106_SWITCHCAPVCC, 0x3C);

  display.clearDisplay();

  display.display();

  

  // 初始顯示

  showOLED();

}

  

void loop() {

  // 計時，每秒進位

  if (millis() - lastTick >= 5) {

    lastTick = millis();

    tickCount++;

    if (tickCount >= 200) {

      tickCount = 0;

      second++;

      if (second >= 60) {

        second = 0;

        minute++;

        if (minute >= 60) {

          minute = 0;

          hour++;

          if (hour >= 24) hour = 0;

        }

      }

      showOLED();

    }

  }

  

  // 開關偵測（切換模式）

  bool reading = digitalRead(buttonPin);

  

  if (reading != lastButtonState) {

    lastDebounceTime = millis();

  }

  

  if ((millis() - lastDebounceTime) > 50) {

    // 只有在狀態改變時才觸發

    static bool buttonState = HIGH;

    if (reading != buttonState) {

      // 這裡偵測「由 LOW 變 HIGH」（即按鈕放開）

      if (buttonState == LOW && reading == HIGH) {

        mode = 1 - mode;

        Serial.println("按鈕放開，切換模式");

        showOLED();

      }

      buttonState = reading;

    }

  }

  lastButtonState = reading;

}

  

// --- 七段顯示掃描控制 ---

void scan7SegISR() {

  static int currentDigit = 0;

  int nums[4];

  

  if (mode == 0) {

    nums[0] = minute / 10;

    nums[1] = minute % 10;

    nums[2] = second / 10;

    nums[3] = second % 10;

  } else {

    nums[0] = hour / 10;

    nums[1] = hour % 10;

    nums[2] = minute / 10;

    nums[3] = minute % 10;

  }

  

  setDigit(currentDigit, nums[currentDigit]);

  currentDigit = (currentDigit + 1) % 4;

}

  

void setDigit(int digit, int num) {

  for (int i = 0; i < 4; i++) digitalWrite(digitPins[i], i == digit ? LOW : HIGH);

  byte code = segCode[num];

  for (int i = 0; i < 7; i++) digitalWrite(segPins[i], (code >> i) & 0x01);

}

  

void clearDigits() {

  for (int i = 0; i < 4; i++) digitalWrite(digitPins[i], HIGH);

}

  

// --- OLED 顯示更新 ---

void showOLED() {

  display.clearDisplay();

  display.setTextSize(4);

  display.setTextColor(WHITE);

  display.setCursor(0, 0);

  

  char buf[6];

  buf[0] = '0' + (mode == 0 ? minute / 10 : hour / 10);

  buf[1] = '0' + (mode == 0 ? minute % 10 : hour % 10);

  buf[2] = ':';

  buf[3] = '0' + (mode == 0 ? second / 10 : minute / 10);

  buf[4] = '0' + (mode == 0 ? second % 10 : minute % 10);

  buf[5] = '\0';

  

  display.print(buf);

  display.display();

}
```