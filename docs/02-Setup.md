---
title: معرفی و راه‌اندازی
nav_order: 2
permalink: /setup/
layout: persian
---

# معرفی OpenGL و GLUT/FreeGLUT

## مقدمه
در این درس، ما با مفاهیم پایه‌ای گرافیک کامپیوتری و برنامه‌نویسی گرافیکی با استفاده از OpenGL آشنا خواهیم شد. این مفاهیم پایه‌ای برای درک عمیق‌تر گرافیک سه‌بعدی و پردازش تصویر ضروری هستند.

## OpenGL چیست؟
OpenGL (Open Graphics Library) یک API گرافیکی استاندارد و کراس‌پلتفرم است که برای ایجاد گرافیک دو بعدی و سه بعدی استفاده می‌شود. این کتابخانه به شما امکان می‌دهد تا:
- اشکال هندسی پایه را رسم کنید
- تبدیلات هندسی (ماتریسی) را اعمال کنید
- نورپردازی و سایه‌اندازی انجام دهید
- بافت‌دهی (texturing) را پیاده‌سازی کنید
- برنامه‌های گرافیکی با عملکرد بالا ایجاد کنید

## ویژگی‌های اصلی OpenGL
- **پلتفرم متقاطع**: روی انواع سیستم‌های عامل از جمله Windows، Linux، macOS، iOS و Android قابل اجراست.
- **مستقل از سخت‌افزار**: با انواع کارت‌های گرافیک سازگار است.
- **حالت بلافصل (Immediate Mode)** و **حالت هسته (Core Mode)**: دو رویکرد متفاوت برای برنامه‌نویسی OpenGL.
- **Pipeline رندرینگ قابل برنامه‌ریزی**: امکان کنترل دقیق روند پردازش گرافیکی.

## مفاهیم پایه در OpenGL

### سیستم مختصات
در OpenGL از یک سیستم مختصات راست‌گرد سه‌بعدی استفاده می‌شود:
- محور X: افقی (از چپ به راست)
- محور Y: عمودی (از پایین به بالا)
- محور Z: عمق (از نزدیک به دور)

### نقطه، خط و چندضلعی
OpenGL اشکال ابتدایی را با استفاده از نقاط، خطوط و مثلث‌ها تعریف می‌کند. هر شکل پیچیده‌ای از ترکیب این اشکال اولیه ساخته می‌شود.

### پنجره و Viewport
- **پنجره**: فضای نمایش در صفحه نمایش واقعی.
- **Viewport**: منطقه‌ای از پنجره که در آن تصویر OpenGL رندر می‌شود.

### خط لوله گرافیکی (Graphics Pipeline)
خط لوله گرافیکی OpenGL شامل مراحل زیر است:
1. **Vertex Processing**: پردازش داده‌های رأس (موقعیت، رنگ، بافت و...)
2. **Primitive Assembly**: جمع‌آوری نقاط، خطوط و مثلث‌ها
3. **Rasterization**: تبدیل اشکال به پیکسل‌ها
4. **Fragment Processing**: پردازش پیکسل‌های تولید شده
5. **Framebuffer Operations**: عملیات نهایی روی پیکسل‌ها قبل از نمایش

## مزایای استفاده از OpenGL
- استاندارد باز و گسترده
- پشتیبانی گسترده توسط سخت‌افزارهای مختلف
- اجرای سریع به دلیل بهینه‌سازی سخت‌افزاری
- مستندات و منابع آموزشی فراوان
- استفاده در صنایع مختلف مانند بازی‌سازی، شبیه‌سازی، ویرایش ویدیو و...

## تاریخچه مختصر نسخه‌های OpenGL
- **OpenGL 1.0** (1992): نسخه اولیه
- **OpenGL 2.0** (2004): معرفی زبان شیدر GLSL
- **OpenGL 3.0** (2008): معرفی پروفایل‌های Core و Compatibility
- **OpenGL 4.0** (2010): پشتیبانی از شیدرهای tessellation
- **OpenGL 4.6** (2017): آخرین نسخه اصلی منتشر شده


## GLUT/FreeGLUT چیست؟
GLUT (OpenGL Utility Toolkit) و FreeGLUT (نسخه آزاد GLUT) کتابخانه‌های کمکی هستند که کار با OpenGL را ساده‌تر می‌کنند. این کتابخانه‌ها امکانات زیر را فراهم می‌کنند:
- ایجاد و مدیریت پنجره‌ها
- مدیریت ورودی کاربر (کیبورد و ماوس)
- ایجاد منوها و زیرمنوها
- مدیریت رویدادها و callback‌ها
- پشتیبانی از تایمرها برای انیمیشن
- مدیریت حالت‌های نمایش مختلف

## نصب و راه‌اندازی

### در ویندوز:
1. نصب MinGW (کامپایلر C++)
   - دانلود و نصب MinGW
   - اضافه کردن مسیر bin به متغیر PATH
2. نصب FreeGLUT
   - دانلود FreeGLUT
   - کپی فایل‌های DLL به مسیر سیستم
   - کپی فایل‌های هدر و کتابخانه به مسیر MinGW
3. تنظیم متغیرهای محیطی
   - اضافه کردن مسیرهای لازم به PATH
   - تنظیم متغیرهای INCLUDE و LIB

### در لینوکس:
```bash
sudo apt-get update
sudo apt-get install build-essential
sudo apt-get install freeglut3-dev
```

### در مک:
```bash
brew install freeglut
```

## مفاهیم پایه‌ای گرافیک کامپیوتری

### 1. سیستم مختصات
- دستگاه مختصات دکارتی
- تبدیل بین دستگاه‌های مختصات
- دستگاه مختصات نرمال شده (-1 تا 1)

### 2. تبدیلات هندسی
- انتقال (Translation)
- چرخش (Rotation)
- مقیاس‌بندی (Scaling)
- ترکیب تبدیلات

### 3. رنگ‌ها و مدل‌های رنگی
- مدل RGB
- مدل HSV
- آلفا بلندینگ
- عمق بافر

## یک برنامه ساده با GLUT

```cpp
#include <GL/glut.h>

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
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB);
    glutInitWindowSize(800, 600);
    glutCreateWindow("اولین برنامه OpenGL - درس گرافیک کامپیوتری");
    
    glutDisplayFunc(display);
    glutReshapeFunc(reshape);
    
    glutMainLoop();
    return 0;
}
```

## ساختار یک برنامه GLUT
1. **مقداردهی اولیه**: `glutInit()`
   - تنظیم پارامترهای اولیه GLUT
   - مقداردهی به متغیرهای سراسری

2. **تنظیم حالت نمایش**: `glutInitDisplayMode()`
   - انتخاب بافر رنگی
   - تنظیم عمق بافر
   - انتخاب حالت نمایش

3. **ایجاد پنجره**: `glutCreateWindow()`
   - تعیین اندازه پنجره
   - تنظیم عنوان پنجره

4. **تعریف توابع callback**: 
   - `glutDisplayFunc()`: برای رسم
   - `glutReshapeFunc()`: برای تغییر اندازه پنجره
   - `glutKeyboardFunc()`: برای ورودی کیبورد
   - `glutMouseFunc()`: برای ورودی ماوس
   - `glutIdleFunc()`: برای انیمیشن

5. **شروع حلقه اصلی**: `glutMainLoop()`
   - مدیریت رویدادها
   - فراخوانی توابع callback

## توابع callback مهم و کاربرد آنها

### 1. تابع نمایش (Display)
```cpp
void display(void) {
    // کد رسم
}
```
- فراخوانی می‌شود هر زمان که نیاز به بازسازی تصویر باشد
- برای رسم اشکال و اجرای تغییرات گرافیکی استفاده می‌شود

### 2. تابع تغییر اندازه (Reshape)
```cpp
void reshape(int w, int h) {
    // کد تنظیم viewport و ماتریس‌ها
}
```
- هنگام تغییر اندازه پنجره فراخوانی می‌شود
- برای تنظیم نسبت تصویر و تبدیلات استفاده می‌شود

### 3. تابع کیبورد (Keyboard)
```cpp
void keyboard(unsigned char key, int x, int y) {
    // کد مدیریت کلیدها
}
```
- برای دریافت ورودی کیبورد استفاده می‌شود
- امکان کنترل برنامه با کلیدها را فراهم می‌کند

### 4. تابع ماوس (Mouse)
```cpp
void mouse(int button, int state, int x, int y) {
    // کد مدیریت کلیک‌ها
}
```
- برای دریافت ورودی ماوس استفاده می‌شود
- امکان تعامل با اشکال را فراهم می‌کند


# راه‌اندازی محیط توسعه OpenGL با GLUT/FreeGLUT

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
