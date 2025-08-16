# Lab 6.2: Add Source Files to Project

## วัตถุประสงค์
เรียนรู้การเพิ่มไฟล์ source code หลายไฟล์ในโปรเจค ESP32 และการแก้ไข CMakeLists.txt เพื่อจัดการ multiple source files

## ขั้นตอนการทำ Lab

### 1. เตรียม Docker Environment (ต่อเนื่องจาก Lab 6.1)

```bash
# สร้าง directory สำหรับ Lab 6.2
mkdir Lab6_2
cd Lab6_2

# สร้างไฟล์ docker-compose.yml (เหมือน Lab 6.1)
```

สร้างไฟล์ `docker-compose.yml`:

```yaml
version: '3.8'

services:
  esp32-dev:
    image: espressif/idf:latest
    container_name: esp32-lab6-2
    volumes:
      - .:/project
    working_dir: /project
    tty: true
    stdin_open: true
    environment:
      - IDF_PATH=/opt/esp/idf
    command: /bin/bash
    networks:
      - esp32-network

networks:
  esp32-network:
    driver: bridge
```

### 2. รัน Docker และสร้าง Project ใหม่

```bash
# รัน Docker container
docker-compose up -d

# เข้า container
docker-compose exec esp32-dev bash

# ใน container - setup ESP-IDF environment
source $IDF_PATH/export.sh

# สร้าง project ใหม่
idf.py create-project lab6_2_multiple_files
cd lab6_2_multiple_files
```

### 3. สร้างไฟล์ .gitignore สำหรับ ESP32

สร้างไฟล์ `.gitignore` ในโฟลเดอร์ project:

```bash
# สร้างไฟล์ .gitignore (ใน Docker container)
cat > .gitignore << 'EOF'
# ESP-IDF Build Output
build/
sdkconfig.old
dependencies.lock

# ESP-IDF Flash Encryption/Secure Boot Keys
*.key
*.pem

# ESP-IDF Development Tools
.vscode/
.devcontainer/

# Python
__pycache__/
*.py[cod]
*$py.class
*.pyc

# OS Generated Files
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db

# IDE Files
*.swp
*.swo
*~
.idea/

# Build artifacts
*.bin
*.elf
*.map
*.hex

# Temporary files
*.tmp
*.temp
*.log

# ESP32 specific
managed_components/
EOF

# แสดงไฟล์ที่สร้างขึ้น
cat .gitignore
```

### 4. สร้างไฟล์ sensor.c

สร้างไฟล์ `main/sensor.c`:

```c
#include <stdio.h>
#include <stdlib.h>
#include "esp_system.h"
#include "esp_random.h"
#include "esp_log.h"
#include "sensor.h"

static const char *TAG = "SENSOR";

void sensor_init(void)
{
    ESP_LOGI(TAG, "🔧 Sensor initialized from file: %s, line: %d", __FILE__, __LINE__);
    ESP_LOGI(TAG, "📡 Sensor module ready for operation");
}

void sensor_read_data(void)
{
    ESP_LOGI(TAG, "📊 Reading sensor data from file: %s, line: %d", __FILE__, __LINE__);
    
    // จำลองการอ่านข้อมูลจาก sensor
    float temperature = 25.5 + (float)(esp_random() % 100) / 10.0f;
    float humidity = 60.0 + (float)(esp_random() % 400) / 10.0f;
    
    ESP_LOGI(TAG, "🌡️  Temperature: %.1f°C", temperature);
    ESP_LOGI(TAG, "💧 Humidity: %.1f%%", humidity);
}

void sensor_check_status(void)
{
    ESP_LOGI(TAG, "✅ Sensor status check from file: %s, line: %d", __FILE__, __LINE__);
    ESP_LOGI(TAG, "📈 All sensors operating normally");
}
```

### 5. สร้างไฟล์ header sensor.h

สร้างไฟล์ `main/sensor.h`:

```c
#ifndef SENSOR_H
#define SENSOR_H

#ifdef __cplusplus
extern "C" {
#endif

/**
 * @brief Initialize sensor module
 */
void sensor_init(void);

/**
 * @brief Read data from sensors
 */
void sensor_read_data(void);

/**
 * @brief Check sensor status
 */
void sensor_check_status(void);

#ifdef __cplusplus
}
#endif

#endif // SENSOR_H
```

### 6. แก้ไข main/CMakeLists.txt (ครั้งที่ 1)

```cmake
idf_component_register(
    SRCS 
        "lab6_2_multiple_files.c"
        "sensor.c"
    INCLUDE_DIRS 
        "."
)
```

### 7. แก้ไข main/lab6_2_multiple_files.c

```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_log.h"
#include "sensor.h"

static const char *TAG = "MAIN";

void app_main(void)
{
    ESP_LOGI(TAG, "🚀 Lab 6.2: Multiple Source Files Demo");
    ESP_LOGI(TAG, "📍 Main function from file: %s, line: %d", __FILE__, __LINE__);
    ESP_LOGI(TAG, "ESP-IDF Version: %s", esp_get_idf_version());
    
    // Initialize sensor
    sensor_init();
    
    int counter = 0;
    
    while (1) {
        ESP_LOGI(TAG, "=== Loop %d ===", counter++);
        
        // Read sensor data every loop
        sensor_read_data();
        
        // Check sensor status every 3 loops
        if (counter % 3 == 0) {
            sensor_check_status();
        }
        
        vTaskDelay(pdMS_TO_TICKS(2000)); // 2 seconds delay
    }
}
```

### 8. ทดสอบ Build และ Run (ครั้งที่ 1)

```bash
# Build project
idf.py build

# รันด้วย QEMU
idf.py qemu

# กด Ctrl+z เพื่อออกจาก monitor
```

### 9. เพิ่มไฟล์ display.c

สร้างไฟล์ `main/display.c`:

```c
#include <stdio.h>
#include "esp_log.h"
#include "display.h"

static const char *TAG = "DISPLAY";

void display_init(void)
{
    ESP_LOGI(TAG, "🖥️  Display initialized from file: %s, line: %d", __FILE__, __LINE__);
    ESP_LOGI(TAG, "💡 Display module ready");
}

void display_show_message(const char* message)
{
    ESP_LOGI(TAG, "📢 Displaying from file: %s, line: %d", __FILE__, __LINE__);
    ESP_LOGI(TAG, "📺 Message: %s", message);
}

void display_show_data(float value1, float value2)
{
    ESP_LOGI(TAG, "📊 Data display from file: %s, line: %d", __FILE__, __LINE__);
    ESP_LOGI(TAG, "📈 Value 1: %.2f", value1);
    ESP_LOGI(TAG, "📉 Value 2: %.2f", value2);
}

void display_clear_screen(void)
{
    ESP_LOGI(TAG, "🧹 Screen cleared from file: %s, line: %d", __FILE__, __LINE__);
    ESP_LOGI(TAG, "✨ Display ready for new content");
}
```

### 10. สร้างไฟล์ header display.h

สร้างไฟล์ `main/display.h`:

```c
#ifndef DISPLAY_H
#define DISPLAY_H

#ifdef __cplusplus
extern "C" {
#endif

/**
 * @brief Initialize display module
 */
void display_init(void);

/**
 * @brief Show message on display
 * @param message Text message to display
 */
void display_show_message(const char* message);

/**
 * @brief Show numeric data on display
 * @param value1 First value to display
 * @param value2 Second value to display
 */
void display_show_data(float value1, float value2);

/**
 * @brief Clear display screen
 */
void display_clear_screen(void);

#ifdef __cplusplus
}
#endif

#endif // DISPLAY_H
```

### 11. แก้ไข main/CMakeLists.txt (ครั้งที่ 2)

```cmake
idf_component_register(
    SRCS 
        "lab6_2_multiple_files.c"
        "sensor.c"
        "display.c"
    INCLUDE_DIRS 
        "."
)
```

### 12. อัพเดท main/lab6_2_multiple_files.c เพื่อใช้ display

```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_log.h"
#include "sensor.h"
#include "display.h"

static const char *TAG = "MAIN";

void app_main(void)
{
    ESP_LOGI(TAG, "🚀 Lab 6.2: Multiple Source Files Demo");
    ESP_LOGI(TAG, "📍 Main function from file: %s, line: %d", __FILE__, __LINE__);
    ESP_LOGI(TAG, "ESP-IDF Version: %s", esp_get_idf_version());
    
    // Initialize modules
    sensor_init();
    display_init();
    
    display_show_message("System Starting...");
    
    int counter = 0;
    
    while (1) {
        ESP_LOGI(TAG, "=== Loop %d ===", counter++);
        
        // Clear display every loop
        display_clear_screen();
        
        // Read sensor data
        sensor_read_data();
        
        // Show data on display
        float temp = 25.5 + (float)(counter % 10);
        float humid = 60.0 + (float)(counter % 20);
        display_show_data(temp, humid);
        
        // Check sensor status every 3 loops
        if (counter % 3 == 0) {
            sensor_check_status();
            display_show_message("Status Check Complete");
        }
        
        vTaskDelay(pdMS_TO_TICKS(2000)); // 2 seconds delay
    }
}
```

### 13. ทดสอบ Build และ Run (ครั้งที่ 2)

```bash
# Build project อีกครั้ง
idf.py build

# รันด้วย QEMU
idf.py qemu

# สังเกตว่าทั้ง sensor และ display ทำงาน
# กด Ctrl+z เพื่อออกจาก monitor
```

## 🏋️ แบบฝึกหัดเพิ่มเติม: เพิ่มไฟล์ LED.c

### 14. สร้างไฟล์ led.c

สร้างไฟล์ `main/led.c`:

```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_log.h"
#include "led.h"

static const char *TAG = "LED";
static bool led_state = false;

void led_init(void)
{
    ESP_LOGI(TAG, "💡 LED initialized from file: %s, line: %d", __FILE__, __LINE__);
    ESP_LOGI(TAG, "🔧 LED module ready");
    led_state = false;
}

void led_on(void)
{
    led_state = true;
    ESP_LOGI(TAG, "✅ LED ON from file: %s, line: %d", __FILE__, __LINE__);
    ESP_LOGI(TAG, "🟢 LED is now ON");
}

void led_off(void)
{
    led_state = false;
    ESP_LOGI(TAG, "❌ LED OFF from file: %s, line: %d", __FILE__, __LINE__);
    ESP_LOGI(TAG, "🔴 LED is now OFF");
}

void led_toggle(void)
{
    if (led_state) {
        led_off();
    } else {
        led_on();
    }
    ESP_LOGI(TAG, "🔄 LED toggled from file: %s, line: %d", __FILE__, __LINE__);
}

bool led_get_state(void)
{
    ESP_LOGD(TAG, "📊 LED state checked from file: %s, line: %d", __FILE__, __LINE__);
    return led_state;
}

void led_blink_task(void *parameter)
{
    ESP_LOGI(TAG, "✨ LED blink task started from file: %s, line: %d", __FILE__, __LINE__);
    
    while (1) {
        led_toggle();
        vTaskDelay(pdMS_TO_TICKS(3000)); // 3 seconds delay
    }
}

void led_start_blinking(void)
{
    ESP_LOGI(TAG, "🚀 Starting LED blink task from file: %s, line: %d", __FILE__, __LINE__);
    xTaskCreate(led_blink_task, "led_blink", 2048, NULL, 5, NULL);
}
```

### 15. สร้างไฟล์ header led.h

สร้างไฟล์ `main/led.h`:

```c
#ifndef LED_H
#define LED_H

#include <stdbool.h>

#ifdef __cplusplus
extern "C" {
#endif

/**
 * @brief Initialize LED module
 */
void led_init(void);

/**
 * @brief Turn LED on
 */
void led_on(void);

/**
 * @brief Turn LED off
 */
void led_off(void);

/**
 * @brief Toggle LED state
 */
void led_toggle(void);

/**
 * @brief Get current LED state
 * @return true if LED is on, false if off
 */
bool led_get_state(void);

/**
 * @brief LED blink task (runs in separate task)
 * @param parameter Task parameter (not used)
 */
void led_blink_task(void *parameter);

/**
 * @brief Start LED blinking in background task
 */
void led_start_blinking(void);

#ifdef __cplusplus
}
#endif

#endif // LED_H
```

### 16. แก้ไข main/CMakeLists.txt (ครั้งที่ 3 - ขั้นสุดท้าย)

```cmake
idf_component_register(
    SRCS 
        "lab6_2_multiple_files.c"
        "sensor.c"
        "display.c"
        "led.c"
    INCLUDE_DIRS 
        "."
)
```

### 17. อัพเดท main/lab6_2_multiple_files.c เพื่อใช้ LED

```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_log.h"
#include "sensor.h"
#include "display.h"
#include "led.h"

static const char *TAG = "MAIN";

void app_main(void)
{
    ESP_LOGI(TAG, "🚀 Lab 6.2: Multiple Source Files Demo");
    ESP_LOGI(TAG, "📍 Main function from file: %s, line: %d", __FILE__, __LINE__);
    ESP_LOGI(TAG, "ESP-IDF Version: %s", esp_get_idf_version());
    
    // Initialize all modules
    sensor_init();
    display_init();
    led_init();
    
    display_show_message("System Starting...");
    
    // Start LED blinking in background
    led_start_blinking();
    
    int counter = 0;
    
    while (1) {
        ESP_LOGI(TAG, "=== Loop %d ===", counter++);
        
        // Clear display every loop
        display_clear_screen();
        
        // Read sensor data
        sensor_read_data();
        
        // Show data on display
        float temp = 25.5 + (float)(counter % 10);
        float humid = 60.0 + (float)(counter % 20);
        display_show_data(temp, humid);
        
        // Show LED status
        if (led_get_state()) {
            display_show_message("LED Status: ON");
        } else {
            display_show_message("LED Status: OFF");
        }
        
        // Check sensor status every 3 loops
        if (counter % 3 == 0) {
            sensor_check_status();
            display_show_message("Status Check Complete");
        }
        
        vTaskDelay(pdMS_TO_TICKS(2000)); // 2 seconds delay
    }
}
```

### 18. ทดสอบ Build และ Run ขั้นสุดท้าย

```bash
# Build project ขั้นสุดท้าย
idf.py build

# ตรวจสอบขนาด binary
idf.py size

# รันด้วย QEMU
idf.py qemu

# สังเกตการทำงานของทั้ง 4 modules:
# - Main (วนลูปหลัก)
# - Sensor (อ่านข้อมูล)
# - Display (แสดงผล)
# - LED (กะพริบทุก 3 วินาที)

# กด Ctrl+z เพื่อออกจาก monitor
```

## 📊 โครงสร้างไฟล์สุดท้าย

```
Lab6_2/
├── docker-compose.yml
└── lab6_2_multiple_files/
    ├── CMakeLists.txt
    ├── .gitignore              # Git ignore file (เพิ่มใหม่)
    ├── main/
    │   ├── CMakeLists.txt          # แก้ไขแล้ว 3 ครั้ง
    │   ├── lab6_2_multiple_files.c # ไฟล์หลัก
    │   ├── sensor.c                # โมดูล sensor
    │   ├── sensor.h                # header สำหรับ sensor
    │   ├── display.c               # โมดูล display
    │   ├── display.h               # header สำหรับ display
    │   ├── led.c                   # โมดูล LED
    │   └── led.h                   # header สำหรับ LED
    ├── build/                      # Build output
    └── README.md
```

## 🔍 คำถามทบทวน

1. **Multiple Source Files**: เหตุใดต้องแยก source code เป็นหลายไฟล์?         Multiple Source Files: แยกไฟล์เพื่อลดความซับซ้อน แก้ง่าย ดูแลสะดวก
2. **CMakeLists.txt Management**: การเพิ่มไฟล์ source ใหม่ต้องแก้ไขอะไรบ้าง?     CMakeLists.txt Management: ต้องเพิ่มไฟล์ใน SRCS และอาจแก้ INCLUDE_DIRS
3. **Header Files**: บทบาทของไฟล์ .h คืออะไร และทำไมต้องมี?                  Header Files (.h): ใช้ประกาศฟังก์ชัน/ตัวแปร เพื่อให้ไฟล์อื่นเรียกใช้ร่วมกันได้
4. **Include Directories**: เหตุใด CMakeLists.txt ต้องระบุ INCLUDE_DIRS?      Include Directories: ต้องใส่ INCLUDE_DIRS เพื่อให้คอมไพเลอร์หาไฟล์ .h เจอ
5. **Git Ignore**: ไฟล์ .gitignore ช่วยอะไรในการจัดการ ESP32 project?         Git Ignore: กันไม่ให้ไฟล์ชั่วคราว/ไม่จำเป็นถูก push ขึ้น repo
6. **Task Management**: การใช้ FreeRTOS task ในโมดูล LED ช่วยอะไร?            Task Management: FreeRTOS task แยกการทำงาน ทำให้ LED ทำงานอิสระไม่บล็อกโปรแกรม
7. **Code Organization**: ข้อดีของการแยกโมดูล sensor, display, led เป็นไฟล์แยกคืออะไร?   ode Organization: แยกโมดูล (sensor, display, led) ช่วยให้โค้ดชัดเจน ดูแลง่าย และนำกลับใช้ซ้ำได้

## 📋 ผลลัพธ์ที่คาดหวัง

เมื่อทำ lab สำเร็จ นักศึกษาจะ:
- **📁 จัดการ Multiple Files**: สามารถสร้างและจัดการหลายไฟล์ source code
- **⚙️ แก้ไข CMakeLists.txt**: เข้าใจการเพิ่ม source files ใน build configuration
- **🔗 Header Files**: เข้าใจการใช้ header files เพื่อเชื่อมโยงระหว่างไฟล์
- **📂 Git Management**: เข้าใจการใช้ .gitignore สำหรับ ESP32 development
- **🏗️ Code Organization**: สามารถแยกโค้ดเป็นโมดูลต่างๆ ตามหน้าที่
- **🔄 Task Management**: เข้าใจการใช้ FreeRTOS tasks สำหรับงานที่ทำงานขนาน
- **📍 Debug Information**: เข้าใจการใช้ __FILE__ และ __LINE__ เพื่อ debug

## 💡 บันทึกผลการทดลอง

**ขั้นตอนที่ 1 (เฉพาะ sensor.c):**
-จำนวนไฟล์ source: 1
-ขนาด binary: ~62 KB (Flash Code)
-การทำงาน: อ่านค่าจาก sensor

**ขั้นตอนที่ 2 (เพิ่ม display.c):**
-จำนวนไฟล์ source: 2
-ขนาด binary: ~100 KB (Flash Code + Flash Data เพิ่มขึ้นจากโค้ดจอ)
-การทำงาน: อ่าน sensor + แสดงผลบนจอ

**ขั้นตอนที่ 3 (เพิ่ม led.c):**
-จำนวนไฟล์ source: 3
-ขนาด binary: ~165 KB (ตามตารางรวม)
-การทำงาน: อ่าน sensor + แสดงผลบนจอ + ควบคุม LED

**สังเกต:**
- การเพิ่มไฟล์ source ส่งผลต่อขนาด binary อย่างไร?
- LED กะพริบทุกกี่วินาที?
- แต่ละโมดูลแสดงข้อมูล file และ line อย่างไร?

## 🛠️ การแก้ไขปัญหาที่พบบ่อย

### ปัญหา: Build Error - undefined reference
```
undefined reference to `sensor_init'
```
**วิธีแก้**: 
- ตรวจสอบว่าได้เพิ่ม "sensor.c" ใน CMakeLists.txt แล้ว
- ตรวจสอบว่าได้ #include "sensor.h" ในไฟล์ main

### ปัญหา: Header file not found
```
fatal error: sensor.h: No such file or directory
```
**วิธีแก้**: 
- ตรวจสอบว่าไฟล์ sensor.h อยู่ในโฟลเดอร์ main/
- ตรวจสอบ INCLUDE_DIRS ใน CMakeLists.txt

### ปัญหา: Function declaration conflicts
```
conflicting types for 'sensor_init'
```
**วิธีแก้**: 
- ตรวจสอบว่า function signature ใน .h และ .c ตรงกัน
- ใช้ #ifdef guards ในไฟล์ header

### ปัญหา: LED task not running
```
LED doesn't blink as expected
```
**วิธีแก้**: 
- ตรวจสอบว่าได้เรียก led_start_blinking() ใน app_main()
- ตรวจสอบ task stack size และ priority

Docker environment ที่ setup จะใช้ต่อเนื่องใน Lab ถัดไป
