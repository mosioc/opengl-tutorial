---
title: مدیریت ورودی
nav_order: 5
permalink: /input/
layout: persian
---

# مدیریت ورودی کاربر در OpenGL

## مقدمه
در این بخش، با مفاهیم پیشرفته مدیریت ورودی کاربر در OpenGL آشنا می‌شویم. ورودی کاربر یکی از مهم‌ترین بخش‌های هر برنامه گرافیکی است که به کاربر امکان تعامل با برنامه را می‌دهد. در این درس، روش‌های مختلف مدیریت ورودی را بررسی می‌کنیم.

## مفاهیم پایه‌ای ورودی

### 1. انواع ورودی
- **ورودی کیبورد**: شامل کلیدهای معمولی و کلیدهای خاص
- **ورودی ماوس**: شامل کلیک‌ها، حرکت و چرخش
- **ورودی لمسی**: برای دستگاه‌های لمسی
- **ورودی گیم‌پد**: برای کنترل‌های بازی

### 2. سیستم‌های مدیریت ورودی
- **GLUT (OpenGL Utility Toolkit)**
- **GLFW (OpenGL Framework)**
- **SDL (Simple DirectMedia Layer)**
- **SFML (Simple and Fast Multimedia Library)**

## مدیریت کیبورد

### 1. کلیدهای معمولی
```cpp
// ساختار برای نگهداری وضعیت کلیدها
struct KeyState {
    bool pressed;
    bool held;
    bool released;
};

// آرایه برای نگهداری وضعیت کلیدها
KeyState keys[256] = {0};

// تابع callback برای فشردن کلید
void keyboardDown(unsigned char key, int x, int y) {
    keys[key].pressed = true;
    keys[key].held = true;
    
    switch(key) {
        case 27:  // ESC
            exit(0);
            break;
        case 'r':
            // تغییر رنگ به قرمز
            glClearColor(1.0, 0.0, 0.0, 1.0);
            break;
        case 'g':
            // تغییر رنگ به سبز
            glClearColor(0.0, 1.0, 0.0, 1.0);
            break;
        case 'b':
            // تغییر رنگ به آبی
            glClearColor(0.0, 0.0, 1.0, 1.0);
            break;
    }
    glutPostRedisplay();
}

// تابع callback برای رها کردن کلید
void keyboardUp(unsigned char key, int x, int y) {
    keys[key].pressed = false;
    keys[key].released = true;
    glutPostRedisplay();
}

// تابع به‌روزرسانی وضعیت کلیدها
void updateKeyStates() {
    for (int i = 0; i < 256; i++) {
        if (keys[i].released) {
            keys[i].held = false;
            keys[i].released = false;
        }
    }
}
```

### 2. کلیدهای خاص
```cpp
// ساختار برای نگهداری وضعیت کلیدهای خاص
struct SpecialKeyState {
    bool pressed;
    bool held;
    bool released;
};

// آرایه برای نگهداری وضعیت کلیدهای خاص
SpecialKeyState specialKeys[256] = {0};

// تابع callback برای فشردن کلیدهای خاص
void specialKeysDown(int key, int x, int y) {
    specialKeys[key].pressed = true;
    specialKeys[key].held = true;
    
    switch(key) {
        case GLUT_KEY_UP:
            // حرکت به بالا
            y_pos += 0.1;
            break;
        case GLUT_KEY_DOWN:
            // حرکت به پایین
            y_pos -= 0.1;
            break;
        case GLUT_KEY_LEFT:
            // حرکت به چپ
            x_pos -= 0.1;
            break;
        case GLUT_KEY_RIGHT:
            // حرکت به راست
            x_pos += 0.1;
            break;
        case GLUT_KEY_PAGE_UP:
            // چرخش به جلو
            rotation_x += 5.0;
            break;
        case GLUT_KEY_PAGE_DOWN:
            // چرخش به عقب
            rotation_x -= 5.0;
            break;
    }
    glutPostRedisplay();
}

// تابع callback برای رها کردن کلیدهای خاص
void specialKeysUp(int key, int x, int y) {
    specialKeys[key].pressed = false;
    specialKeys[key].released = true;
    glutPostRedisplay();
}

// تابع به‌روزرسانی وضعیت کلیدهای خاص
void updateSpecialKeyStates() {
    for (int i = 0; i < 256; i++) {
        if (specialKeys[i].released) {
            specialKeys[i].held = false;
            specialKeys[i].released = false;
        }
    }
}
```

## مدیریت ماوس

### 1. کلیک‌های ماوس
```cpp
// ساختار برای نگهداری وضعیت ماوس
struct MouseState {
    bool leftButton;
    bool rightButton;
    bool middleButton;
    int x;
    int y;
    int lastX;
    int lastY;
    bool firstMouse;
};

MouseState mouse = {false, false, false, 0, 0, 0, 0, true};

// تابع callback برای کلیک‌های ماوس
void mouse(int button, int state, int x, int y) {
    // به‌روزرسانی وضعیت دکمه‌ها
    switch(button) {
        case GLUT_LEFT_BUTTON:
            mouse.leftButton = (state == GLUT_DOWN);
            break;
        case GLUT_RIGHT_BUTTON:
            mouse.rightButton = (state == GLUT_DOWN);
            break;
        case GLUT_MIDDLE_BUTTON:
            mouse.middleButton = (state == GLUT_DOWN);
            break;
    }
    
    // به‌روزرسانی موقعیت ماوس
    mouse.x = x;
    mouse.y = y;
    
    if (mouse.firstMouse) {
        mouse.lastX = x;
        mouse.lastY = y;
        mouse.firstMouse = false;
    }
    
    // تبدیل مختصات پنجره به مختصات OpenGL
    float x_pos = (float)x / glutGet(GLUT_WINDOW_WIDTH) * 2 - 1;
    float y_pos = 1 - (float)y / glutGet(GLUT_WINDOW_HEIGHT) * 2;
    
    if (mouse.leftButton) {
        // ایجاد یک دایره در محل کلیک
        createCircle(x_pos, y_pos, 0.1);
    }
    
    glutPostRedisplay();
}
```

### 2. حرکت ماوس
```cpp
// تابع callback برای حرکت ماوس
void motion(int x, int y) {
    // محاسبه تغییر موقعیت
    float xoffset = x - mouse.lastX;
    float yoffset = mouse.lastY - y;
    
    // به‌روزرسانی موقعیت قبلی
    mouse.lastX = x;
    mouse.lastY = y;
    
    // تبدیل مختصات پنجره به مختصات OpenGL
    float x_pos = (float)x / glutGet(GLUT_WINDOW_WIDTH) * 2 - 1;
    float y_pos = 1 - (float)y / glutGet(GLUT_WINDOW_HEIGHT) * 2;
    
    if (mouse.leftButton) {
        // چرخش دوربین
        camera.ProcessMouseMovement(xoffset, yoffset);
    }
    
    glutPostRedisplay();
}

// تابع callback برای حرکت ماوس بدون کلیک
void passiveMotion(int x, int y) {
    // به‌روزرسانی موقعیت ماوس
    mouse.x = x;
    mouse.y = y;
    
    if (mouse.firstMouse) {
        mouse.lastX = x;
        mouse.lastY = y;
        mouse.firstMouse = false;
    }
    
    glutPostRedisplay();
}
```

### 3. چرخش چرخ ماوس
```cpp
// تابع callback برای چرخش چرخ ماوس
void mouseWheel(int button, int dir, int x, int y) {
    if (dir > 0) {
        // بزرگنمایی
        camera.ProcessMouseScroll(1.0);
    } else {
        // کوچک‌نمایی
        camera.ProcessMouseScroll(-1.0);
    }
    glutPostRedisplay();
}
```

## مدیریت ورودی لمسی

### 1. تشخیص لمس
```cpp
// ساختار برای نگهداری وضعیت لمس
struct TouchState {
    bool active;
    float x;
    float y;
    float pressure;
};

TouchState touch = {false, 0.0, 0.0, 0.0};

// تابع callback برای لمس
void touchFunc(int x, int y, int pressure) {
    touch.active = true;
    touch.x = (float)x / glutGet(GLUT_WINDOW_WIDTH) * 2 - 1;
    touch.y = 1 - (float)y / glutGet(GLUT_WINDOW_HEIGHT) * 2;
    touch.pressure = (float)pressure / 255.0;
    
    glutPostRedisplay();
}

// تابع callback برای پایان لمس
void touchUpFunc(int x, int y) {
    touch.active = false;
    glutPostRedisplay();
}
```

## مدیریت گیم‌پد

### 1. تشخیص گیم‌پد
```cpp
// ساختار برای نگهداری وضعیت گیم‌پد
struct GamepadState {
    bool connected;
    float leftStickX;
    float leftStickY;
    float rightStickX;
    float rightStickY;
    float leftTrigger;
    float rightTrigger;
    bool buttons[16];
};

GamepadState gamepad = {false, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, {false}};

// تابع callback برای گیم‌پد
void gamepadFunc(unsigned int buttonMask, int x, int y, int z) {
    gamepad.connected = true;
    
    // به‌روزرسانی وضعیت دکمه‌ها
    for (int i = 0; i < 16; i++) {
        gamepad.buttons[i] = (buttonMask & (1 << i)) != 0;
    }
    
    // به‌روزرسانی وضعیت استیک‌ها
    gamepad.leftStickX = (float)x / 32767.0;
    gamepad.leftStickY = (float)y / 32767.0;
    gamepad.rightStickX = (float)z / 32767.0;
    
    glutPostRedisplay();
}
```

## مثال کامل

```cpp
#include <GL/glut.h>
#include <vector>
#include <cmath>

// ساختارهای داده
struct Circle {
    float x, y;
    float radius;
    float r, g, b;
};

// متغیرهای سراسری
std::vector<Circle> circles;
float cameraDistance = 5.0;
float cameraRotation = 0.0;

// تابع رسم دایره
void drawCircle(float x, float y, float radius, float r, float g, float b) {
    glColor3f(r, g, b);
    glBegin(GL_TRIANGLE_FAN);
    glVertex2f(x, y);
    for (int i = 0; i <= 360; i += 10) {
        float angle = i * M_PI / 180.0;
        glVertex2f(x + radius * cos(angle), y + radius * sin(angle));
    }
    glEnd();
}

// تابع رسم صحنه
void display() {
    glClear(GL_COLOR_BUFFER_BIT);
    
    // رسم همه دایره‌ها
    for (const auto& circle : circles) {
        drawCircle(circle.x, circle.y, circle.radius, circle.r, circle.g, circle.b);
    }
    
    glutSwapBuffers();
}

// تابع به‌روزرسانی
void update(int value) {
    // به‌روزرسانی وضعیت کلیدها
    updateKeyStates();
    updateSpecialKeyStates();
    
    // به‌روزرسانی دوربین
    if (specialKeys[GLUT_KEY_LEFT].held) {
        cameraRotation -= 1.0;
    }
    if (specialKeys[GLUT_KEY_RIGHT].held) {
        cameraRotation += 1.0;
    }
    if (specialKeys[GLUT_KEY_UP].held) {
        cameraDistance -= 0.1;
    }
    if (specialKeys[GLUT_KEY_DOWN].held) {
        cameraDistance += 0.1;
    }
    
    glutPostRedisplay();
    glutTimerFunc(16, update, 0); // حدود 60 FPS
}

// تابع تغییر اندازه پنجره
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
    glutCreateWindow("مدیریت ورودی کاربر - درس گرافیک کامپیوتری");
    
    glClearColor(0.0, 0.0, 0.0, 1.0);
    
    // ثبت توابع callback
    glutDisplayFunc(display);
    glutReshapeFunc(reshape);
    glutKeyboardFunc(keyboardDown);
    glutKeyboardUpFunc(keyboardUp);
    glutSpecialFunc(specialKeysDown);
    glutSpecialUpFunc(specialKeysUp);
    glutMouseFunc(mouse);
    glutMotionFunc(motion);
    glutPassiveMotionFunc(passiveMotion);
    glutMouseWheelFunc(mouseWheel);
    glutTimerFunc(0, update, 0);
    
    glutMainLoop();
    return 0;
}
```

## نکات مهم
1. همیشه بعد از تغییر وضعیت، `glutPostRedisplay()` را فراخوانی کنید
2. برای تبدیل مختصات پنجره به مختصات OpenGL از فرمول‌های مناسب استفاده کنید
3. از متغیرهای سراسری برای نگهداری وضعیت برنامه استفاده کنید
4. برای کلیدهای خاص از `glutSpecialFunc` استفاده کنید
5. همیشه وضعیت ورودی‌ها را در هر فریم به‌روز کنید
6. از سیستم‌های مدیریت ورودی مدرن مانند GLFW یا SDL برای پروژه‌های بزرگ استفاده کنید

## منابع بیشتر
- [مستندات GLUT](https://www.opengl.org/resources/libraries/glut/)
- [راهنمای ورودی کاربر](https://www.opengl.org/wiki/OpenGL_Utility_Toolkit)
- [مستندات GLFW](https://www.glfw.org/documentation.html)
- [مستندات SDL](https://wiki.libsdl.org/)
- [مقالات GDC درباره مدیریت ورودی](https://www.gdc.com/)
- [کتاب Game Programming Patterns](https://gameprogrammingpatterns.com/) 
