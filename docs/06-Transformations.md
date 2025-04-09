---
title: تبدیلات هندسی
nav_order: 6
permalink: /tf/
layout: persian
---


# تبدیلات هندسی در OpenGL

## مقدمه
در این بخش، با مفاهیم پیشرفته تبدیلات هندسی در OpenGL آشنا می‌شویم. تبدیلات هندسی یکی از پایه‌ای‌ترین مفاهیم در گرافیک کامپیوتری هستند که به ما امکان می‌دهند اشیاء را در فضای سه‌بعدی حرکت دهیم، بچرخانیم و تغییر اندازه دهیم. در این درس، علاوه بر مفاهیم پایه‌ای، روش‌های پیشرفته‌تر تبدیلات را نیز بررسی می‌کنیم.

## مفاهیم پایه‌ای تبدیلات

### 1. سیستم‌های مختصات
- **سیستم مختصات جهانی (World Coordinate System)**: سیستم مرجع اصلی برای تمام اشیاء در صحنه
- **سیستم مختصات محلی (Local Coordinate System)**: سیستم مختصات خاص هر شیء
- **سیستم مختصات دوربین (Camera Coordinate System)**: سیستم مختصات مرتبط با دیدگاه دوربین
- **سیستم مختصات صفحه نمایش (Screen Coordinate System)**: سیستم مختصات دو بعدی برای نمایش نهایی

### 2. تبدیلات پایه
- **جابجایی (Translation)**: حرکت یک شیء در راستای یک یا چند محور
- **چرخش (Rotation)**: چرخاندن یک شیء حول یک محور یا نقطه
- **مقیاس‌بندی (Scaling)**: تغییر اندازه یک شیء در راستای یک یا چند محور
- **انعکاس (Reflection)**: آینه‌ای کردن یک شیء نسبت به یک صفحه یا محور

## ماتریس‌های تبدیل

در گرافیک کامپیوتری، تبدیلات به صورت ماتریس‌های 4×4 نمایش داده می‌شوند که با بردارهای همگن (x, y, z, w) کار می‌کنند.

### مفهوم ماتریس انتقال

برای جابجایی یک شیء به اندازه (tx, ty, tz):

```
| 1  0  0  tx |
| 0  1  0  ty |
| 0  0  1  tz |
| 0  0  0  1  |
```

### مفهوم ماتریس مقیاس‌بندی

برای تغییر مقیاس با ضرایب (sx, sy, sz):

```
| sx  0   0   0 |
| 0   sy  0   0 |
| 0   0   sz  0 |
| 0   0   0   1 |
```

### مفهوم ماتریس چرخش

برای چرخش حول محور x به اندازه θ:

```
| 1  0       0        0 |
| 0  cos(θ)  -sin(θ)  0 |
| 0  sin(θ)  cos(θ)   0 |
| 0  0       0        1 |
```

برای چرخش حول محور y به اندازه θ:

```
| cos(θ)   0  sin(θ)  0 |
| 0        1  0       0 |
| -sin(θ)  0  cos(θ)  0 |
| 0        0  0       1 |
```

برای چرخش حول محور z به اندازه θ:

```
| cos(θ)  -sin(θ)  0  0 |
| sin(θ)  cos(θ)   0  0 |
| 0       0        1  0 |
| 0       0        0  1 |
```

## کد ماتریس‌های تبدیل

### 1. ماتریس مدل (Model Matrix)
ماتریس مدل، تبدیلات مربوط به موقعیت، چرخش و اندازه یک شیء را در فضای جهانی تعریف می‌کند.

```cpp
void setupModelMatrix() {
    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();  // بارگذاری ماتریس یکه
    
    // اعمال تبدیلات
    glTranslatef(x, y, z);  // جابجایی
    glRotatef(angle, 0.0f, 1.0f, 0.0f);  // چرخش
    glScalef(scale_x, scale_y, scale_z);  // مقیاس‌بندی
}
```

#### ترتیب اعمال تبدیلات
ترتیب اعمال تبدیلات بسیار مهم است. در OpenGL، تبدیلات از راست به چپ اعمال می‌شوند:

```cpp
// ترتیب صحیح: مقیاس‌بندی -> چرخش -> جابجایی
glLoadIdentity();
glTranslatef(x, y, z);
glRotatef(angle, 0.0f, 1.0f, 0.0f);
glScalef(scale_x, scale_y, scale_z);
```

### 2. ماتریس نمایش (View Matrix)
ماتریس نمایش، موقعیت و جهت دوربین را در صحنه تعریف می‌کند.

```cpp
void setupViewMatrix() {
    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();
    
    // تنظیم دوربین
    gluLookAt(
        eye_x, eye_y, eye_z,  // موقعیت چشم
        center_x, center_y, center_z,  // نقطه نگاه
        up_x, up_y, up_z  // بردار بالا
    );
}
```

#### دوربین اول شخص
```cpp
void setupFirstPersonCamera() {
    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();
    
    // محاسبه بردارهای دوربین
    glm::vec3 direction;
    direction.x = cos(glm::radians(yaw)) * cos(glm::radians(pitch));
    direction.y = sin(glm::radians(pitch));
    direction.z = sin(glm::radians(yaw)) * cos(glm::radians(pitch));
    glm::vec3 cameraFront = glm::normalize(direction);
    
    // تنظیم دوربین
    gluLookAt(
        cameraPos.x, cameraPos.y, cameraPos.z,
        cameraPos.x + cameraFront.x, cameraPos.y + cameraFront.y, cameraPos.z + cameraFront.z,
        cameraUp.x, cameraUp.y, cameraUp.z
    );
}
```

### 3. ماتریس برجستگی (Projection Matrix)
ماتریس برجستگی، نحوه نمایش اشیاء سه‌بعدی روی صفحه نمایش دو بعدی را تعریف می‌کند.

```cpp
void setupProjectionMatrix() {
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    
    // برجستگی پرسپکتیو
    gluPerspective(45.0f, aspect_ratio, 0.1f, 100.0f);
    
    // یا برجستگی ارتوگرافیک
    // glOrtho(-1.0f, 1.0f, -1.0f, 1.0f, -1.0f, 1.0f);
}
```

#### مقایسه برجستگی‌های مختلف
- **برجستگی پرسپکتیو (Perspective Projection)**: برای ایجاد عمق و فاصله در صحنه‌های واقع‌گرایانه
- **برجستگی ارتوگرافیک (Orthographic Projection)**: برای نمایش دقیق اندازه‌ها بدون اعوجاج پرسپکتیو
- **برجستگی اوبلیک (Oblique Projection)**: برای نمایش سه‌بعدی در نقشه‌های فنی

## تبدیلات پیشرفته

### 1. تبدیلات ترکیبی
ترکیب چندین تبدیل برای ایجاد حرکت‌های پیچیده‌تر:

```cpp
void applyComplexTransformation() {
    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();
    
    // ترکیب چندین تبدیل
    glTranslatef(0.0f, 0.0f, -5.0f);  // جابجایی به عقب
    glRotatef(angle, 0.0f, 1.0f, 0.0f);  // چرخش حول محور Y
    glTranslatef(2.0f, 0.0f, 0.0f);  // جابجایی به راست
    glRotatef(angle * 2, 0.0f, 0.0f, 1.0f);  // چرخش حول محور Z
}
```

### 2. تبدیلات نسبی
تبدیلات نسبت به موقعیت فعلی یک شیء:

```cpp
void applyRelativeTransformation() {
    // ذخیره ماتریس فعلی
    glPushMatrix();
    
    // اعمال تبدیلات نسبی
    glTranslatef(1.0f, 0.0f, 0.0f);  // جابجایی نسبی
    glRotatef(angle, 0.0f, 1.0f, 0.0f);  // چرخش نسبی
    
    // رسم شیء
    drawObject();
    
    // بازیابی ماتریس قبلی
    glPopMatrix();
}
```

### 3. تبدیلات سلسله مراتبی
تبدیلات برای ساختارهای سلسله مراتبی مانند روبات یا کاراکتر:

```cpp
void drawRobotArm() {
    // پایه روبات
    glPushMatrix();
    glTranslatef(0.0f, 0.0f, 0.0f);
    glRotatef(baseAngle, 0.0f, 1.0f, 0.0f);
    drawBase();
    
    // بازوی اول
    glPushMatrix();
    glTranslatef(0.0f, 1.0f, 0.0f);
    glRotatef(arm1Angle, 0.0f, 0.0f, 1.0f);
    drawArm1();
    
    // بازوی دوم
    glPushMatrix();
    glTranslatef(1.0f, 0.0f, 0.0f);
    glRotatef(arm2Angle, 0.0f, 0.0f, 1.0f);
    drawArm2();
    
    // دست روبات
    glPushMatrix();
    glTranslatef(1.0f, 0.0f, 0.0f);
    glRotatef(handAngle, 0.0f, 1.0f, 0.0f);
    drawHand();
    glPopMatrix();
    
    glPopMatrix();
    glPopMatrix();
    glPopMatrix();
}
```

### 4. تبدیلات با استفاده از کواترنیون‌ها
استفاده از کواترنیون‌ها برای چرخش‌های روان و بدون قفل گیمبال:

```cpp
void applyQuaternionRotation() {
    // تعریف کواترنیون
    glm::quat rotation = glm::quat(glm::radians(angle), 0.0f, 1.0f, 0.0f);
    
    // تبدیل کواترنیون به ماتریس
    glm::mat4 rotationMatrix = glm::mat4_cast(rotation);
    
    // اعمال ماتریس چرخش
    glMultMatrixf(glm::value_ptr(rotationMatrix));
}
```

## تبدیلات در شیدرها

### 1. تبدیلات در شیدرهای مدرن
در OpenGL مدرن، تبدیلات در شیدرها انجام می‌شوند:

```glsl
// شیدر ورتیکس
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aNormal;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

out vec3 FragPos;
out vec3 Normal;

void main() {
    FragPos = vec3(model * vec4(aPos, 1.0));
    Normal = mat3(transpose(inverse(model))) * aNormal;
    gl_Position = projection * view * model * vec4(aPos, 1.0);
}
```

### 2. تبدیلات نرمال
تبدیل صحیح نرمال‌ها برای محاسبات نور:

```glsl
// محاسبه ماتریس نرمال
Normal = mat3(transpose(inverse(model))) * aNormal;
```

## مثال کامل

```cpp
#include <GL/glut.h>
#include <glm/glm.hpp>
#include <glm/gtc/matrix_transform.hpp>
#include <glm/gtc/type_ptr.hpp>
#include <vector>
#include <cmath>

// ساختار برای نگهداری اطلاعات یک شیء
struct Object {
    glm::vec3 position;
    glm::vec3 rotation;
    glm::vec3 scale;
    glm::vec3 color;
};

// متغیرهای سراسری
std::vector<Object> objects;
float cameraDistance = 5.0f;
float cameraRotation = 0.0f;
float cameraPitch = 0.0f;
float cameraYaw = -90.0f;
glm::vec3 cameraPos = glm::vec3(0.0f, 0.0f, 5.0f);
glm::vec3 cameraFront = glm::vec3(0.0f, 0.0f, -1.0f);
glm::vec3 cameraUp = glm::vec3(0.0f, 1.0f, 0.0f);

// تابع رسم مکعب
void drawCube(const Object& obj) {
    // اعمال تبدیلات
    glPushMatrix();
    glTranslatef(obj.position.x, obj.position.y, obj.position.z);
    glRotatef(obj.rotation.x, 1.0f, 0.0f, 0.0f);
    glRotatef(obj.rotation.y, 0.0f, 1.0f, 0.0f);
    glRotatef(obj.rotation.z, 0.0f, 0.0f, 1.0f);
    glScalef(obj.scale.x, obj.scale.y, obj.scale.z);
    
    // تنظیم رنگ
    glColor3f(obj.color.x, obj.color.y, obj.color.z);
    
    // رسم مکعب
    glBegin(GL_QUADS);
        // وجه جلو
        glVertex3f(-1.0f, -1.0f, 1.0f);
        glVertex3f(1.0f, -1.0f, 1.0f);
        glVertex3f(1.0f, 1.0f, 1.0f);
        glVertex3f(-1.0f, 1.0f, 1.0f);
        
        // وجه پشت
        glVertex3f(-1.0f, -1.0f, -1.0f);
        glVertex3f(-1.0f, 1.0f, -1.0f);
        glVertex3f(1.0f, 1.0f, -1.0f);
        glVertex3f(1.0f, -1.0f, -1.0f);
        
        // وجه بالا
        glVertex3f(-1.0f, 1.0f, -1.0f);
        glVertex3f(-1.0f, 1.0f, 1.0f);
        glVertex3f(1.0f, 1.0f, 1.0f);
        glVertex3f(1.0f, 1.0f, -1.0f);
        
        // وجه پایین
        glVertex3f(-1.0f, -1.0f, -1.0f);
        glVertex3f(1.0f, -1.0f, -1.0f);
        glVertex3f(1.0f, -1.0f, 1.0f);
        glVertex3f(-1.0f, -1.0f, 1.0f);
        
        // وجه راست
        glVertex3f(1.0f, -1.0f, -1.0f);
        glVertex3f(1.0f, 1.0f, -1.0f);
        glVertex3f(1.0f, 1.0f, 1.0f);
        glVertex3f(1.0f, -1.0f, 1.0f);
        
        // وجه چپ
        glVertex3f(-1.0f, -1.0f, -1.0f);
        glVertex3f(-1.0f, -1.0f, 1.0f);
        glVertex3f(-1.0f, 1.0f, 1.0f);
        glVertex3f(-1.0f, 1.0f, -1.0f);
    glEnd();
    
    glPopMatrix();
}

// تابع رسم هرم
void drawPyramid(const Object& obj) {
    // اعمال تبدیلات
    glPushMatrix();
    glTranslatef(obj.position.x, obj.position.y, obj.position.z);
    glRotatef(obj.rotation.x, 1.0f, 0.0f, 0.0f);
    glRotatef(obj.rotation.y, 0.0f, 1.0f, 0.0f);
    glRotatef(obj.rotation.z, 0.0f, 0.0f, 1.0f);
    glScalef(obj.scale.x, obj.scale.y, obj.scale.z);
    
    // تنظیم رنگ
    glColor3f(obj.color.x, obj.color.y, obj.color.z);
    
    // رسم هرم
    glBegin(GL_TRIANGLES);
        // وجه جلو
        glVertex3f(0.0f, 1.0f, 0.0f);
        glVertex3f(-1.0f, -1.0f, 1.0f);
        glVertex3f(1.0f, -1.0f, 1.0f);
        
        // وجه راست
        glVertex3f(0.0f, 1.0f, 0.0f);
        glVertex3f(1.0f, -1.0f, 1.0f);
        glVertex3f(1.0f, -1.0f, -1.0f);
        
        // وجه پشت
        glVertex3f(0.0f, 1.0f, 0.0f);
        glVertex3f(1.0f, -1.0f, -1.0f);
        glVertex3f(-1.0f, -1.0f, -1.0f);
        
        // وجه چپ
        glVertex3f(0.0f, 1.0f, 0.0f);
        glVertex3f(-1.0f, -1.0f, -1.0f);
        glVertex3f(-1.0f, -1.0f, 1.0f);
    glEnd();
    
    // قاعده هرم
    glBegin(GL_QUADS);
        glVertex3f(-1.0f, -1.0f, 1.0f);
        glVertex3f(1.0f, -1.0f, 1.0f);
        glVertex3f(1.0f, -1.0f, -1.0f);
        glVertex3f(-1.0f, -1.0f, -1.0f);
    glEnd();
    
    glPopMatrix();
}

// تابع رسم صحنه
void display() {
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    
    // تنظیم ماتریس مدل-ویو
    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();
    
    // تنظیم دوربین
    gluLookAt(
        cameraPos.x, cameraPos.y, cameraPos.z,
        cameraPos.x + cameraFront.x, cameraPos.y + cameraFront.y, cameraPos.z + cameraFront.z,
        cameraUp.x, cameraUp.y, cameraUp.z
    );
    
    // رسم همه اشیاء
    for (const auto& obj : objects) {
        if (obj.scale.x == obj.scale.y && obj.scale.y == obj.scale.z) {
            // اگر مقیاس در همه جهت‌ها یکسان است، احتمالاً مکعب است
            drawCube(obj);
        } else {
            // در غیر این صورت، هرم است
            drawPyramid(obj);
        }
    }
    
    glutSwapBuffers();
}

// تابع به‌روزرسانی دوربین
void updateCamera() {
    // محاسبه بردارهای دوربین
    glm::vec3 direction;
    direction.x = cos(glm::radians(cameraYaw)) * cos(glm::radians(cameraPitch));
    direction.y = sin(glm::radians(cameraPitch));
    direction.z = sin(glm::radians(cameraYaw)) * cos(glm::radians(cameraPitch));
    cameraFront = glm::normalize(direction);
}

// تابع تغییر اندازه پنجره
void reshape(int w, int h) {
    glViewport(0, 0, w, h);
    
    // تنظیم ماتریس برجستگی
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    gluPerspective(45.0f, (float)w/h, 0.1f, 100.0f);
    
    glMatrixMode(GL_MODELVIEW);
}

// تابع مدیریت کیبورد
void keyboard(unsigned char key, int x, int y) {
    float cameraSpeed = 0.1f;
    
    switch(key) {
        case 27:  // ESC
            exit(0);
            break;
        case 'w':
            cameraPos += cameraSpeed * cameraFront;
            break;
        case 's':
            cameraPos -= cameraSpeed * cameraFront;
            break;
        case 'a':
            cameraPos -= glm::normalize(glm::cross(cameraFront, cameraUp)) * cameraSpeed;
            break;
        case 'd':
            cameraPos += glm::normalize(glm::cross(cameraFront, cameraUp)) * cameraSpeed;
            break;
        case '+':
            // افزایش مقیاس همه اشیاء
            for (auto& obj : objects) {
                obj.scale += 0.1f;
            }
            break;
        case '-':
            // کاهش مقیاس همه اشیاء
            for (auto& obj : objects) {
                obj.scale -= 0.1f;
            }
            break;
    }
    
    updateCamera();
    glutPostRedisplay();
}

// تابع مدیریت کلیدهای خاص
void specialKeys(int key, int x, int y) {
    float rotationSpeed = 5.0f;
    
    switch(key) {
        case GLUT_KEY_UP:
            cameraPitch += rotationSpeed;
            break;
        case GLUT_KEY_DOWN:
            cameraPitch -= rotationSpeed;
            break;
        case GLUT_KEY_LEFT:
            cameraYaw -= rotationSpeed;
            break;
        case GLUT_KEY_RIGHT:
            cameraYaw += rotationSpeed;
            break;
    }
    
    // محدود کردن زاویه pitch
    if (cameraPitch > 89.0f) cameraPitch = 89.0f;
    if (cameraPitch < -89.0f) cameraPitch = -89.0f;
    
    updateCamera();
    glutPostRedisplay();
}

// تابع به‌روزرسانی
void update(int value) {
    // چرخش اشیاء
    for (auto& obj : objects) {
        obj.rotation.y += 1.0f;
        if (obj.rotation.y > 360.0f) {
            obj.rotation.y -= 360.0f;
        }
    }
    
    glutPostRedisplay();
    glutTimerFunc(16, update, 0);  // حدود 60 FPS
}

// تابع مقداردهی اولیه
void init() {
    // ایجاد اشیاء
    Object cube;
    cube.position = glm::vec3(-2.0f, 0.0f, 0.0f);
    cube.rotation = glm::vec3(0.0f, 0.0f, 0.0f);
    cube.scale = glm::vec3(1.0f, 1.0f, 1.0f);
    cube.color = glm::vec3(1.0f, 0.0f, 0.0f);  // قرمز
    objects.push_back(cube);
    
    Object pyramid;
    pyramid.position = glm::vec3(2.0f, 0.0f, 0.0f);
    pyramid.rotation = glm::vec3(0.0f, 0.0f, 0.0f);
    pyramid.scale = glm::vec3(1.0f, 1.5f, 1.0f);  // مقیاس متفاوت در جهت Y
    pyramid.color = glm::vec3(0.0f, 0.0f, 1.0f);  // آبی
    objects.push_back(pyramid);
}

int main(int argc, char** argv) {
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB | GLUT_DEPTH);
    glutInitWindowSize(800, 600);
    glutCreateWindow("تبدیلات هندسی - درس گرافیک کامپیوتری");
    
    glEnable(GL_DEPTH_TEST);
    glClearColor(0.0f, 0.0f, 0.0f, 1.0f);
    
    init();
    
    glutDisplayFunc(display);
    glutReshapeFunc(reshape);
    glutKeyboardFunc(keyboard);
    glutSpecialFunc(specialKeys);
    glutTimerFunc(0, update, 0);
    
    glutMainLoop();
    return 0;
}
```

## نکات مهم
1. ترتیب اعمال تبدیلات مهم است (جابجایی، چرخش، مقیاس‌بندی)
2. همیشه قبل از رسم، ماتریس مدل را ریست کنید
3. برای عمق‌نمایی، `GL_DEPTH_TEST` را فعال کنید
4. از `glutTimerFunc` برای انیمیشن استفاده کنید
5. برای تبدیلات پیچیده، از `glPushMatrix` و `glPopMatrix` استفاده کنید
6. برای چرخش‌های روان، از کواترنیون‌ها استفاده کنید
7. در OpenGL مدرن، تبدیلات در شیدرها انجام می‌شوند
8. برای تبدیلات سلسله مراتبی، از ساختار درختی استفاده کنید

## منابع بیشتر
- [مستندات OpenGL](https://www.opengl.org/documentation/)
- [راهنمای تبدیلات](https://www.opengl.org/wiki/Transformations)
- [کتاب Mathematics for 3D Game Programming and Computer Graphics](https://www.amazon.com/Mathematics-Programming-Computer-Graphics-Development/dp/1435458869)
- [مقالات GDC درباره تبدیلات و انیمیشن](https://www.gdc.com/)
- [مستندات GLM](https://github.com/g-truc/glm)
- [کتاب Real-Time Rendering](https://www.realtimerendering.com/) 
