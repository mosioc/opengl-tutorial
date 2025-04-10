---
title: مدیریت پنجره
nav_order: 3
permalink: /window/
layout: persian
---


# مدیریت پنجره در OpenGL

## مقدمه
در این بخش، ما با مفاهیم پایه‌ای مدیریت پنجره در OpenGL آشنا خواهیم شد. پنجره‌ها پایه‌ای‌ترین عنصر رابط کاربری در برنامه‌های گرافیکی هستند و درک نحوه مدیریت آنها برای ایجاد برنامه‌های گرافیکی تعاملی ضروری است.

## مفاهیم پایه‌ای پنجره

### 1. ساختار پنجره
- **عنوان پنجره**: نمایش نام برنامه
- **نوار عنوان**: شامل دکمه‌های کنترل پنجره
- **محدوده رسم**: ناحیه‌ای که محتوای گرافیکی در آن نمایش داده می‌شود
- **حاشیه‌ها**: فاصله بین محدوده رسم و لبه‌های پنجره

### 2. دستگاه مختصات پنجره
- **مختصات صفحه**: بر حسب پیکسل، با مبدأ در گوشه بالا-چپ
- **مختصات OpenGL**: بر حسب واحد نرمال شده (-1 تا 1)
- **تبدیل بین دستگاه‌های مختصات**: تبدیل از مختصات صفحه به مختصات OpenGL

## ایجاد و مدیریت پنجره با GLUT

### 1. مقداردهی اولیه GLUT
```cpp
#include <GL/glut.h>

int main(int argc, char** argv) {
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB);
    glutInitWindowSize(800, 600);
    glutInitWindowPosition(100, 100);
    glutCreateWindow("مدیریت پنجره در OpenGL");
    glutMainLoop();
    return 0;
}
```

### 2. تنظیمات نمایش
```cpp
// تنظیم حالت نمایش
glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB | GLUT_DEPTH);

// تنظیم اندازه پنجره
glutInitWindowSize(width, height);

// تنظیم موقعیت پنجره
glutInitWindowPosition(x, y);
```

### 3. مدیریت تغییر اندازه پنجره
```cpp
void reshape(int w, int h) {
    // تنظیم viewport
    glViewport(0, 0, w, h);
    
    // تنظیم ماتریس برجستگی
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    gluOrtho2D(-1.0, 1.0, -1.0, 1.0);
    
    // تنظیم ماتریس مدل-نما
    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();
}
```

## ویژگی‌های پیشرفته پنجره

### 1. حالت‌های نمایش
```cpp
// حالت تمام صفحه
glutFullScreen();

// حالت شناور
glutSetWindowTitle("پنجره شناور");

// حالت مینیمایز
glutIconifyWindow();
```

### 2. مدیریت چند پنجره
```cpp
// ایجاد پنجره اصلی
int mainWindow = glutCreateWindow("پنجره اصلی");

// ایجاد پنجره فرعی
int subWindow = glutCreateSubWindow(mainWindow, x, y, width, height);

// تغییر پنجره فعال
glutSetWindow(subWindow);
```

### 3. منوها و زیرمنوها
```cpp
// ایجاد منوی اصلی
int menu = glutCreateMenu(menuCallback);

// اضافه کردن گزینه‌ها به منو
glutAddMenuEntry("گزینه 1", 1);
glutAddMenuEntry("گزینه 2", 2);

// اتصال منو به دکمه راست ماوس
glutAttachMenu(GLUT_RIGHT_BUTTON);
```

## مدیریت رویدادهای پنجره

### 1. رویدادهای ماوس
```cpp
void mouse(int button, int state, int x, int y) {
    if (button == GLUT_LEFT_BUTTON && state == GLUT_DOWN) {
        // تبدیل مختصات ماوس به مختصات OpenGL
        float glX = (2.0f * x) / glutGet(GLUT_WINDOW_WIDTH) - 1.0f;
        float glY = 1.0f - (2.0f * y) / glutGet(GLUT_WINDOW_HEIGHT);
        // ...
    }
}
```

### 2. رویدادهای کیبورد
```cpp
void keyboard(unsigned char key, int x, int y) {
    switch (key) {
        case 27: // ESC
            exit(0);
            break;
        case 'f':
            glutFullScreen();
            break;
        // ...
    }
}
```

### 3. رویدادهای حرکت ماوس
```cpp
void motion(int x, int y) {
    // پردازش حرکت ماوس
}

void passiveMotion(int x, int y) {
    // پردازش حرکت ماوس بدون کلیک
}
```

## کد نمونه کامل

```cpp
#include <GL/glut.h>
#include <iostream>

// متغیرهای سراسری
int windowWidth = 800;
int windowHeight = 600;
bool isFullScreen = false;

// تابع callback برای رسم
void display() {
    glClear(GL_COLOR_BUFFER_BIT);
    
    // رسم محتوا
    glBegin(GL_QUADS);
        glColor3f(1.0, 0.0, 0.0);
        glVertex2f(-0.5, -0.5);
        glColor3f(0.0, 1.0, 0.0);
        glVertex2f(0.5, -0.5);
        glColor3f(0.0, 0.0, 1.0);
        glVertex2f(0.5, 0.5);
        glColor3f(1.0, 1.0, 0.0);
        glVertex2f(-0.5, 0.5);
    glEnd();
    
    glutSwapBuffers();
}

// تابع callback برای تغییر اندازه پنجره
void reshape(int w, int h) {
    windowWidth = w;
    windowHeight = h;
    
    glViewport(0, 0, w, h);
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    gluOrtho2D(-1.0, 1.0, -1.0, 1.0);
    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();
}

// تابع callback برای کیبورد
void keyboard(unsigned char key, int x, int y) {
    switch (key) {
        case 27: // ESC
            exit(0);
            break;
        case 'f':
            isFullScreen = !isFullScreen;
            if (isFullScreen) {
                glutFullScreen();
            } else {
                glutReshapeWindow(windowWidth, windowHeight);
            }
            break;
    }
}

// تابع callback برای منو
void menu(int value) {
    switch (value) {
        case 1:
            glutSetWindowTitle("گزینه 1 انتخاب شد");
            break;
        case 2:
            glutSetWindowTitle("گزینه 2 انتخاب شد");
            break;
        case 3:
            exit(0);
            break;
    }
}

int main(int argc, char** argv) {
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB);
    glutInitWindowSize(windowWidth, windowHeight);
    glutInitWindowPosition(100, 100);
    glutCreateWindow("مدیریت پنجره در OpenGL");
    
    // تنظیم توابع callback
    glutDisplayFunc(display);
    glutReshapeFunc(reshape);
    glutKeyboardFunc(keyboard);
    
    // ایجاد منو
    int menuId = glutCreateMenu(menu);
    glutAddMenuEntry("گزینه 1", 1);
    glutAddMenuEntry("گزینه 2", 2);
    glutAddMenuEntry("خروج", 3);
    glutAttachMenu(GLUT_RIGHT_BUTTON);
    
    glutMainLoop();
    return 0;
}
```

## منابع بیشتر
- [مستندات GLUT](https://www.opengl.org/resources/libraries/glut/)
- [OpenGL Window Management](https://www.khronos.org/opengl/wiki/Creating_an_OpenGL_Context)
- [GLUT Programming Guide](https://www.opengl.org/archives/resources/code/samples/glut_examples/examples/redbook/)
- [مقالات آکادمیک SIGGRAPH](https://www.siggraph.org/) 
