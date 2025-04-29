---
title: ترسیم اشکال
nav_order: 4
permalink: /shapes/
layout: persian
---

# ترسیم اشکال در OpenGL

## مقدمه
در این بخش، ما با مفاهیم پایه‌ای رسم اشکال هندسی در OpenGL آشنا خواهیم شد. رسم اشکال پایه‌ای‌ترین عملیات در گرافیک کامپیوتری است و درک نحوه رسم و مدیریت اشکال برای ایجاد برنامه‌های گرافیکی پیچیده ضروری است.

## مفاهیم پایه‌ای رسم اشکال

### 1. دستگاه مختصات OpenGL
- **مختصات نرمال شده**: محدوده -1 تا 1 در هر بعد
- **مبدأ مختصات**: مرکز پنجره (0,0)
- **جهت محورها**: 
  - محور X: مثبت به راست
  - محور Y: مثبت به بالا
  - محور Z: مثبت به بیرون صفحه (در حالت سه‌بعدی)

### 2. انواع اولیه (Primitives)
OpenGL از چند نوع اولیه پایه برای رسم اشکال استفاده می‌کند:
- **GL_POINTS**: نقاط منفرد
- **GL_LINES**: خطوط منفرد
- **GL_LINE_STRIP**: خطوط متصل
- **GL_LINE_LOOP**: خطوط متصل و بسته
- **GL_TRIANGLES**: مثلث‌های منفرد
- **GL_TRIANGLE_STRIP**: مثلث‌های متصل
- **GL_TRIANGLE_FAN**: مثلث‌های متصل به یک نقطه مرکزی
- **GL_QUADS**: چهارضلعی‌های منفرد
- **GL_QUAD_STRIP**: چهارضلعی‌های متصل
- **GL_POLYGON**: چندضلعی‌های محدب

### 3. رنگ‌ها در OpenGL
- **مدل رنگی RGB**: ترکیب قرمز، سبز و آبی
- **محدوده رنگ**: 0.0 تا 1.0 برای هر کانال
- **آلفا بلندینگ**: برای شفافیت (0.0 تا 1.0)

## رسم اشکال پایه

### 1. رسم نقطه
```cpp
void drawPoint() {
    glPointSize(5.0);  // تنظیم اندازه نقطه
    glBegin(GL_POINTS);
        glColor3f(1.0, 0.0, 0.0);  // قرمز
        glVertex2f(0.0, 0.0);      // مرکز پنجره
    glEnd();
}
```

### 2. رسم خط
```cpp
void drawLine() {
    glLineWidth(2.0);  // تنظیم ضخامت خط
    glBegin(GL_LINES);
        glColor3f(0.0, 1.0, 0.0);  // سبز
        glVertex2f(-0.5, -0.5);    // نقطه شروع
        glColor3f(0.0, 0.0, 1.0);  // آبی
        glVertex2f(0.5, 0.5);      // نقطه پایان
    glEnd();
}
```

### 3. رسم مثلث
```cpp
void drawTriangle() {
    glBegin(GL_TRIANGLES);
        glColor3f(1.0, 0.0, 0.0);  // قرمز
        glVertex2f(-0.5, -0.5);    // گوشه پایین-چپ
        glColor3f(0.0, 1.0, 0.0);  // سبز
        glVertex2f(0.5, -0.5);     // گوشه پایین-راست
        glColor3f(0.0, 0.0, 1.0);  // آبی
        glVertex2f(0.0, 0.5);      // گوشه بالا
    glEnd();
}
```

### 4. رسم مربع
```cpp
void drawSquare() {
    glBegin(GL_QUADS);
        glColor3f(1.0, 0.0, 0.0);  // قرمز
        glVertex2f(-0.5, -0.5);    // گوشه پایین-چپ
        glColor3f(0.0, 1.0, 0.0);  // سبز
        glVertex2f(0.5, -0.5);     // گوشه پایین-راست
        glColor3f(0.0, 0.0, 1.0);  // آبی
        glVertex2f(0.5, 0.5);      // گوشه بالا-راست
        glColor3f(1.0, 1.0, 0.0);  // زرد
        glVertex2f(-0.5, 0.5);     // گوشه بالا-چپ
    glEnd();
}
```

### 5. رسم دایره
```cpp
void drawCircle() {
    const int segments = 100;
    const float radius = 0.5;
    
    glBegin(GL_LINE_LOOP);
        glColor3f(1.0, 0.0, 0.0);  // قرمز
        for (int i = 0; i < segments; i++) {
            float theta = 2.0f * M_PI * float(i) / float(segments);
            float x = radius * cosf(theta);
            float y = radius * sinf(theta);
            glVertex2f(x, y);
        }
    glEnd();
}
```

## رسم اشکال پیشرفته

### 1. رسم چندضلعی
```cpp
void drawPolygon(int sides) {
    glBegin(GL_POLYGON);
        glColor3f(1.0, 0.0, 0.0);  // قرمز
        for (int i = 0; i < sides; i++) {
            float theta = 2.0f * M_PI * float(i) / float(sides);
            float x = 0.5f * cosf(theta);
            float y = 0.5f * sinf(theta);
            glVertex2f(x, y);
        }
    glEnd();
}
```

### 2. رسم ستاره
```cpp
void drawStar(int points) {
    const float outerRadius = 0.5;
    const float innerRadius = 0.25;
    
    glBegin(GL_TRIANGLE_FAN);
        glColor3f(1.0, 1.0, 0.0);  // زرد
        glVertex2f(0.0, 0.0);      // مرکز
        
        for (int i = 0; i <= points * 2; i++) {
            float radius = (i % 2 == 0) ? outerRadius : innerRadius;
            float theta = M_PI * float(i) / float(points);
            float x = radius * cosf(theta);
            float y = radius * sinf(theta);
            glVertex2f(x, y);
        }
    glEnd();
}
```

### 3. رسم منحنی‌ها
```cpp
void drawCurve() {
    glBegin(GL_LINE_STRIP);
        glColor3f(0.0, 1.0, 0.0);  // سبز
        for (float t = -1.0; t <= 1.0; t += 0.01) {
            float x = t;
            float y = t * t;  // تابع سهمی
            glVertex2f(x, y);
        }
    glEnd();
}
```

## تبدیلات هندسی

### 1. انتقال (Translation)
```cpp
void translate(float x, float y) {
    glTranslatef(x, y, 0.0);
}
```

### 2. چرخش (Rotation)
```cpp
void rotate(float angle) {
    glRotatef(angle, 0.0, 0.0, 1.0);  // چرخش حول محور Z
}
```

### 3. مقیاس‌بندی (Scaling)
```cpp
void scale(float x, float y) {
    glScalef(x, y, 1.0);
}
```

### 4. ذخیره و بازیابی ماتریس
```cpp
void saveAndRestoreMatrix() {
    glPushMatrix();  // ذخیره ماتریس فعلی
    // انجام تبدیلات
    glPopMatrix();   // بازیابی ماتریس ذخیره شده
}
```

## پاک‌سازی و آماده‌سازی صفحه نمایش

قبل از رندرینگ هر فریم، باید صفحه نمایش را پاک کرده و برای رندرینگ آماده کنید. در OpenGL این کار با توابع `glClearColor` و `glClear` انجام می‌شود.

### تنظیم رنگ پس‌زمینه با glClearColor

تابع `glClearColor` رنگ پس‌زمینه‌ای را تعیین می‌کند که هنگام پاک‌سازی صفحه استفاده می‌شود:

```cpp
void glClearColor(GLfloat red, GLfloat green, GLfloat blue, GLfloat alpha);
```

پارامترها:
- `red`, `green`, `blue`: مؤلفه‌های رنگی در محدوده 0.0 تا 1.0
- `alpha`: شفافیت در محدوده 0.0 (کاملاً شفاف) تا 1.0 (کاملاً مات)

مثال‌های کاربردی:
```cpp
// تنظیم پس‌زمینه سیاه
glClearColor(0.0f, 0.0f, 0.0f, 1.0f);

// تنظیم پس‌زمینه سفید
glClearColor(1.0f, 1.0f, 1.0f, 1.0f);

// تنظیم پس‌زمینه آبی آسمانی
glClearColor(0.53f, 0.81f, 0.92f, 1.0f);

// تنظیم پس‌زمینه خاکستری
glClearColor(0.5f, 0.5f, 0.5f, 1.0f);
```

### پاک‌سازی بافرها با glClear

تابع `glClear` بافرهای مشخص‌شده را با مقادیر پیش‌فرض پاک می‌کند:

```cpp
void glClear(GLbitfield mask);
```

پارامتر `mask` ترکیبی از یک یا چند پرچم است که مشخص می‌کند کدام بافرها باید پاک شوند:

| پرچم | توضیح |
| --- | --- |
| `GL_COLOR_BUFFER_BIT` | بافر رنگ (پاک‌سازی با رنگ تنظیم‌شده در `glClearColor`) |
| `GL_DEPTH_BUFFER_BIT` | بافر عمق (پاک‌سازی با مقدار تنظیم‌شده در `glClearDepth`) |
| `GL_STENCIL_BUFFER_BIT` | بافر استنسیل (پاک‌سازی با مقدار تنظیم‌شده در `glClearStencil`) |
| `GL_ACCUM_BUFFER_BIT` | بافر انباشت (پاک‌سازی با مقدار تنظیم‌شده در `glClearAccum`) |

مثال‌های کاربردی:
```cpp
// پاک‌سازی فقط بافر رنگ
glClear(GL_COLOR_BUFFER_BIT);

// پاک‌سازی بافر رنگ و عمق (معمول در رندرینگ سه‌بعدی)
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

// پاک‌سازی همه بافرهای اصلی
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT | GL_STENCIL_BUFFER_BIT);
```

### تنظیمات تکمیلی

برای کنترل بیشتر بر روی عملیات پاک‌سازی، می‌توانید از این توابع استفاده کنید:

```cpp
// تنظیم مقدار پیش‌فرض بافر عمق (بین 0.0 تا 1.0، پیش‌فرض 1.0)
glClearDepth(1.0);

// تنظیم مقدار پیش‌فرض بافر استنسیل (عدد صحیح، پیش‌فرض 0)
glClearStencil(0);

// تنظیم مقدار پیش‌فرض بافر انباشت
glClearAccum(0.0f, 0.0f, 0.0f, 0.0f);
```

### کارایی و بهینه‌سازی

- **عملیات پرهزینه**: پاک‌سازی بافرها می‌تواند عملیات نسبتاً سنگینی باشد، خصوصاً در وضوح‌های بالا.
- **تکنیک Scissor**: می‌توانید با استفاده از `glScissor` فقط بخشی از صفحه را پاک کنید:
  ```cpp
  glEnable(GL_SCISSOR_TEST);
  glScissor(x, y, width, height);
  glClear(GL_COLOR_BUFFER_BIT);
  glDisable(GL_SCISSOR_TEST);
  ```
- **بهینه‌سازی عملکرد**: در برخی سناریوها، ممکن است بتوانید از پاک‌سازی هر فریم صرف‌نظر کنید (مثلاً اگر کل صفحه با محتوای جدید روی آن رسم می‌شود).

### یک مثال کامل

```cpp
void display() {
    // تنظیم رنگ پس‌زمینه به آبی تیره
    glClearColor(0.0f, 0.1f, 0.2f, 1.0f);
    
    // پاک‌سازی بافر رنگ و عمق
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    
    // رسم اشکال...
    drawScene();
    
    // تعویض بافرهای جلو و عقب
    glutSwapBuffers();
}
```


## کد نمونه کامل

```cpp
#include <GL/glut.h>
#include <cmath>

// متغیرهای سراسری
float rotationAngle = 0.0;
float scaleFactor = 1.0;
float translationX = 0.0;
float translationY = 0.0;

// تابع callback برای رسم
void display() {
    glClear(GL_COLOR_BUFFER_BIT);
    
    // رسم مربع با تبدیلات
    glPushMatrix();
        glTranslatef(translationX, translationY, 0.0);
        glRotatef(rotationAngle, 0.0, 0.0, 1.0);
        glScalef(scaleFactor, scaleFactor, 1.0);
        
        glBegin(GL_QUADS);
            glColor3f(1.0, 0.0, 0.0);  // قرمز
            glVertex2f(-0.5, -0.5);
            glColor3f(0.0, 1.0, 0.0);  // سبز
            glVertex2f(0.5, -0.5);
            glColor3f(0.0, 0.0, 1.0);  // آبی
            glVertex2f(0.5, 0.5);
            glColor3f(1.0, 1.0, 0.0);  // زرد
            glVertex2f(-0.5, 0.5);
        glEnd();
    glPopMatrix();
    
    // رسم دایره
    glPushMatrix();
        glTranslatef(-0.7, 0.0, 0.0);
        glBegin(GL_LINE_LOOP);
            glColor3f(1.0, 0.0, 0.0);  // قرمز
            for (int i = 0; i < 100; i++) {
                float theta = 2.0f * M_PI * float(i) / 100.0f;
                float x = 0.3f * cosf(theta);
                float y = 0.3f * sinf(theta);
                glVertex2f(x, y);
            }
        glEnd();
    glPopMatrix();
    
    // رسم مثلث
    glPushMatrix();
        glTranslatef(0.7, 0.0, 0.0);
        glBegin(GL_TRIANGLES);
            glColor3f(0.0, 1.0, 0.0);  // سبز
            glVertex2f(-0.3, -0.3);
            glColor3f(0.0, 0.0, 1.0);  // آبی
            glVertex2f(0.3, -0.3);
            glColor3f(1.0, 0.0, 0.0);  // قرمز
            glVertex2f(0.0, 0.3);
        glEnd();
    glPopMatrix();
    
    glutSwapBuffers();
}

// تابع callback برای تغییر اندازه پنجره
void reshape(int w, int h) {
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
        case 'r': // چرخش
            rotationAngle += 10.0;
            break;
        case 's': // مقیاس‌بندی
            scaleFactor += 0.1;
            break;
        case 'd': // کاهش مقیاس
            scaleFactor -= 0.1;
            break;
        case 'x': // حرکت به راست
            translationX += 0.1;
            break;
        case 'y': // حرکت به بالا
            translationY += 0.1;
            break;
    }
    glutPostRedisplay();
}

int main(int argc, char** argv) {
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB);
    glutInitWindowSize(800, 600);
    glutInitWindowPosition(100, 100);
    glutCreateWindow("رسم اشکال در OpenGL - درس گرافیک کامپیوتری");
    
    glutDisplayFunc(display);
    glutReshapeFunc(reshape);
    glutKeyboardFunc(keyboard);
    
    glutMainLoop();
    return 0;
}
```

## منابع بیشتر
- [مستندات رسمی OpenGL](https://www.opengl.org/documentation/)
- [OpenGL Primitives](https://www.khronos.org/opengl/wiki/Primitive)
- [OpenGL Transformations](https://www.khronos.org/opengl/wiki/Transformations)
- [مقالات آکادمیک SIGGRAPH](https://www.siggraph.org/)
- [کتاب OpenGL Programming Guide (Red Book)](https://www.opengl.org/archives/resources/code/samples/glut_examples/examples/redbook/)
