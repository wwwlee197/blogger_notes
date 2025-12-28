
Lab 9 超音波感測器與倒車感測設計

實驗目的： Arduino UNO與超音波感測器，將測距結果比較其精準度，並顯示與PC上。

實驗步驟：

1.  先將Arduino連接一個超音波感測器與一個喇叭。讀取超音波與障礙物之距離，並將距離顯示在串列serial monitor中。

2.  首先，輸入4筆數字，分別為倒車雷達警示距離的範圍，之後依照距離完成倒車雷達之功能。例如: 輸入 80, 30, 20, 5 分別表示倒車雷達警示在此範圍內將使喇叭發出不同之警示(參考課本範例)。4筆數值請顯判斷式子，確保老師檢查可能先輸入大到小或是小到大不同情況。

3.  將程式碼PDF 上傳。

4.  再將實驗結果用手機拍攝成果影片，拍攝成果影片，請測試兩組不同數字之結果。手機橫向拍攝，先成果在拍解釋程式碼設計之原理。

5.  影片上傳YouTube 將超連結繳交即可(需要開放給老師看到)。

```css
const byte TrigPin = 6;

const byte EchoPin = 7;

const byte BuzzerPin = 9;

long duration;

long cm;

  

int levels[4] = {5, 20, 30, 80}; // 預設區間

  

void setup() {

  Serial.begin(9600);

  pinMode(TrigPin, OUTPUT);

  pinMode(EchoPin, INPUT);

  pinMode(BuzzerPin, OUTPUT);

  Serial.println("請輸入四組距離區間(以空格分隔)，  例如：5 20 30 80");

}

  

void loop() {

  // 檢查是否有新的 Serial 輸入

  if (Serial.available()) {

    String input = Serial.readStringUntil('\n');

    input.trim();

    if (input.indexOf(' ') > 0) {

      // 解析四組距離區間（以空格分隔）

      int lastIndex = 0;

      for (int i = 0; i < 4; i++) {

        int idx = input.indexOf(' ', lastIndex);

        String numStr;

        if (i < 3) {

          numStr = input.substring(lastIndex, idx);

          lastIndex = idx + 1;

        } else {

          numStr = input.substring(lastIndex);

        }

        numStr.trim();

        levels[i] = numStr.toInt();

      }

      // 由小到大排序

      for (int i = 0; i < 3; i++) {

        for (int j = i + 1; j < 4; j++) {

          if (levels[i] > levels[j]) {

            int tmp = levels[i];

            levels[i] = levels[j];

            levels[j] = tmp;

          }

        }

      }

      // 列印

      Serial.print("已設定區間（由小到大）：");

      for (int i = 0; i < 4; i++) {

        Serial.print(levels[i]);

        if (i < 3) Serial.print(" ");

      }

      delay(2000);

      Serial.println();

    }

  }

  

  // 超音波測距

  digitalWrite(TrigPin, LOW);

  delayMicroseconds(2);

  digitalWrite(TrigPin, HIGH);

  delayMicroseconds(10);

  digitalWrite(TrigPin, LOW);

  delayMicroseconds(2);

  duration = pulseIn(EchoPin, HIGH);

  cm = duration / 58;

  

  Serial.print("目前距離：");

  Serial.print(cm);

  Serial.println(" 公分");

  

  // 由小到大區間判斷蜂鳴器

  if (cm <= levels[0]) {

    tone(BuzzerPin, 400);

  } else if (cm <= levels[1]) {

    tone(BuzzerPin, 400, 100);

    delay(500);

  } else if (cm <= levels[2]) {

    tone(BuzzerPin, 400, 100);

    delay(1000);

  } else if (cm <= levels[3]) {

    tone(BuzzerPin, 400, 100);

    delay(1500);

  } else {

    // tone(BuzzerPin, 400, 100);

    delay(500);

  }

}
```