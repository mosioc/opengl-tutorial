---
title: دوربین
nav_order: 11
permalink: /camera/
layout: persian
---

# دوربین در OpenGL - درس گرافیک کامپیوتری

## مقدمه
دوربین در OpenGL به ما امکان می‌دهد تا صحنه سه بعدی را از زوایای مختلف مشاهده کنیم. برای این کار، ما نیاز به تعریف و مدیریت ماتریس‌های view و projection داریم. در این درس، علاوه بر مفاهیم پایه‌ای دوربین، تکنیک‌های پیشرفته‌تر مانند دوربین‌های سوم شخص، دوربین‌های تعاملی، و انواع مختلف projection را نیز بررسی می‌کنیم.

## مفاهیم پایه‌ای دوربین

### 1. سیستم مختصات دوربین
- **دستگاه مختصات جهانی (World Space)**: دستگاه مختصات اصلی که تمام اشیاء در آن قرار دارند
- **دستگاه مختصات دوربین (Camera Space)**: دستگاه مختصاتی که با دوربین هم‌مرکز است
- **دستگاه مختصات صفحه نمایش (Screen Space)**: دستگاه مختصات دو بعدی که برای نمایش نهایی استفاده می‌شود

### 2. پارامترهای اصلی دوربین
- **موقعیت دوربین (Eye Position)**: نقطه‌ای که دوربین از آنجا به صحنه نگاه می‌کند
- **نقطه مورد نظر (Target Position)**: نقطه‌ای که دوربین به آن نگاه می‌کند
- **بردار Up**: جهت بالا برای دوربین (معمولاً بردار Y مثبت)
- **زاویه دید (Field of View)**: زاویه‌ای که دوربین می‌تواند ببیند
- **نسبت تصویر (Aspect Ratio)**: نسبت عرض به ارتفاع پنجره نمایش
- **صفحه نزدیک و دور (Near/Far Planes)**: محدوده عمق قابل مشاهده

## ماتریس View
ماتریس view تعیین می‌کند که دوربین کجا قرار دارد و به کجا نگاه می‌کند. این ماتریس شامل:
- موقعیت دوربین (eye position)
- نقطه مورد نظر (target position)
- بردار up

```cpp
#include <GL/glut.h>
#include <cmath>

// ساختار برای نگهداری اطلاعات دوربین
struct Camera {
    float x, y, z;        // موقعیت دوربین
    float targetX, targetY, targetZ;  // نقطه مورد نظر
    float upX, upY, upZ;  // بردار up
    
    Camera() {
        x = 0.0f; y = 0.0f; z = 5.0f;
        targetX = 0.0f; targetY = 0.0f; targetZ = 0.0f;
        upX = 0.0f; upY = 1.0f; upZ = 0.0f;
    }
    
    void lookAt() {
        gluLookAt(x, y, z,           // موقعیت دوربین
                  targetX, targetY, targetZ,  // نقطه مورد نظر
                  upX, upY, upZ);     // بردار up
    }
};

Camera camera;

void display() {
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    glLoadIdentity();
    
    // اعمال ماتریس view
    camera.lookAt();
    
    // رسم یک مکعب ساده
    glutSolidCube(1.0);
    
    glutSwapBuffers();
}

// حرکت دوربین با کلیدهای جهت‌دار
void specialKeys(int key, int x, int y) {
    float speed = 0.5f;
    switch(key) {
        case GLUT_KEY_LEFT:
            camera.x -= speed;
            camera.targetX -= speed;
            break;
        case GLUT_KEY_RIGHT:
            camera.x += speed;
            camera.targetX += speed;
            break;
        case GLUT_KEY_UP:
            camera.z -= speed;
            camera.targetZ -= speed;
            break;
        case GLUT_KEY_DOWN:
            camera.z += speed;
            camera.targetZ += speed;
            break;
    }
    glutPostRedisplay();
}

int main(int argc, char** argv) {
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB | GLUT_DEPTH);
    glutInitWindowSize(800, 600);
    glutCreateWindow("دوربین OpenGL");
    
    glEnable(GL_DEPTH_TEST);
    
    glutDisplayFunc(display);
    glutSpecialFunc(specialKeys);
    
    // تنظیم ماتریس projection
    glMatrixMode(GL_PROJECTION);
    gluPerspective(45.0f, 800.0f/600.0f, 0.1f, 100.0f);
    glMatrixMode(GL_MODELVIEW);
    
    glutMainLoop();
    return 0;
}
```

### 3. محاسبه ماتریس View به صورت دستی
در برخی موارد، ممکن است بخواهیم ماتریس view را به صورت دستی محاسبه کنیم:

```cpp
void calculateViewMatrix(float* viewMatrix, 
                        float eyeX, float eyeY, float eyeZ,
                        float targetX, float targetY, float targetZ,
                        float upX, float upY, float upZ) {
    // محاسبه بردار جهت (forward)
    float forwardX = targetX - eyeX;
    float forwardY = targetY - eyeY;
    float forwardZ = targetZ - eyeZ;
    
    // نرمال‌سازی بردار جهت
    float length = sqrt(forwardX*forwardX + forwardY*forwardY + forwardZ*forwardZ);
    forwardX /= length;
    forwardY /= length;
    forwardZ /= length;
    
    // محاسبه بردار راست (right) با استفاده از ضرب خارجی
    float rightX = forwardY * upZ - forwardZ * upY;
    float rightY = forwardZ * upX - forwardX * upZ;
    float rightZ = forwardX * upY - forwardY * upX;
    
    // نرمال‌سازی بردار راست
    length = sqrt(rightX*rightX + rightY*rightY + rightZ*rightZ);
    rightX /= length;
    rightY /= length;
    rightZ /= length;
    
    // محاسبه بردار بالا (up) با استفاده از ضرب خارجی
    float upVectorX = rightY * forwardZ - rightZ * forwardY;
    float upVectorY = rightZ * forwardX - rightX * forwardZ;
    float upVectorZ = rightX * forwardY - rightY * forwardX;
    
    // نرمال‌سازی بردار بالا
    length = sqrt(upVectorX*upVectorX + upVectorY*upVectorY + upVectorZ*upVectorZ);
    upVectorX /= length;
    upVectorY /= length;
    upVectorZ /= length;
    
    // ساخت ماتریس view
    viewMatrix[0] = rightX;    viewMatrix[1] = upVectorX;   viewMatrix[2] = -forwardX;  viewMatrix[3] = 0.0f;
    viewMatrix[4] = rightY;    viewMatrix[5] = upVectorY;   viewMatrix[6] = -forwardY;  viewMatrix[7] = 0.0f;
    viewMatrix[8] = rightZ;    viewMatrix[9] = upVectorZ;   viewMatrix[10] = -forwardZ; viewMatrix[11] = 0.0f;
    viewMatrix[12] = -(rightX*eyeX + rightY*eyeY + rightZ*eyeZ);
    viewMatrix[13] = -(upVectorX*eyeX + upVectorY*eyeY + upVectorZ*eyeZ);
    viewMatrix[14] = forwardX*eyeX + forwardY*eyeY + forwardZ*eyeZ;
    viewMatrix[15] = 1.0f;
}
```

## ماتریس Projection
ماتریس projection تعیین می‌کند که چگونه اشیاء سه بعدی به صفحه دو بعدی نمایش منتقل می‌شوند. دو نوع projection اصلی وجود دارد:

### 1. Perspective Projection (پرسپکتیو)
برای ایجاد عمق و پرسپکتیو واقع‌گرایانه استفاده می‌شود. در این نوع projection، اشیاء دورتر کوچکتر به نظر می‌رسند.

```cpp
gluPerspective(45.0f,    // زاویه دید (FOV)
               aspect,    // نسبت عرض به ارتفاع
               0.1f,     // near plane
               100.0f);  // far plane
```

### 2. Orthographic Projection (ارتوگرافیک)
برای نمایش بدون پرسپکتیو استفاده می‌شود. در این نوع projection، اشیاء با هر فاصله‌ای به یک اندازه نمایش داده می‌شوند.

```cpp
glOrtho(-1.0, 1.0,      // left, right
        -1.0, 1.0,      // bottom, top
        -1.0, 1.0);     // near, far
```

### 3. تفاوت‌های اصلی بین Perspective و Orthographic
| ویژگی | Perspective | Orthographic |
|--------|-------------|--------------|
| پرسپکتیو | دارد | ندارد |
| کاربرد | صحنه‌های واقع‌گرایانه | نقشه‌ها، رابط‌های کاربری، CAD |
| پارامترها | FOV, aspect, near, far | left, right, bottom, top, near, far |
| محاسبات | پیچیده‌تر | ساده‌تر |

### 4. محاسبه ماتریس Orthographic به صورت دستی
```cpp
void calculateOrthoMatrix(float* orthoMatrix,
                         float left, float right,
                         float bottom, float top,
                         float near, float far) {
    // پاک کردن ماتریس
    for(int i = 0; i < 16; i++) {
        orthoMatrix[i] = 0.0f;
    }
    
    // مقادیر اصلی
    orthoMatrix[0] = 2.0f / (right - left);
    orthoMatrix[5] = 2.0f / (top - bottom);
    orthoMatrix[10] = -2.0f / (far - near);
    orthoMatrix[15] = 1.0f;
    
    // مقادیر انتقال
    orthoMatrix[12] = -(right + left) / (right - left);
    orthoMatrix[13] = -(top + bottom) / (top - bottom);
    orthoMatrix[14] = -(far + near) / (far - near);
}
```

### 5. ViewBox در SVG و ارتباط آن با Orthographic Projection
ViewBox در SVG مفهومی مشابه Orthographic Projection در OpenGL دارد. ViewBox تعریف می‌کند که کدام بخش از محتوای SVG باید در پنجره نمایش داده شود.

```svg
<svg width="800" height="600" viewBox="0 0 100 100">
  <!-- محتوا -->
</svg>
```

در این مثال، ViewBox از (0,0) تا (100,100) تعریف شده است، که معادل Orthographic Projection با پارامترهای زیر است:
```cpp
glOrtho(0.0, 100.0,  // left, right
        0.0, 100.0,  // bottom, top
        -1.0, 1.0);  // near, far
```

ViewBox به ما امکان می‌دهد تا:
1. مقیاس‌بندی خودکار محتوا را کنترل کنیم
2. بخش خاصی از محتوا را نمایش دهیم
3. زوم و پن را بدون تغییر در محتوای اصلی پیاده‌سازی کنیم

## تکنیک‌های پیشرفته دوربین

### 1. دوربین سوم شخص (Third-Person Camera)
دوربینی که یک شیء را از فاصله دنبال می‌کند:

```cpp
struct ThirdPersonCamera {
    float distance;      // فاصله از شیء
    float height;        // ارتفاع دوربین
    float angle;         // زاویه چرخش دوربین
    
    void update(float targetX, float targetY, float targetZ) {
        // محاسبه موقعیت دوربین بر اساس موقعیت هدف
        camera.x = targetX + distance * sin(angle);
        camera.y = targetY + height;
        camera.z = targetZ + distance * cos(angle);
        
        // تنظیم نقطه مورد نظر به موقعیت هدف
        camera.targetX = targetX;
        camera.targetY = targetY;
        camera.targetZ = targetZ;
    }
    
    void rotate(float deltaAngle) {
        angle += deltaAngle;
    }
    
    void adjustDistance(float deltaDistance) {
        distance += deltaDistance;
        if(distance < 1.0f) distance = 1.0f;
    }
    
    void adjustHeight(float deltaHeight) {
        height += deltaHeight;
    }
};

ThirdPersonCamera thirdPersonCam;
float targetX = 0.0f, targetY = 0.0f, targetZ = 0.0f;

void updateCamera() {
    thirdPersonCam.update(targetX, targetY, targetZ);
    camera.lookAt();
}
```

### 2. دوربین اول شخص (First-Person Camera)
دوربینی که از دید شخصیت یا بازیکن به صحنه نگاه می‌کند:

```cpp
struct FirstPersonCamera {
    float yaw;           // چرخش افقی
    float pitch;         // چرخش عمودی
    float positionX, positionY, positionZ;
    
    FirstPersonCamera() {
        yaw = 0.0f;
        pitch = 0.0f;
        positionX = 0.0f;
        positionY = 1.7f;  // ارتفاع چشم
        positionZ = 0.0f;
    }
    
    void update() {
        // محاسبه بردار جهت بر اساس yaw و pitch
        float directionX = cos(pitch) * sin(yaw);
        float directionY = sin(pitch);
        float directionZ = cos(pitch) * cos(yaw);
        
        // تنظیم موقعیت دوربین
        camera.x = positionX;
        camera.y = positionY;
        camera.z = positionZ;
        
        // تنظیم نقطه مورد نظر
        camera.targetX = positionX + directionX;
        camera.targetY = positionY + directionY;
        camera.targetZ = positionZ + directionZ;
    }
    
    void rotate(float deltaYaw, float deltaPitch) {
        yaw += deltaYaw;
        pitch += deltaPitch;
        
        // محدود کردن pitch برای جلوگیری از چرخش کامل
        if(pitch > M_PI/2.0f - 0.1f) pitch = M_PI/2.0f - 0.1f;
        if(pitch < -M_PI/2.0f + 0.1f) pitch = -M_PI/2.0f + 0.1f;
    }
    
    void move(float deltaX, float deltaY, float deltaZ) {
        positionX += deltaX;
        positionY += deltaY;
        positionZ += deltaZ;
    }
    
    void moveForward(float distance) {
        float directionX = cos(pitch) * sin(yaw);
        float directionZ = cos(pitch) * cos(yaw);
        
        positionX += directionX * distance;
        positionZ += directionZ * distance;
    }
    
    void moveRight(float distance) {
        float directionX = sin(yaw);
        float directionZ = -cos(yaw);
        
        positionX += directionX * distance;
        positionZ += directionZ * distance;
    }
};

FirstPersonCamera firstPersonCam;

void updateCamera() {
    firstPersonCam.update();
    camera.lookAt();
}
```

### 3. دوربین تعاملی با ماوس (Mouse-Controlled Camera)
```cpp
int lastX = 400, lastY = 300;  // مرکز پنجره
bool firstMouse = true;
float mouseSensitivity = 0.1f;

void mouseCallback(int x, int y) {
    if(firstMouse) {
        lastX = x;
        lastY = y;
        firstMouse = false;
        return;
    }
    
    float xOffset = (x - lastX) * mouseSensitivity;
    float yOffset = (lastY - y) * mouseSensitivity;
    
    lastX = x;
    lastY = y;
    
    // چرخش دوربین
    firstPersonCam.rotate(xOffset * 0.01f, yOffset * 0.01f);
}

void mouseWheelCallback(int button, int dir, int x, int y) {
    if(dir > 0) {
        // زوم کردن
        thirdPersonCam.adjustDistance(-0.5f);
    } else {
        // زوم بیرون
        thirdPersonCam.adjustDistance(0.5f);
    }
}

int main(int argc, char** argv) {
    // ... کد قبلی ...
    
    glutMotionFunc(mouseCallback);
    glutMouseFunc(mouseWheelCallback);
    
    // ... ادامه کد ...
}
```

### 4. دوربین با قابلیت زوم (Zoomable Camera)
```cpp
struct ZoomableCamera {
    float fov;
    float minFov, maxFov;
    
    ZoomableCamera() {
        fov = 45.0f;
        minFov = 1.0f;
        maxFov = 90.0f;
    }
    
    void zoom(float delta) {
        fov += delta;
        if(fov < minFov) fov = minFov;
        if(fov > maxFov) fov = maxFov;
        
        // به‌روزرسانی ماتریس projection
        glMatrixMode(GL_PROJECTION);
        glLoadIdentity();
        gluPerspective(fov, aspect, near, far);
        glMatrixMode(GL_MODELVIEW);
    }
};

ZoomableCamera zoomCam;

void mouseWheelCallback(int button, int dir, int x, int y) {
    if(dir > 0) {
        // زوم کردن
        zoomCam.zoom(-1.0f);
    } else {
        // زوم بیرون
        zoomCam.zoom(1.0f);
    }
}
```

### 5. دوربین با قابلیت پن (Pan Camera)
```cpp
void panCamera(float deltaX, float deltaY) {
    // محاسبه بردار راست دوربین
    float rightX = camera.targetX - camera.x;
    float rightY = 0.0f;
    float rightZ = camera.targetZ - camera.z;
    
    // نرمال‌سازی
    float length = sqrt(rightX*rightX + rightZ*rightZ);
    rightX /= length;
    rightZ /= length;
    
    // محاسبه بردار بالا
    float upX = rightZ;
    float upY = 0.0f;
    float upZ = -rightX;
    
    // حرکت دوربین و نقطه مورد نظر
    camera.x += rightX * deltaX + upX * deltaY;
    camera.y += deltaY;
    camera.z += rightZ * deltaX + upZ * deltaY;
    
    camera.targetX += rightX * deltaX + upX * deltaY;
    camera.targetY += deltaY;
    camera.targetZ += rightZ * deltaX + upZ * deltaY;
}
```

## مثال کامل: دوربین پیشرفته

```cpp
#include <GL/glut.h>
#include <cmath>
#include <glm/glm.hpp>
#include <glm/gtc/matrix_transform.hpp>
#include <glm/gtc/type_ptr.hpp>

// متغیرهای سراسری
float aspect = 800.0f / 600.0f;
float near = 0.1f;
float far = 100.0f;
bool usePerspective = true;

// ساختار برای نگهداری اطلاعات دوربین
struct Camera {
    glm::vec3 position;
    glm::vec3 target;
    glm::vec3 up;
    
    Camera() {
        position = glm::vec3(0.0f, 0.0f, 5.0f);
        target = glm::vec3(0.0f, 0.0f, 0.0f);
        up = glm::vec3(0.0f, 1.0f, 0.0f);
    }
    
    void lookAt() {
        glm::mat4 view = glm::lookAt(position, target, up);
        glMultMatrixf(glm::value_ptr(view));
    }
    
    void move(const glm::vec3& delta) {
        position += delta;
        target += delta;
    }
    
    void rotate(float angle, const glm::vec3& axis) {
        glm::mat4 rotation = glm::rotate(glm::mat4(1.0f), angle, axis);
        position = glm::vec3(rotation * glm::vec4(position - target, 0.0f)) + target;
    }
};

Camera camera;

void display() {
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    glLoadIdentity();
    
    // اعمال ماتریس view
    camera.lookAt();
    
    // رسم یک مکعب ساده
    glColor3f(1.0f, 0.0f, 0.0f);
    glutSolidCube(1.0);
    
    glutSwapBuffers();
}

void reshape(int w, int h) {
    glViewport(0, 0, w, h);
    aspect = (float)w / (float)h;
    
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    
    if(usePerspective) {
        gluPerspective(45.0f, aspect, near, far);
    } else {
        glOrtho(-5.0f * aspect, 5.0f * aspect, -5.0f, 5.0f, near, far);
    }
    
    glMatrixMode(GL_MODELVIEW);
}

void specialKeys(int key, int x, int y) {
    float speed = 0.5f;
    glm::vec3 move(0.0f);
    
    switch(key) {
        case GLUT_KEY_LEFT:
            move.x = -speed;
            break;
        case GLUT_KEY_RIGHT:
            move.x = speed;
            break;
        case GLUT_KEY_UP:
            move.z = -speed;
            break;
        case GLUT_KEY_DOWN:
            move.z = speed;
            break;
        case GLUT_KEY_PAGE_UP:
            move.y = speed;
            break;
        case GLUT_KEY_PAGE_DOWN:
            move.y = -speed;
            break;
    }
    
    camera.move(move);
    glutPostRedisplay();
}

void keyboard(unsigned char key, int x, int y) {
    switch(key) {
        case 'r':
            camera.rotate(0.1f, glm::vec3(0.0f, 1.0f, 0.0f));
            break;
        case 'l':
            camera.rotate(-0.1f, glm::vec3(0.0f, 1.0f, 0.0f));
            break;
        case 'p':
            usePerspective = !usePerspective;
            reshape(800, 600);
            break;
        case 27:  // ESC
            exit(0);
            break;
    }
    
    glutPostRedisplay();
}

int main(int argc, char** argv) {
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB | GLUT_DEPTH);
    glutInitWindowSize(800, 600);
    glutCreateWindow("دوربین پیشرفته - درس گرافیک کامپیوتری");
    
    glEnable(GL_DEPTH_TEST);
    
    glutDisplayFunc(display);
    glutReshapeFunc(reshape);
    glutSpecialFunc(specialKeys);
    glutKeyboardFunc(keyboard);
    
    glutMainLoop();
    return 0;
}
```

## تمرینات عملی

### تمرین 1: دوربین‌های پایه
1. یک دوربین با قابلیت چرخش 360 درجه ایجاد کنید
2. امکان زوم کردن (نزدیک/دور شدن) را به دوربین اضافه کنید
3. یک دوربین سوم شخص برای دنبال کردن یک شیء متحرک ایجاد کنید
4. امکان تغییر بین حالت‌های perspective و orthographic را پیاده‌سازی کنید

### تمرین 2: دوربین‌های پیشرفته
1. یک دوربین اول شخص با کنترل ماوس پیاده‌سازی کنید
2. یک دوربین با قابلیت پن (حرکت افقی/عمودی) ایجاد کنید
3. یک دوربین با قابلیت چرخش حول یک نقطه ثابت پیاده‌سازی کنید
4. یک دوربین با قابلیت تغییر FOV (زاویه دید) ایجاد کنید

### تمرین 3: دوربین‌های تعاملی
1. یک دوربین که با کلیک و کشیدن ماوس بچرخد
2. یک دوربین که با چرخ ماوس زوم کند
3. یک دوربین که با کلیدهای WASD حرکت کند
4. یک دوربین که با ژست‌های لمسی (pinch, pan) کنترل شود

## مفاهیم پیشرفته

### 1. دوربین‌های پیشرفته
- **دوربین‌های سینمایی (Cinematic Cameras)**: برای ایجاد صحنه‌های سینمایی با حرکت‌های نرم
- **دوربین‌های چندگانه (Multi-Camera Systems)**: برای نمایش همزمان چند دیدگاه
- **دوربین‌های VR (VR Cameras)**: برای تجربیات واقعیت مجازی
- **دوربین‌های AR (AR Cameras)**: برای واقعیت افزوده

### 2. بهینه‌سازی دوربین
- **Frustum Culling**: حذف اشیاء خارج از میدان دید دوربین
- **LOD (Level of Detail)**: کاهش جزئیات برای اشیاء دور
- **Occlusion Culling**: حذف اشیاء پنهان شده توسط اشیاء دیگر
- **Viewport Culling**: بهینه‌سازی برای نمایش در چند viewport

### 3. دوربین در OpenGL مدرن
- **دوربین در شیدرها (Camera in Shaders)**: انتقال پارامترهای دوربین به شیدرها
- **دوربین با استفاده از Uniform Buffer Objects**: بهینه‌سازی انتقال داده‌های دوربین
- **دوربین با استفاده از Compute Shaders**: محاسبات پیشرفته دوربین در GPU
- **دوربین با استفاده از Transform Feedback**: ذخیره نتایج محاسبات دوربین

## نکات مهم
1. همیشه قبل از رسم، ماتریس modelview را با `glLoadIdentity()` ریست کنید
2. برای تغییر بین ماتریس‌های مختلف از `glMatrixMode()` استفاده کنید
3. در صورت استفاده از عمق، `GL_DEPTH_TEST` را فعال کنید
4. برای عملکرد بهتر، near و far plane را متناسب با صحنه تنظیم کنید
5. برای دوربین‌های تعاملی، از زمان واقعی برای به‌روزرسانی استفاده کنید
6. برای جلوگیری از مشکل gimbal lock، از کواترنیون‌ها برای چرخش استفاده کنید
7. برای دوربین‌های سوم شخص، فاصله مناسب را با توجه به اندازه صحنه تنظیم کنید
8. برای دوربین‌های orthographic، نسبت تصویر را با دقت تنظیم کنید تا اعوجاج ایجاد نشود

## منابع بیشتر
- [مستندات OpenGL - Camera](https://www.opengl.org/archives/resources/faq/technical/transformations.htm)
- [آموزش Camera در OpenGL](https://learnopengl.com/Getting-started/Camera)
- [کتاب Real-Time Rendering](https://www.realtimerendering.com/)
- [مقالات GDC درباره دوربین](https://www.gdc.com/)
- [مستندات GLM](https://github.com/g-truc/glm)
- [آموزش ViewBox در SVG](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/viewBox) 