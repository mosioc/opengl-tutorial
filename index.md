---
title: اوپن‌جی‌ال
nav_order: 1
permalink: /opengl/
layout: persian
---


# معرفی OpenGL و GLUT/FreeGLUT - درس گرافیک کامپیوتری

## مقدمه
در این درس، ما با مفاهیم پایه‌ای گرافیک کامپیوتری و برنامه‌نویسی گرافیکی با استفاده از OpenGL آشنا خواهیم شد. این مفاهیم پایه‌ای برای درک عمیق‌تر گرافیک سه‌بعدی و پردازش تصویر ضروری هستند.

## OpenGL چیست؟
OpenGL (Open Graphics Library) یک API گرافیکی استاندارد و کراس‌پلتفرم است که برای ایجاد گرافیک دو بعدی و سه بعدی استفاده می‌شود. این کتابخانه به شما امکان می‌دهد تا:
- اشکال هندسی پایه را رسم کنید
- تبدیلات هندسی (ماتریسی) را اعمال کنید
- نورپردازی و سایه‌اندازی انجام دهید
- بافت‌دهی (texturing) را پیاده‌سازی کنید
- برنامه‌های گرافیکی با عملکرد بالا ایجاد کنید

### تاریخچه OpenGL
- توسعه یافته توسط Silicon Graphics در سال 1992
- استاندارد صنعتی برای گرافیک سه‌بعدی
- پشتیبانی از اکثر سیستم‌عامل‌ها و سخت‌افزارها

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

## تمرینات عملی

### تمرین 1: رسم اشکال پایه
1. یک پنجره با رنگ پس‌زمینه سفید ایجاد کنید
2. یک مربع با رنگ قرمز رسم کنید
3. یک دایره با رنگ آبی رسم کنید
4. یک مثلث با رنگ سبز رسم کنید

### تمرین 2: تعامل با کاربر
1. با کلیک ماوس، رنگ اشکال را تغییر دهید
2. با کلیدهای جهت‌دار، اشکال را جابجا کنید
3. با کلید + و -، اندازه اشکال را تغییر دهید

### تمرین 3: انیمیشن
1. یک حرکت نوسانی ساده ایجاد کنید
2. یک حرکت چرخشی پیاده‌سازی کنید
3. یک حرکت مرکب (ترکیبی) بسازید

## مفاهیم پیشرفته (برای مطالعه بیشتر)
1. نورپردازی و سایه‌اندازی
2. بافت‌دهی و نگاشت بافت
3. شیدرها و پردازش گرافیکی
4. بهینه‌سازی عملکرد
5. گرافیک سه‌بعدی پیشرفته

## منابع بیشتر
- [مستندات رسمی OpenGL](https://www.opengl.org/documentation/)
- [مستندات FreeGLUT](https://www.transmissionzero.co.uk/software/freeglut-devel/)
- [آموزش جامع OpenGL](https://learnopengl.com/)
- [کتاب OpenGL Programming Guide](https://www.opengl.org/archives/resources/code/samples/glut_examples/examples/redbook/)
- [مقالات آکادمیک گرافیک کامپیوتری](https://www.siggraph.org/) 