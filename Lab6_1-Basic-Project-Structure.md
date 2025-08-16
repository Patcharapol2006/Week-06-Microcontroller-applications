# Lab 6-1: สร้าง Basic ESP32 Project Structure

## วัตถุประสงค์
เรียนรู้การสร้างโครงสร้าง project ESP32 แบบ standard และเข้าใจการทำงานของ CMake build system

## ขั้นตอนการทำ Lab

### 1. เตรียม Docker Environment (ต่อเนื่องจากสัปดาห์ก่อน)

```bash
# สร้าง directory สำหรับ Lab 6
mkdir Lab6
cd Lab6

# สร้างไฟล์ docker-compose.yml (เหมือนสัปดาห์ก่อน)
```

สร้างไฟล์ `docker-compose.yml`:

```yaml
version: '3.8'

services:
  esp32-dev:
    image: espressif/idf:latest
    container_name: esp32-lab5
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

### 2. รัน Docker และสร้าง Project

```bash
# รัน Docker container (คำสั่งเดิมที่คุ้นเคย)
docker-compose up -d


# ถ้าขึ้นข้อความลักษณะนี้ แสดงว่ายังไม่ได้รัน Docker Desktop ให้ไปรันให้เรียบร้อยก่อน 
# error during connect: Get "http://%2F%2F.%2Fpipe%2FdockerDesktopLinuxEngine/v1.51/images/espressif/idf:latest/json": open //./pipe/dockerDesktopLinuxEngine: The system cannot find the file specified.


# เข้า container
docker-compose exec esp32-dev bash

# ใน container - setup ESP-IDF environment
source $IDF_PATH/export.sh

# สร้าง project ใหม่
idf.py create-project lab6_1_basic_build
cd lab6_1_basic_build
```

### 3. สร้างไฟล์ .gitignore สำหรับ ESP32

สร้างไฟล์ `.gitignore` ในโฟลเดอร์ project ให้มีเนื้อหาต่อไปนี้

```bash
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
```

```bash
# แสดงไฟล์ที่สร้างขึ้น
cat .gitignore
```

### 4. ศึกษาโครงสร้าง Project ที่สร้างขึ้น

```
lab1_basic_build/
├── CMakeLists.txt          # Top-level CMake configuration
├── main/                   # Main application directory
│   ├── CMakeLists.txt     # Main component CMake file
│   └── lab1_basic_build.c # Main source file (auto-generated)
├── .gitignore             # Git ignore file (เพิ่มใหม่)
└── README.md              # Project documentation
```


### 5. แก้ไข main/lab1_basic_build.c

```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_log.h"

static const char *TAG = "LAB6_1";

void app_main(void)
{
    ESP_LOGI(TAG, "Lab 6.1: Basic ESP32 Project Structure");
    ESP_LOGI(TAG, "ESP-IDF Version: %s", esp_get_idf_version());
    ESP_LOGI(TAG, "Free heap size: %d bytes", esp_get_free_heap_size());
    
    int counter = 0;
    
    while (1) {
        ESP_LOGI(TAG, "Build system test - Counter: %d", counter++);
        vTaskDelay(pdMS_TO_TICKS(2000)); // 2 seconds delay
    }
}
```

### 6. ศึกษาไฟล์ CMakeLists.txt

#### Top-level CMakeLists.txt
```cmake
# The following lines of boilerplate have to be in your project's
# CMakeLists.txt, at minimum.
cmake_minimum_required(VERSION 3.16)

include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(lab1_basic_build)
```

#### main/CMakeLists.txt
```cmake
idf_component_register(SRCS "lab1_basic_build.c"
                       INCLUDE_DIRS ".")
```

### 7. การ Build Project (ใน Docker Container)

```bash
# ตรวจสอบว่าอยู่ใน container และ directory ที่ถูกต้อง
pwd  # ควรแสดง /project/lab6_1_basic_build

# Configure project (first time only)
idf.py set-target esp32

# Build project
idf.py build

# Check build output
ls build/

# ตรวจสอบรายการ ควรเห็นไฟล์  lab6_1_basic_build.elf

```

### 8. การวิเคราะห์ Build Output (ใน Docker Container)

```bash
# ดูขนาด binary
idf.py size

# บันทึกภาพตาราง ใส่ในไฟล์ส่งงาน
<img width="943" height="434" alt="image" src="https://github.com/user-attachments/assets/e453269d-799c-47a7-b5de-9ba966b32919" />

# ดูรายละเอียดขนาดตาม component
idf.py size-components

# ถ้ามีปัญหาในการดูรายละเอียดขนาดตาม component บนหน้าจอ ให้ใช้คำสั่ง
idf.py size-components > size-components.txt

# แล้วแนบไฟล์ size-components.txt ในโฟลเดอร์ส่งงาน

# ดูรายละเอียดขนาดตาม source file
idf.py size-files

# ถ้ามีปัญหาในการดูรายละเอียดขนาดตาม source file บนหน้าจอ ให้ใช้คำสั่ง
idf.py size-files > size-files.txt

# แล้วแนบไฟล์ size-files.txt ในโฟลเดอร์ส่งงาน

```




### 9. การ Flash และทดสอบ (ต้องออกจาก Docker หรือใช้ QEMU)

#### วิธีที่ 1: ใช้ QEMU Emulator (แนะนำสำหรับ Lab)
```bash
# ใน Docker container
idf.py qemu 

# กด Ctrl+z เพื่อออกจาก monitor
```

## 🐳 Docker Environment Overview

### โครงสร้างไฟล์หลังจากทำ Lab เสร็จ:

```
Lab1-ESP32-Basic-Project/         # โฟลเดอร์หลัก
├── docker-compose.yml            # Docker configuration (เหมือนสัปดาห์ก่อน)
└── lab1_basic_build/            # ESP32 project ที่สร้างใน container
    ├── CMakeLists.txt           # Top-level CMake configuration
    ├── main/                    # Main application directory
    │   ├── CMakeLists.txt      # Main component CMake file
    │   └── lab1_basic_build.c  # Main source file
    ├── build/                   # Build output (สร้างหลัง build)
    └── README.md               # Project documentation
```

### ข้อดีของการใช้ Docker:
- **🔧 Environment Consistency**: ESP-IDF environment เหมือนกันทุกเครื่อง
- **📦 No Installation Hassle**: ไม่ต้องติดตั้ง ESP-IDF บน host system
- **🔄 Easy Cleanup**: ลบ container ได้ง่าย ไม่ทิ้งไฟล์ขยะ
- **👥 Team Collaboration**: ทุกคนใช้ environment เดียวกัน

### คำสั่ง Docker ที่ใช้บ่อย:
```bash
# เริ่ม container
docker-compose up -d

# เข้า container
docker-compose exec esp32-dev bash

# ดู container ที่กำลังทำงาน
docker-compose ps

# หยุด container
docker-compose down

# ดู logs
docker-compose logs esp32-dev
```

## การทดลองเพิ่มเติม

### 1. เพิ่ม Build Information (ใน Docker Container)

แก้ไข main/lab6_1_basic_build.c:

```bash
# เข้า container (ถ้ายังไม่ได้เข้า)
docker-compose exec esp32-dev bash
source $IDF_PATH/export.sh
cd lab6_1_basic_build

# แก้ไขไฟล์
```

```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_log.h"

static const char *TAG = "LAB1";

void print_build_info(void)
{
    ESP_LOGI(TAG, "=== Build Information ===");
    ESP_LOGI(TAG, "Project Name: lab6_1_basic_build");
    ESP_LOGI(TAG, "ESP-IDF Version: %s", esp_get_idf_version());
    ESP_LOGI(TAG, "Compile Date: %s", __DATE__);
    ESP_LOGI(TAG, "Compile Time: %s", __TIME__);
    ESP_LOGI(TAG, "Chip Model: %s", CONFIG_IDF_TARGET);
    ESP_LOGI(TAG, "Free Heap: %d bytes", esp_get_free_heap_size());
}

void app_main(void)
{
    print_build_info();
    
    int counter = 0;
    
    while (1) {
        ESP_LOGI(TAG, "Running... Counter: %d", counter++);
        
        // แสดงสถานะ memory ทุกๆ 10 ครั้ง
        if (counter % 10 == 0) {
            ESP_LOGI(TAG, "Current free heap: %d bytes", esp_get_free_heap_size());
        }
        
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}
```


บันทึกผลการ simulate ในโฟลเดอร์ส่งงาน


## 🔍 คำถามทบทวน

1. **Docker vs Native Setup**: อธิบายข้อดีของการใช้ Docker เปรียบเทียบกับการติดตั้ง ESP-IDF บน host system Docker vs Native Setup

ข้อดีของ Docker

Environment คงที่ ลดปัญหา build บนเครื่องอื่นไม่สำเร็จ

แยก environment ออกจาก host ป้องกัน dependency conflict

Setup ง่าย แค่ pull image ก็เริ่มพัฒนาได้

พกพาได้ ใช้บน Linux, macOS, Windows โดยไม่ต้องกังวล OS

Host system สะอาด ไม่ต้องติดตั้ง toolchain และ dependencies เยอะ

ข้อดีของ Native Setup

Performance ดีกว่าเล็กน้อย

ใช้งานร่วมกับ IDE บน host ได้สะดวก

ไม่ต้องเรียนรู้คำสั่ง Docker
2. **Build Process**: อธิบายขั้นตอนการ build ของ ESP-IDF ใน Docker container ตั้งแต่ source code จนได้ binary เตรียม Docker Container

Run container ที่มี ESP-IDF ติดตั้งพร้อมใช้งาน

Mount project folder จาก host เข้า container (เช่น -v $(pwd):/project)

ตั้งค่า Environment

รัน source /opt/esp-idf/export.sh หรือ idf.py export เพื่อโหลด environment variables ของ ESP-IDF

ตั้งค่า PATH ให้ toolchain, Python, CMake, Ninja สามารถเรียกใช้งานได้

CMake Configuration

รัน idf.py set-target <target> เพื่อเลือก ESP32 รุ่นที่ต้องการ

CMake อ่าน CMakeLists.txt ทั้งระดับ project และ component

สร้างไฟล์ build system และ config (Makefile/Ninja build files)

Build Project

รัน idf.py build

CMake + Ninja คอมไพล์ source code ของทุก component → object files → ลิงก์เป็น firmware binary (.elf)

สร้างไฟล์เพิ่มเติม เช่น .bin, .map

Output Binary

Binary จะอยู่ในโฟลเดอร์ build/

สามารถแฟลชลง ESP32 ด้วย idf.py flash
3. **CMake Files**: บทบาทของไฟล์ CMakeLists.txt แต่ละไฟล์คืออะไร และทำงานอย่างไรใน Docker environment? อยู่ใน root folder ของโปรเจกต์

บทบาท:

กำหนด ชื่อ project (project(<name>))

ระบุ minimum CMake version

Include ESP-IDF toolchain และ component (include($ENV{IDF_PATH}/tools/cmake/project.cmake))

Docker ทำงานตรงนี้เหมือน native เพราะ container มี ESP-IDF อยู่แล้ว
แต่ละ component มี CMakeLists.txt ของตัวเอง

บทบาท:

กำหนด source files (set(srcs ...))

ระบุ header files (set(include_dirs ...))

กำหนด dependencies ของ component (idf_component_register(...))

Docker ใช้ component path จาก container environment ทำให้คอมไพล์ได้ตรงตาม dependencies
4. **Git Ignore**: ไฟล์ .gitignore มีความสำคัญอย่างไรสำหรับ ESP32 project development? 
ไม่เก็บไฟล์ build/ output

เช่น build/, .pio/, *.elf, *.bin

เหล่านี้ถูกสร้างขึ้นใหม่ทุกครั้งที่ build → ไม่ควรเก็บใน Git

ลดขนาด repository

ไฟล์ binary หรือ intermediate files มีขนาดใหญ่ → ทำให้ repo โตเร็ว

.gitignore ช่วยเก็บเฉพาะ source code และ configuration

ป้องกัน conflict

ไฟล์ build บางครั้งเปลี่ยนทุก build → ถ้าเก็บใน Git อาจเกิด merge conflict

ความปลอดภัย

ป้องกันไม่ให้ไฟล์ส่วนตัวหรือ keys ถูก push เช่น .env หรือ sdkconfig (ถ้าไม่อยาก share)
5. **Container Persistence**: ข้อมูลใดบ้างที่จะหายไปเมื่อ restart container และข้อมูลใดที่จะอยู่ต่อ?
Container filesystem ภายใน ที่ไม่ได้ mount เป็น volume

เช่น ถ้าสร้างไฟล์ใน /root หรือ /esp ภายใน container แต่ไม่ได้ mount folder จาก host → เมื่อ container ถูกลบหรือ restart ข้อมูลเหล่านี้จะหาย

Temporary build files

หากไม่ได้ mount build/ หรือ project folder จาก host → build output จะหายเมื่อ container ถูกลบ

2. ข้อมูลที่จะอยู่ต่อ

ข้อมูลใน volumes หรือ bind mounts

เช่น docker run -v $(pwd):/project → ไฟล์ source code, sdkconfig, CMakeLists.txt จะอยู่บน host และไม่หาย

Docker image

ทุกอย่างที่ติดตั้งใน image (ESP-IDF, toolchain, Python packages) จะอยู่ตลอด → container ใหม่จาก image จะพร้อมใช้งานทันที
6. **Development Workflow**: เปรียบเทียบ workflow การพัฒนาระหว่างการใช้ Docker กับการทำงานบน native system
1. Workflow บน Docker

Pull Docker image ของ ESP-IDF

Run container พร้อม mount project folder จาก host

เข้า container (docker exec -it <container> /bin/bash)

โหลด environment ของ ESP-IDF (source /opt/esp-idf/export.sh)

ใช้คำสั่ง idf.py ใน container:

idf.py set-target

idf.py build → build binary

idf.py flash → flash ลง ESP32

Source code อยู่บน host → เปลี่ยนโค้ดได้ทันทีโดยไม่เสียข้อมูลเมื่อ container restart
ติดตั้ง ESP-IDF และ dependencies บน host

ตั้งค่า PATH และ environment variables

ใช้ idf.py บน host:

idf.py set-target

idf.py build

idf.py flash

Source code และ build output อยู่บน host

ข้อดี:

Performance ดีกว่าเล็กน้อย

IDE integration ง่าย (VS Code, CLion)

ไม่ต้องเรียนรู้คำสั่ง Docker

ข้อเสีย:

ต้องจัดการ dependency และ updates เอง

อาจเกิดปัญหา “works on my machine”

## 📋 ผลลัพธ์ที่คาดหวัง

เมื่อทำ lab สำเร็จ นักเรียนจะ:
- **🐳 เข้าใจการใช้ Docker**: สามารถใช้ Docker สำหรับ ESP32 development ได้
- **🏗️ เข้าใจโครงสร้าง project**: สามารถสร้างและจัดการ ESP32 project structure
- **📂 Git Management**: เข้าใจการใช้ .gitignore สำหรับ ESP32 development
- **⚙️ ใช้ idf.py ได้**: สามารถใช้คำสั่ง idf.py เบื้องต้นใน Docker environment
- **📊 วิเคราะห์ build output**: เข้าใจ build process และสามารถวิเคราะห์ผลลัพธ์ได้
- **🔧 ปรับแต่ง build**: สามารถแก้ไข CMakeLists.txt เพื่อปรับแต่ง build configuration

## 🛠️ การแก้ไขปัญหาที่พบบ่อย

### ปัญหา: Docker Container ไม่ start
```bash
docker-compose up -d
# Error: Cannot connect to Docker daemon
```
**วิธีแก้**: 
- ตรวจสอบว่า Docker Desktop เปิดอยู่
- รัน `docker --version` เพื่อตรวจสอบการติดตั้ง

### ปัญหา: Cannot access /project directory
```bash
bash: cd: /project: No such file or directory
```
**วิธีแก้**: 
- ตรวจสอบ volume mapping ใน docker-compose.yml
- ตรวจสอบว่าอยู่ใน directory ที่ถูกต้องบน host

### ปัญหา: ESP-IDF not found
```bash
idf.py: command not found
```
**วิธีแก้**: 
- รัน `source $IDF_PATH/export.sh` ใน container
- ตรวจสอบ environment variables

### ปัญหา: Permission denied when building
```bash
Permission denied: cannot create directory 'build'
```
**วิธีแก้**: 
- ตรวจสอบ file permissions บน host system
- ใช้ `chmod 755` ให้กับ project directory

### ปัญหา: Container stops immediately
```bash
docker-compose ps
# Shows container as "Exited"
```
**วิธีแก้**: 
- ตรวจสอบ docker-compose.yml configuration
- ใช้ `docker-compose logs esp32-dev` เพื่อดู error logs

Docker environment ที่ setup ในสัปดาห์นี้จะใช้ต่อเนื่องตลอดเทอม
