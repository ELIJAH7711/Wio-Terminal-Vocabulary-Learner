![Arduino编译状态](https://github.com/ELIJAH7711/Wio-Terminal-Vocabulary-Learner/workflows/Arduino%20CI/badge.svg)
![许可证](https://img.shields.io/github/license/ELIJAH7711/Wio-Terminal-Vocabulary-Learner)
# Wio-Terminal-Vocabulary-Learner
“Easy Vocabulary Memorization Program for Wio Terminal - Electronics for Beginners”
# Wio Terminal 背单词学习器

![image](https://github.com/user-attachments/assets/f3ebf96e-901c-480d-82e2-0bf2268efc27)


## 项目概述
基于Wio Terminal的简易背单词工具，适合电子初学者入门学习

## 硬件要求
- Wio Terminal ×1
- USB Type-C数据线 ×1

## 快速开始
1. 安装[Arduino IDE](https://www.arduino.cc/en/software)
2. 添加Wio Terminal支持包
3. 克隆本仓库
4. 打开`Vocabulary_Learner/Vocabulary_Learner.ino`
5. 上传到设备

## 功能说明
- 按右侧按键切换单词
- 屏幕显示英文单词和中文释义
- 可扩展的单词库

## 详细指南
### 1. Wio Terminal设计理念
Wio Terminal 是一款高度集成的开发板，设计理念是“All-in-One”：

●内置 2.4英寸LCD屏幕（320×240分辨率）

●集成 Wi-Fi/BLE、三轴加速度计、光传感器

●提供 Grove扩展接口（简化传感器连接）

●兼容 Arduino和MicroPython，适合快速原型开发

●核心目标：让电子项目开发无需额外接线，降低初学者门槛

### 2. 接口说明
![image](https://github.com/user-attachments/assets/7c0d1c31-08e8-4984-aa5a-ae9aec9698be)
	


### 3. 连接步骤
1.安装驱动：

--下载Wio Terminal Arduino支持包
--Arduino IDE → 工具 → 开发板 → 选择 "Seeed SAMD Boards" → 选 "Wio Terminal"

2.连接电脑：

--USB线连接Wio Terminal与电脑
--工具 → 端口 → 选择对应COM口

3.上传测试程序：
```
void setup() { pinMode(LED_BUILTIN, OUTPUT); }  
void loop() {  
  digitalWrite(LED_BUILTIN, HIGH); // 点亮蓝色LED  
  delay(1000);  
  digitalWrite(LED_BUILTIN, LOW);  
  delay(1000);  
}  
```
点击“上传” → 观察板载LED闪烁即成功


### 4. 代码示例
```cpp
const int WORD_COUNT = 5; // 单词数量

String words[WORD_COUNT] = {
  "Apple", 
  "Banana",
  "Computer",
  "Engineer",
  "Microcontroller"
};

String meanings[WORD_COUNT] = {
  "苹果", 
  "香蕉",
  "电脑",
  "工程师",
  "微控制器"
};
```

```
###（完整代码）
#include <TFT_eSPI.h>
#include <SPI.h>
#include <Wire.h>

TFT_eSPI tft;

// 单词库结构
struct WordEntry {
  const char* word;
  const char* meaning;
  const char* example;
  bool mastered;
};

// 单词库数据（存储在Flash中）
const WordEntry wordDatabase[] = {
  {"algorithm", "suan fa", "The search algorithm is very efficient.", false},
  {"variable", "bian liang", "Declare a variable to store the value.", false},
  {"function", "han shu", "This function calculates the square root.", false},
  {"loop", "xun huan", "Use a loop to repeat the process.", false},
  {"syntax", "yu fa", "Python has a clean and readable syntax.", false},
  {"debug", "diao shi", "It took hours to debug this code.", false},
  {"compile", "bian yi", "Compile the program before running.", false},
  {"array", "shu zu", "Store the data in an array structure.", false},
  {"pointer", "zhi zhen", "Pointers provide direct memory access.", false},
  {"recursion", "di gui", "Recursion can solve this problem elegantly.", false},
  {"interface", "jie kou", "Design a clear interface for the module.", false},
  {"database", "shu ju ku", "Store user information in a database.", false},
  {"encryption", "jia mi", "Use encryption to protect sensitive data.", false},
  {"framework", "kuang jia", "This framework simplifies web development.", false},
  {"cache", "huan cun", "Implement cache to improve performance.", false}
};

const int wordCount = sizeof(wordDatabase) / sizeof(wordDatabase[0]);
int currentIndex = 0;
bool showMeaning = false;
int masteredCount = 0;
int reviewCount = 0;
bool inReviewMode = false;
int originalIndex = 0;

// 按钮引脚定义（使用Wio Terminal的5向摇杆）
#define JOYSTICK_UP_PIN 38
#define JOYSTICK_DOWN_PIN 37
#define JOYSTICK_LEFT_PIN 39
#define JOYSTICK_RIGHT_PIN 36
#define JOYSTICK_CENTER_PIN 35

// 颜色定义
#define BACKGROUND_COLOR TFT_BLACK
#define TEXT_COLOR TFT_WHITE
#define HIGHLIGHT_COLOR TFT_YELLOW
#define MASTERED_COLOR TFT_GREEN
#define NEW_WORD_COLOR TFT_BLUE
#define BUTTON_COLOR TFT_MAGENTA
#define STATUS_COLOR TFT_NAVY
#define GREY_COLOR 0x7BEF  // 自定义灰色 (RGB565)

// 按钮状态跟踪
unsigned long lastButtonPress = 0;
const int debounceDelay = 50; // 减少防抖延迟到50ms
const int refreshDelay = 100; // 刷新延迟

void setup() {
  // 初始化串口
  Serial.begin(115200);
  Serial.println("Starting Vocabulary App (Fast Response)...");
  
  // 初始化显示屏
  tft.begin();
  tft.setRotation(3);
  tft.fillScreen(BACKGROUND_COLOR);
  tft.setTextFont(2); // 使用更快的字体
  
  // 初始化按钮（使用内部上拉电阻）
  pinMode(JOYSTICK_UP_PIN, INPUT_PULLUP);
  pinMode(JOYSTICK_DOWN_PIN, INPUT_PULLUP);
  pinMode(JOYSTICK_LEFT_PIN, INPUT_PULLUP);
  pinMode(JOYSTICK_RIGHT_PIN, INPUT_PULLUP);
  pinMode(JOYSTICK_CENTER_PIN, INPUT_PULLUP);
  
  // 快速显示欢迎界面
  tft.fillScreen(TFT_BLACK);
  tft.setTextColor(TFT_YELLOW);
  tft.setTextSize(3);
  tft.drawString("WioTerm Vocab", 15, 30);
  tft.setTextColor(TFT_CYAN);
  tft.setTextSize(2);
  tft.drawString("v5.0 - Fast", 70, 80);
  
  // 初始化单词状态
  updateMasteredCount();
  
  // 快速显示第一个单词
  displayWord();
  
  // 打印初始按钮状态（用于调试）
  printButtonStatus();
}

void loop() {
  // 检查按钮输入
  checkButtons();
  
  // 短暂延迟防止过于频繁检测
  delay(5);
}

// 打印按钮状态（调试用）
void printButtonStatus() {
  Serial.println("Button Status (1=HIGH, 0=LOW):");
  Serial.print("UP: "); Serial.println(digitalRead(JOYSTICK_UP_PIN));
  Serial.print("DOWN: "); Serial.println(digitalRead(JOYSTICK_DOWN_PIN));
  Serial.print("LEFT: "); Serial.println(digitalRead(JOYSTICK_LEFT_PIN));
  Serial.print("RIGHT: "); Serial.println(digitalRead(JOYSTICK_RIGHT_PIN));
  Serial.print("CENTER: "); Serial.println(digitalRead(JOYSTICK_CENTER_PIN));
  Serial.println("----------------------------");
}

void updateMasteredCount() {
  masteredCount = 0;
  for (int i = 0; i < wordCount; i++) {
    if (wordDatabase[i].mastered) {
      masteredCount++;
    }
  }
}

void displayWord() {
  // 只刷新必要的部分，而不是整个屏幕
  if (!inReviewMode) {
    tft.fillScreen(BACKGROUND_COLOR);
  }
  
  // 显示顶部状态栏
  drawStatusBar();
  
  // 显示当前单词
  tft.setTextColor(TEXT_COLOR);
  tft.setTextSize(3);
  tft.setCursor(20, 60);
  tft.println(wordDatabase[currentIndex].word);
  
  // 显示例句
  tft.setTextColor(TFT_CYAN);
  tft.setTextSize(1);
  tft.setCursor(20, 110);
  tft.println("Example:");
  tft.setCursor(20, 130);
  tft.println(wordDatabase[currentIndex].example);
  
  if (showMeaning) {
    // 显示释义
    tft.setTextColor(HIGHLIGHT_COLOR);
    tft.setTextSize(2);
    tft.setCursor(20, 170);
    tft.print("Meaning: ");
    tft.println(wordDatabase[currentIndex].meaning);
    
    // 显示提示
    tft.setTextColor(GREY_COLOR);
    tft.setTextSize(1);
    tft.setCursor(40, 210);
    tft.println("Press UP to mark mastered");
  } else {
    // 显示提示
    tft.setTextColor(GREY_COLOR);
    tft.setTextSize(1);
    tft.setCursor(30, 170);
    tft.println("Press CENTER to show meaning");
    tft.setCursor(30, 190);
    tft.println("Press LEFT/RIGHT to navigate");
    tft.setCursor(30, 210);
    tft.println("Press DOWN to review mastered");
  }
  
  // 显示按钮映射
  drawButtonMap();
  
  if (inReviewMode) {
    // 显示复习模式提示
    tft.setTextColor(TFT_YELLOW);
    tft.setTextSize(1);
    tft.setCursor(120, 40);
    tft.print("REVIEW MODE");
  }
}

void drawStatusBar() {
  // 绘制状态栏背景
  tft.fillRect(0, 0, tft.width(), 30, STATUS_COLOR);
  
  // 显示当前进度
  tft.setTextColor(TEXT_COLOR);
  tft.setTextSize(1);
  tft.setCursor(10, 10);
  tft.print(String(currentIndex + 1));
  tft.print("/");
  tft.print(wordCount);
  
  // 显示已掌握单词数量
  tft.setCursor(90, 10);
  tft.print("Mastered: ");
  tft.print(masteredCount);
  
  // 显示单词掌握状态
  if (wordDatabase[currentIndex].mastered) {
    tft.setTextColor(MASTERED_COLOR);
    tft.setCursor(200, 10);
    tft.print("Mastered");
  } else {
    tft.setTextColor(NEW_WORD_COLOR);
    tft.setCursor(200, 10);
    tft.print("New Word");
  }
}

void drawButtonMap() {
  // 在屏幕底部显示按钮功能
  tft.setTextColor(BUTTON_COLOR);
  tft.setTextSize(1);
  
  // 左按钮
  tft.fillRect(10, 220, 60, 20, STATUS_COLOR);
  tft.setCursor(15, 225);
  tft.print("LEFT");
  
  // 右按钮
  tft.fillRect(80, 220, 60, 20, STATUS_COLOR);
  tft.setCursor(85, 225);
  tft.print("RIGHT");
  
  // 中央按钮
  tft.fillRect(150, 220, 80, 20, STATUS_COLOR);
  tft.setCursor(155, 225);
  tft.print("CENTER");
  
  // 上按钮
  tft.fillRect(240, 220, 60, 20, STATUS_COLOR);
  tft.setCursor(245, 225);
  tft.print("UP");
  
  // 下按钮
  tft.fillRect(10, 245, 80, 20, STATUS_COLOR);
  tft.setCursor(15, 250);
  tft.print("DOWN");
}

void checkButtons() {
  // 检查防抖时间
  if (millis() - lastButtonPress < debounceDelay) {
    return;
  }
  
  // 左按钮 - 上一个单词
  if (digitalRead(JOYSTICK_LEFT_PIN) == LOW) {
    Serial.println("LEFT button pressed");
    lastButtonPress = millis();
    showMeaning = false;
    currentIndex = (currentIndex - 1 + wordCount) % wordCount;
    displayWord();
    return;
  }
  
  // 右按钮 - 下一个单词
  if (digitalRead(JOYSTICK_RIGHT_PIN) == LOW) {
    Serial.println("RIGHT button pressed");
    lastButtonPress = millis();
    showMeaning = false;
    currentIndex = (currentIndex + 1) % wordCount;
    displayWord();
    return;
  }
  
  // 中央按钮 - 显示/隐藏释义
  if (digitalRead(JOYSTICK_CENTER_PIN) == LOW) {
    Serial.println("CENTER button pressed");
    lastButtonPress = millis();
    showMeaning = !showMeaning;
    displayWord();
    return;
  }
  
  // 上按钮 - 标记为已掌握
  if (digitalRead(JOYSTICK_UP_PIN) == LOW) {
    Serial.println("UP button pressed");
    lastButtonPress = millis();
    if (!wordDatabase[currentIndex].mastered) {
      // 更新掌握状态
      WordEntry* word = (WordEntry*)&wordDatabase[currentIndex];
      word->mastered = true;
      updateMasteredCount();
      
      // 显示快速确认
      tft.fillRect(40, 80, 240, 60, TFT_DARKGREEN);
      tft.drawRect(40, 80, 240, 60, TFT_GREEN);
      tft.setTextColor(TFT_WHITE);
      tft.setTextSize(2);
      tft.setCursor(60, 100);
      tft.print("Marked!");
      delay(300); // 短暂显示确认
    }
    displayWord();
    return;
  }
  
  // 下按钮 - 复习已掌握的单词
  if (digitalRead(JOYSTICK_DOWN_PIN) == LOW) {
    Serial.println("DOWN button pressed");
    lastButtonPress = millis();
    enterReviewMode();
    return;
  }
  
  // 在复习模式下检查按钮
  if (inReviewMode) {
    checkReviewButtons();
  }
}

void enterReviewMode() {
  int masteredWords[wordCount];
  int count = 0;
  
  // 收集已掌握的单词索引
  for (int i = 0; i < wordCount; i++) {
    if (wordDatabase[i].mastered) {
      masteredWords[count++] = i;
    }
  }
  
  if (count == 0) {
    // 没有已掌握的单词
    tft.fillRect(40, 80, 240, 60, TFT_DARKGREEN);
    tft.drawRect(40, 80, 240, 60, TFT_GREEN);
    tft.setTextColor(TFT_WHITE);
    tft.setTextSize(2);
    tft.setCursor(60, 100);
    tft.print("No mastered words!");
    delay(800);
    displayWord();
    return;
  }
  
  // 进入复习模式
  inReviewMode = true;
  originalIndex = currentIndex;
  reviewCount = (reviewCount + 1) % count;
  currentIndex = masteredWords[reviewCount];
  showMeaning = false;
  
  // 刷新显示
  displayWord();
}

void checkReviewButtons() {
  // 检查防抖时间
  if (millis() - lastButtonPress < debounceDelay) {
    return;
  }
  
  // 中央按钮 - 显示/隐藏释义
  if (digitalRead(JOYSTICK_CENTER_PIN) == LOW) {
    Serial.println("CENTER pressed in review");
    lastButtonPress = millis();
    showMeaning = !showMeaning;
    displayWord();
    return;
  }
  
  // 下按钮 - 下一个已掌握单词
  if (digitalRead(JOYSTICK_DOWN_PIN) == LOW) {
    Serial.println("DOWN pressed in review");
    lastButtonPress = millis();
    
    int masteredWords[wordCount];
    int count = 0;
    
    // 重新收集已掌握的单词索引
    for (int i = 0; i < wordCount; i++) {
      if (wordDatabase[i].mastered) {
        masteredWords[count++] = i;
      }
    }
    
    if (count == 0) {
      inReviewMode = false;
      currentIndex = originalIndex;
      displayWord();
      return;
    }
    
    reviewCount = (reviewCount + 1) % count;
    currentIndex = masteredWords[reviewCount];
    showMeaning = false;
    displayWord();
    return;
  }
  
  // 左/右按钮 - 退出复习模式
  if (digitalRead(JOYSTICK_LEFT_PIN) == LOW || digitalRead(JOYSTICK_RIGHT_PIN) == LOW) {
    Serial.println("Exit review mode");
    lastButtonPress = millis();
    inReviewMode = false;
    currentIndex = originalIndex;
    showMeaning = false;
    displayWord();
    return;
  }
}
```
## 5. 常见问题
Q1：屏幕不亮怎么办？

A：

--检查 tft.init() 是否在setup()中调用

--添加背光控制：pinMode(LCD_BACKLIGHT, OUTPUT); digitalWrite(LCD_BACKLIGHT, HIGH);

Q2：按键无反应？

A：

--确认引脚模式设置为 INPUT_PULLUP

--用万用表检测按键是否损坏（按下时电阻应接近0Ω）

Q3：如何添加更多单词？

A：

--扩展数组并修改循环上限

## 扩展建议
1. 增加发音功能
2. 添加Wi-Fi联网
3. 美化UI界面

## 贡献指南
欢迎提交Pull Request！请遵循：
## 🤝 如何贡献
1. Fork本仓库
2. 创建新分支 (`git checkout -b feature/improvement`)
3. 提交修改 (`git commit -am 'Add new feature'`)
4. 推送到分支 (`git push origin feature/improvement`)
5. 创建Pull Request

## 📜 行为准则
- 保持友善和专业的交流环境
- 欢迎初学者提问
- 所有贡献需遵守MIT许可证
