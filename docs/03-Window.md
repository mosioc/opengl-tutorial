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

## تنظیمات Viewport (دید)

در OpenGL، viewport تعیین می‌کند که چه بخشی از پنجره برای رندرینگ استفاده می‌شود. با استفاده از تابع `glViewport` می‌توانید ناحیه نمایش را تنظیم کنید.

### تابع glViewport

```cpp
void glViewport(GLint x, GLint y, GLsizei width, GLsizei height);
```

پارامترها:
- `x, y`: مختصات گوشه پایین-چپ viewport نسبت به پنجره
- `width, height`: عرض و ارتفاع viewport به پیکسل

### مدیریت تغییر اندازه پنجره

یکی از چالش‌های پنجره‌ها، مدیریت تغییر اندازه آن‌ها است. زمانی که کاربر اندازه پنجره را تغییر می‌دهد، باید viewport را نیز متناسب با آن تنظیم کنید:

```cpp
void reshapeCallback(int width, int height) {
    // تنظیم viewport برای تطابق با اندازه جدید پنجره
    glViewport(0, 0, width, height);
    
    // تنظیم ماتریس پروجکشن
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    
    // حفظ نسبت تصویر (aspect ratio)
    double aspectRatio = (double)width / (double)height;
    if (width >= height) {
        gluOrtho2D(-aspectRatio, aspectRatio, -1.0, 1.0);
    } else {
        gluOrtho2D(-1.0, 1.0, -1.0/aspectRatio, 1.0/aspectRatio);
    }
    
    // بازگشت به ماتریس مدل-ویو
    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();
}
```

### نکات مهم:

1. **حفظ نسبت تصویر**: بدون تنظیم صحیح نسبت تصویر، اشکال ممکن است هنگام تغییر اندازه پنجره کشیده شوند.

2. **ثبت callback**: این تابع باید با استفاده از `glutReshapeFunc` به عنوان callback تغییر اندازه ثبت شود:
   ```cpp
   glutReshapeFunc(reshapeCallback);
   ```

3. **زمان‌بندی**: تابع `glViewport` معمولاً در تابع reshape فراخوانی می‌شود، اما می‌توانید در هر زمانی آن را فراخوانی کنید تا ناحیه رندرینگ را تغییر دهید.

### مثال کاربردی: چندین viewport

گاهی ممکن است بخواهید چندین viewport در یک پنجره داشته باشید (مثلاً برای نمایش یک شیء از چندین زاویه):

```cpp
void display() {
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    
    // viewport بالا-چپ (نمای بالا)
    glViewport(0, height/2, width/2, height/2);
    drawObjectTopView();
    
    // viewport بالا-راست (نمای جلو)
    glViewport(width/2, height/2, width/2, height/2);
    drawObjectFrontView();
    
    // viewport پایین-چپ (نمای کنار)
    glViewport(0, 0, width/2, height/2);
    drawObjectSideView();
    
    // viewport پایین-راست (نمای پرسپکتیو)
    glViewport(width/2, 0, width/2, height/2);
    drawObjectPerspectiveView();
    
    glutSwapBuffers();
}
```

## پرچم‌های نمایش (Display Flags)

در زمان ایجاد پنجره در OpenGL با GLUT، می‌توانید رفتار و ویژگی‌های پنجره را با پرچم‌های نمایش مختلف تعیین کنید. این پرچم‌ها به تابع `glutInitDisplayMode` ارسال می‌شوند.

### پرچم‌های پایه

```cpp
glutInitDisplayMode(GLUT_RGBA | GLUT_DOUBLE | GLUT_DEPTH);
```

پرچم‌های اصلی:

| پرچم | توضیح |
| --- | --- |
| `GLUT_RGB` یا `GLUT_RGBA` | استفاده از مدل رنگ RGB (یا RGBA با کانال آلفا) |
| `GLUT_INDEX` | استفاده از مدل رنگ indexed color (کمتر استفاده می‌شود) |
| `GLUT_SINGLE` | استفاده از یک بافر نمایش |
| `GLUT_DOUBLE` | استفاده از بافر دوگانه برای انیمیشن روان‌تر |
| `GLUT_DEPTH` | فعال‌سازی بافر عمق برای رندرینگ سه‌بعدی |
| `GLUT_STENCIL` | فعال‌سازی بافر استنسیل برای جلوه‌های خاص |
| `GLUT_ACCUM` | فعال‌سازی بافر انباشت برای جلوه‌های پیشرفته |

### انتخاب بافر مناسب

انتخاب صحیح بافرها بر کارایی و کیفیت تصویر تأثیر مستقیم دارد:

1. **بافر دوگانه (Double Buffer)**:
   برای برنامه‌های با انیمیشن، معمولاً از بافر دوگانه استفاده می‌شود تا از پدیده تیرگی (flickering) جلوگیری شود:
   ```cpp
   glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB);
   // در انتهای تابع display:
   glutSwapBuffers();
   ```

2. **بافر عمق (Depth Buffer)**:
   برای رندرینگ صحیح اشیاء سه‌بعدی ضروری است:
   ```cpp
   glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB | GLUT_DEPTH);
   // و در تابع display یا init:
   glEnable(GL_DEPTH_TEST);
   ```

3. **بافر استنسیل (Stencil Buffer)**:
   برای جلوه‌های خاص مانند سایه‌ها، انعکاس‌ها و برش‌ها:
   ```cpp
   glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB | GLUT_DEPTH | GLUT_STENCIL);
   ```

### تنظیم وضوح رنگ

برای تنظیم جزئیات بیشتر، می‌توانید از توابع اضافی استفاده کنید:

```cpp
// تنظیم وضوح رنگ (معمولاً قبل از glutInit)
glutInitDisplayString("rgba=8 depth=16 double");
```

این روش انعطاف‌پذیری بیشتری نسبت به `glutInitDisplayMode` فراهم می‌کند.

### مثال کامل

```cpp
int main(int argc, char** argv) {
    glutInit(&argc, argv);
    
    // درخواست بافر دوگانه، رنگ RGBA و بافر عمق
    glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGBA | GLUT_DEPTH);
    
    // تنظیم موقعیت و اندازه پنجره
    glutInitWindowPosition(100, 100);
    glutInitWindowSize(800, 600);
    
    // ایجاد پنجره با عنوان مشخص
    glutCreateWindow("برنامه OpenGL من");
    
    // فعال‌سازی تست عمق
    glEnable(GL_DEPTH_TEST);
    
    // تنظیم توابع callback
    glutDisplayFunc(display);
    glutReshapeFunc(reshape);
    
    // شروع حلقه اصلی
    glutMainLoop();
    
    return 0;
}
```

### نکات پیشرفته

- **بافر نمونه‌برداری چندگانه (Multisampling)**: برای آنتی‌آلیاسینگ (anti-aliasing) می‌توانید از `GLUT_MULTISAMPLE` استفاده کنید.
- **سازگاری با سیستم‌عامل**: برخی ترکیبات بافر ممکن است در تمام سیستم‌ها پشتیبانی نشوند، بنابراین همیشه باید آماده مدیریت خطاها باشید.
- **بهینه‌سازی**: تنها بافرهایی را فعال کنید که واقعاً به آن‌ها نیاز دارید تا از مصرف بی‌جهت حافظه جلوگیری شود.




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
