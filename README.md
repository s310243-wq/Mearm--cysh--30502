# Mearm--cysh--30502
四連桿 MeArm 機構四軸機械手臂開發專案 (4-Axis MeArm Robot Arm)本專案採用 Arduino UNO 開發板、小型伺服馬達（SG90/MG90S）及雙軸搖桿模組，開發一台基於 四連桿 MeArm 機構 的四軸機械手臂。專案從問題定義、研究規劃、設計實作、測試評估到反思優化，完整驗證了 MeArm 機構的簡潔高效性與 Arduino 平台的控制能力。📌 專案亮點 (Features)四連桿機構（Four-bar linkage）：採用平行四邊形連桿設計，當馬達轉動時能保持末端夾爪姿態相對穩定，簡化運動學計算。直觀搖桿控制：透過雙軸搖桿（KY-023/PS2 模組）對機械手臂進行即時的二維空間移動與角度控制。低成本與高擴展性：結合 3D 列印（PLA 材質）與開源硬體，提供高客製化彈性與低門檻的自動化示範平台。獨立電源管理：採用外部 5V 2A 獨立供電，解決伺服馬達供電不足與抖動問題。🛠️ 硬體規格與需求 (Hardware Specification)元件名稱型號/規格說明 / 選用原因主控板Arduino UNO R3穩定性高、教學資源豐富、適合原型開發伺服馬達 (x4)SG90 / MG90SSG90 輕量經濟；MG90S 具金屬齒輪更耐用雙軸搖桿模組KY-023 / PS2 搖桿提供 X, Y 類比訊號與按鈕數位訊號機械手臂骨架3D 列印 (PLA) / 壓克力採用 MeArm 開源設計並進行尺寸校準電源供應5V 2A 獨立電源確保 4 顆伺服馬達穩定供電，避免 Arduino 板載穩壓器過載連接線材杜邦線、M3 螺絲標準化組裝線材與配件🔌 電路接線說明 (Wiring Diagram)1. 伺服馬達 (Servos)訊號線 (PWM)：基座 (Yaw) $\rightarrow$ Arduino D9下臂/肩膀 (Shoulder) $\rightarrow$ Arduino D10上臂/手肘 (Elbow) $\rightarrow$ Arduino D11夾爪 (Gripper) $\rightarrow$ Arduino D12電源線：所有伺服馬達 VCC 接外部 5V 電源正極，GND 接外部電源負極。2. 雙軸搖桿 (Joystick)VRX (X 軸) $\rightarrow$ Arduino A0VRY (Y 軸) $\rightarrow$ Arduino A1SW (按鈕) $\rightarrow$ Arduino D2VCC / GND $\rightarrow$ Arduino 5V / GND⚠️ 注意事項：請務必將外部電源的 GND 與 Arduino 的 GND 相連（共地），以確保訊號傳輸正常。💻 程式碼簡介 (Code Example)專案採用 Arduino C++ 編寫，透過 <Servo.h> 函式庫控制伺服馬達，並將搖桿的類比讀值映射至各軸馬達角度。C++#include <Servo.h>

Servo baseServo;    // 基座 (Yaw)
Servo shoulderServo; // 肩膀 (下臂)
Servo elbowServo;    // 手肘 (上臂)
Servo gripperServo;  // 夾爪

int joyX = A0;      // 搖桿 X 軸
int joyY = A1;      // 搖桿 Y 軸
int joySW = 2;      // 搖桿按鈕 (用於夾爪開合)

int baseAngle = 90;
int shoulderAngle = 90;
int elbowAngle = 90;
int gripperAngle = 90; // 預設夾爪半開

void setup() {
  baseServo.attach(9);
  shoulderServo.attach(10);
  elbowServo.attach(11);
  gripperServo.attach(12);
  
  pinMode(joySW, INPUT_PULLUP); // 按鈕設為上拉輸入
  
  // 初始化位置
  baseServo.write(baseAngle);
  shoulderServo.write(shoulderAngle);
  elbowServo.write(elbowAngle);
  gripperServo.write(gripperAngle);
}

void loop() {
  int xVal = analogRead(joyX);
  int yVal = analogRead(joyY);
  int swVal = digitalRead(joySW);

  // 控制基座旋轉
  baseAngle = map(xVal, 0, 1023, 0, 180);
  baseServo.write(baseAngle);

  // 控制手臂升降與延伸 (簡化運動學映射)
  shoulderAngle = map(yVal, 0, 1023, 45, 135);
  elbowAngle = map(yVal, 0, 1023, 135, 45);
  shoulderServo.write(shoulderAngle);
  elbowServo.write(elbowAngle);

  // 按鈕控制夾爪開合
  if (swVal == LOW) {
    gripperAngle = 180; // 關閉/抓取
  } else {
    gripperAngle = 90;  // 打開
  }
  gripperServo.write(gripperAngle);

  delay(15); // 避免馬達抖動
}
🔍 常見問題與除錯 (Troubleshooting)伺服馬達抖動或無力：原因：Arduino 板載 5V 電流不足。解法：使用外部 5V 2A 電源單獨供電給伺服馬達，並確認已與 Arduino 共地 (Common GND)。搖桿控制不精準 / 靜止時漂移：原因：搖桿類比訊號微小抖動。解法：在程式中設定搖桿中立區（Deadzone），並調整 map() 函數的輸出邊界。機構關節卡頓：原因：3D 列印公差或螺絲過緊。解法：適度鬆開連桿連接處螺絲，或重新校正 3D 列印件的孔位公差。🚀 未來展望 (Future Work)[ ] 精確逆向運動學 (Inverse Kinematics)：引入 Cartesian 座標系（X, Y, Z），實現直角坐標下的末端軌跡控制。[ ] 多模式控制：整合藍牙模組（HC-05/06）支援手機 APP 或 Serial 介面指令控制。[ ] 感測器回饋：於夾爪處新增壓力感測器（FSR），避免抓取易碎物品時施力過大。
