---
title: راه‌اندازی
nav_order: 2
permalink: /setup/
layout: persian
---

# راه‌اندازی محیط توسعه OpenGL با GLUT/FreeGLUT - درس گرافیک کامپیوتری

## مقدمه
در این بخش، ما با نحوه راه‌اندازی محیط توسعه OpenGL برای درس گرافیک کامپیوتری آشنا خواهیم شد. این محیط توسعه شامل ابزارهای لازم برای کامپایل، اجرا و اشکال‌زدایی برنامه‌های گرافیکی است.

## پیش‌نیازها
- سیستم عامل ویندوز، لینوکس یا مک
- دسترسی به اینترنت برای دانلود نرم‌افزارها
- حداقل 2GB فضای خالی
- پردازنده با پشتیبانی از OpenGL 3.3 یا بالاتر
- کارت گرافیک با درایورهای به‌روز
- حداقل 4GB RAM (8GB توصیه می‌شود)

## نصب در ویندوز

### 1. نصب MinGW
1. به [سایت MinGW](https://sourceforge.net/projects/mingw/) بروید
2. فایل نصب‌کننده را دانلود کنید
3. در زمان نصب، پکیج‌های زیر را انتخاب کنید:
   - mingw32-base
   - mingw32-gcc-g++
   - mingw32-gcc-objc
   - msys-base
   - mingw32-make
4. مسیر MinGW را به متغیر PATH اضافه کنید:
   - به Control Panel > System > Advanced System Settings > Environment Variables بروید
   - در System Variables، PATH را پیدا کنید و Edit را بزنید
   - مسیر `C:\MinGW\bin` را اضافه کنید
5. تست نصب:
   ```bash
   g++ --version
   ```

### 2. نصب FreeGLUT
1. به [سایت FreeGLUT](https://www.transmissionzero.co.uk/software/freeglut-devel/) بروید
2. فایل `freeglut-MSVC.zip` را دانلود کنید
3. محتویات فایل را در مسیر مناسب استخراج کنید
4. فایل‌های زیر را کپی کنید:
   - `freeglut.dll` به `C:\Windows\System32`
   - `freeglut.h` به `C:\MinGW\include\GL`
   - `freeglut.lib` به `C:\MinGW\lib`
5. تست نصب:
   ```bash
   g++ -v
   ```

### 3. نصب ویرایشگر کد (Visual Studio Code)
1. دانلود و نصب [Visual Studio Code](https://code.visualstudio.com/)
2. نصب افزونه‌های ضروری:
   - C/C++ Extension Pack
   - OpenGL Support
   - Code Runner
   - C++ Intellisense
3. تنظیمات پیشنهادی:
   ```json
   {
       "C_Cpp.default.includePath": [
           "${workspaceFolder}/**",
           "C:/MinGW/include"
       ],
       "C_Cpp.default.compilerPath": "C:/MinGW/bin/g++.exe"
   }
   ```

### 4. تست نصب
1. یک فایل `test.cpp` با محتوای زیر ایجاد کنید:

```cpp
#include <GL/glut.h>
#include <iostream>

// تابع callback برای رسم
void display() {
    glClear(GL_COLOR_BUFFER_BIT);
    
    // رسم یک مثلث رنگی
    glBegin(GL_TRIANGLES);
        glColor3f(1.0, 0.0, 0.0);  // قرمز
        glVertex2f(-0.5, -0.5);
        glColor3f(0.0, 1.0, 0.0);  // سبز
        glVertex2f(0.5, -0.5);
        glColor3f(0.0, 0.0, 1.0);  // آبی
        glVertex2f(0.0, 0.5);
    glEnd();
    
    glutSwapBuffers();
}

// تابع callback برای تغییر اندازه پنجره
void reshape(int w, int h) {
    glViewport(0, 0, w, h);
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    gluOrtho2D(-1.0, 1.0, -1.0, 1.0);
    glMatrixMode(GL_MODELVIEW);
}

int main(int argc, char** argv) {
    // مقداردهی اولیه GLUT
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB);
    glutInitWindowSize(800, 600);
    glutCreateWindow("تست نصب OpenGL - درس گرافیک کامپیوتری");
    
    // تنظیم توابع callback
    glutDisplayFunc(display);
    glutReshapeFunc(reshape);
    
    // شروع حلقه اصلی
    glutMainLoop();
    return 0;
}
```

2. کامپایل و اجرای برنامه:
```bash
g++ test.cpp -o test -lopengl32 -lglu32 -lfreeglut
./test
```

## نصب در لینوکس (Ubuntu/Debian)

### 1. نصب کامپایلر و کتابخانه‌ها
```bash
# به‌روزرسانی سیستم
sudo apt-get update
sudo apt-get upgrade

# نصب ابزارهای پایه
sudo apt-get install build-essential
sudo apt-get install cmake

# نصب کتابخانه‌های OpenGL
sudo apt-get install freeglut3-dev
sudo apt-get install libglew-dev
sudo apt-get install libglm-dev
```

### 2. نصب ویرایشگر کد
```bash
# نصب Visual Studio Code
sudo snap install code --classic

# یا نصب CLion
sudo snap install clion --classic
```

### 3. تست نصب
همان کد تست بالا را استفاده کنید و با دستور زیر کامپایل کنید:
```bash
g++ test.cpp -o test -lglut -lGL -lGLU
./test
```

## نصب در مک

### 1. نصب Xcode Command Line Tools
```bash
xcode-select --install
```

### 2. نصب Homebrew
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 3. نصب FreeGLUT و سایر ابزارها
```bash
brew install freeglut
brew install glew
brew install glm
```

### 4. نصب ویرایشگر کد
```bash
brew install --cask visual-studio-code
# یا
brew install --cask clion
```

### 5. تست نصب
همان کد تست بالا را استفاده کنید و با دستور زیر کامپایل کنید:
```bash
g++ test.cpp -o test -framework OpenGL -framework GLUT
./test
```

## رفع مشکلات رایج

### 1. خطای "freeglut.dll not found"
- مطمئن شوید که `freeglut.dll` در مسیر سیستم یا کنار فایل اجرایی قرار دارد
- در ویندوز، فایل را در `C:\Windows\System32` کپی کنید
- بررسی کنید که مسیر DLL در PATH سیستم قرار دارد

### 2. خطای "GL/glut.h: No such file or directory"
- مسیر include را درست تنظیم کنید
- مطمئن شوید که FreeGLUT درست نصب شده است
- بررسی کنید که فایل‌های هدر در مسیر درست کپی شده‌اند

### 3. خطای لینک
- مطمئن شوید که همه کتابخانه‌های مورد نیاز را لینک کرده‌اید
- در ویندوز: `-lopengl32 -lglu32 -lfreeglut`
- در لینوکس: `-lglut -lGL -lGLU`
- در مک: `-framework OpenGL -framework GLUT`

### 4. خطای "OpenGL version not supported"
- به‌روزرسانی درایورهای کارت گرافیک
- بررسی پشتیبانی OpenGL در سیستم
- استفاده از نسخه سازگار OpenGL

## محیط توسعه پیشنهادی

### 1. Visual Studio Code
- سبک و سریع
- پشتیبانی از افزونه‌های متنوع
- تنظیمات قابل شخصی‌سازی
- پشتیبانی از Git

### 2. CLion
- IDE قدرتمند برای C++
- پشتیبانی از CMake
- ابزارهای اشکال‌زدایی پیشرفته
- رابط کاربری حرفه‌ای

### 3. Code::Blocks
- سبک و ساده
- مناسب برای مبتدیان
- پشتیبانی از MinGW
- رابط کاربری کاربرپسند

### 4. Dev-C++
- مناسب برای ویندوز
- نصب و راه‌اندازی آسان
- مناسب برای پروژه‌های کوچک
- پشتیبانی از MinGW

## تنظیمات پیشنهادی برای محیط توسعه

### 1. Visual Studio Code
```json
{
    "C_Cpp.default.includePath": [
        "${workspaceFolder}/**",
        "C:/MinGW/include"
    ],
    "C_Cpp.default.compilerPath": "C:/MinGW/bin/g++.exe",
    "C_Cpp.default.cppStandard": "c++17",
    "C_Cpp.default.intelliSenseMode": "windows-gcc-x86"
}
```

### 2. CMake (برای پروژه‌های بزرگتر)
```cmake
cmake_minimum_required(VERSION 3.10)
project(OpenGLProject)

find_package(OpenGL REQUIRED)
find_package(GLUT REQUIRED)

add_executable(${PROJECT_NAME} main.cpp)
target_link_libraries(${PROJECT_NAME} OpenGL::GL GLUT::GLUT)
```

## منابع بیشتر
- [راهنمای نصب MinGW](https://sourceforge.net/projects/mingw/)
- [راهنمای نصب FreeGLUT](https://www.transmissionzero.co.uk/software/freeglut-devel/)
- [مستندات OpenGL](https://www.opengl.org/documentation/)
- [راهنمای Visual Studio Code برای C++](https://code.visualstudio.com/docs/cpp/cpp-ide)
- [مقالات آکادمیک SIGGRAPH](https://www.siggraph.org/)
- [OpenGL Programming Guide (Red Book)](https://www.opengl.org/archives/resources/code/samples/glut_examples/examples/redbook/) 