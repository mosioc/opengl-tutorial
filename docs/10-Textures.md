---
title: بافت‌ها
nav_order: 10
permalink: /textures/
layout: persian
---


# بافت‌دهی در OpenGL

## مقدمه
بافت‌دهی (Texturing) یکی از مهم‌ترین تکنیک‌ها در گرافیک کامپیوتری است که به ما امکان می‌دهد سطح اشکال‌ها را با تصاویر یا الگوهای پیچیده پر کنیم. این تکنیک باعث می‌شود که اشکال سه‌بعدی واقعی‌تر به نظر برسند و جزئیات بیشتری داشته باشند.

## مفاهیم پایه‌ای بافت‌دهی

### 1. بافت چیست؟
- **تعریف**: تصویر دو بعدی که روی سطح یک شیء سه‌بعدی نگاشت می‌شود
- **کاربردها**: 
  - شبیه‌سازی مواد مختلف (چوب، فلز، پارچه و غیره)
  - اضافه کردن جزئیات به مدل‌های ساده
  - ایجاد محیط‌های واقعی‌تر

### 2. مختصات بافت (Texture Coordinates)
برای اعمال یک بافت روی یک شکل، هر رأس به یک مختصات بافت نیاز دارد که تعیین می‌کند چه بخشی از بافت به آن رأس اختصاص داده شود.
- **محدوده**: از (0,0) تا (1,1)
- **نام‌گذاری**: معمولاً با s و t یا u و v نمایش داده می‌شوند
- **نقشه‌برداری**: ارتباط بین مختصات بافت و مختصات سه‌بعدی
=> نکته‌های اضافی:
- (0, 0) گوشه پایین چپ تصویر است.
- (1, 1) گوشه بالا راست تصویر است.

```
(0,1)          (1,1)
   +-------------+
   |             |
   |             |
   |             |
   |             |
   +-------------+
(0,0)          (1,0)
```

### 3. پارامترهای بافت
- **فیلترینگ**: نحوه نمایش بافت در مقیاس‌های مختلف
  - **مینیفیکیشن**: کاهش اندازه بافت
  - **مگنیفیکیشن**: افزایش اندازه بافت
- **تکرار**: نحوه تکرار بافت در سطوح بزرگتر
- **حاشیه**: نحوه رفتار بافت در لبه‌های شیء


#### مثال و جزئیات دقیق از پارامترهای بافت

### حالت‌های Wrapping

تعیین می‌کنند وقتی مختصات بافت خارج از محدوده [0,1] هستند، چه اتفاقی می‌افتد:

- **GL_REPEAT**: تکرار بافت (پیش‌فرض)
- **GL_MIRRORED_REPEAT**: تکرار آینه‌ای بافت
- **GL_CLAMP_TO_EDGE**: مختصات به نزدیک‌ترین لبه محدود می‌شوند
- **GL_CLAMP_TO_BORDER**: از رنگ حاشیه تعیین شده استفاده می‌شود

```cpp
// تنظیم حالت wrapping برای محورهای S و T
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);

// تنظیم رنگ حاشیه اگر از GL_CLAMP_TO_BORDER استفاده می‌کنید
float borderColor[] = { 1.0f, 1.0f, 0.0f, 1.0f };
glTexParameterfv(GL_TEXTURE_2D, GL_TEXTURE_BORDER_COLOR, borderColor);
```

### فیلترینگ بافت

تعیین می‌کند وقتی بافت بزرگ‌تر یا کوچک‌تر از سایز اصلی نمایش داده می‌شود، چگونه نمونه‌برداری شود:

- **GL_NEAREST**: نزدیک‌ترین تکسل را انتخاب می‌کند (پیکسلی)
- **GL_LINEAR**: میانگین وزن‌دار تکسل‌های مجاور (نرم‌تر)

```cpp
// تنظیم فیلتر برای کوچک‌نمایی و بزرگ‌نمایی
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

### میپ‌مپ (Mipmaps)

نسخه‌های کوچک‌تر از بافت اصلی که برای سطوح مختلف جزئیات استفاده می‌شوند:

```cpp
// ایجاد خودکار میپ‌مپ‌ها
glGenerateMipmap(GL_TEXTURE_2D);

// تنظیم فیلتر میپ‌مپ
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
```

## فرمت‌های بافت

OpenGL از انواع مختلفی از فرمت‌های بافت پشتیبانی می‌کند:

- **GL_RGB**: سه کانال رنگی R, G, B
- **GL_RGBA**: چهار کانال R, G, B, A (آلفا برای شفافیت)
- **GL_RED**: فقط کانال قرمز
- **GL_DEPTH_COMPONENT**: برای بافت‌های عمق
- و فرمت‌های بیشتر...

```cpp
// برای یک تصویر RGBA (با کانال آلفا)
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, data);
```

## چند بافتی (Multiple Textures)

شما می‌توانید چندین بافت را همزمان در یک شیدر استفاده کنید:

```cpp
// فعال کردن واحدهای بافت
glActiveTexture(GL_TEXTURE0);
glBindTexture(GL_TEXTURE_2D, texture1);
glActiveTexture(GL_TEXTURE1);
glBindTexture(GL_TEXTURE_2D, texture2);

// تنظیم uniform در شیدر
glUniform1i(glGetUniformLocation(shaderProgram, "texture1"), 0);
glUniform1i(glGetUniformLocation(shaderProgram, "texture2"), 1);
```

و در شیدر فرگمنت:

```glsl
uniform sampler2D texture1;
uniform sampler2D texture2;

void main()
{
    FragColor = mix(texture(texture1, TexCoord), texture(texture2, TexCoord), 0.2);
}
```

## پیاده‌سازی بافت‌دهی در OpenGL

### 1. بارگذاری بافت
```cpp
// تعریف ساختار برای نگهداری اطلاعات بافت
struct Texture {
    GLuint id;
    int width;
    int height;
    int channels;
};

// تابع بارگذاری بافت از فایل
Texture loadTexture(const char* filename) {
    Texture texture;
    unsigned char* data;
    
    // استفاده از کتابخانه stb_image برای بارگذاری تصویر
    data = stbi_load(filename, &texture.width, &texture.height, &texture.channels, 0);
    if (data == NULL) {
        printf("خطا در بارگذاری بافت: %s\n", filename);
        exit(1);
    }
    
    // ایجاد شناسه بافت
    glGenTextures(1, &texture.id);
    glBindTexture(GL_TEXTURE_2D, texture.id);
    
    // تنظیم پارامترهای بافت
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
    
    // بارگذاری داده‌های بافت
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, texture.width, texture.height, 0, 
                 GL_RGB, GL_UNSIGNED_BYTE, data);
    
    // آزادسازی حافظه
    stbi_image_free(data);
    
    return texture;
}
```

### 2. نگاشت بافت روی اشکال
```cpp
// تابع رسم یک مربع با بافت
void drawTexturedQuad(Texture texture) {
    glEnable(GL_TEXTURE_2D);
    glBindTexture(GL_TEXTURE_2D, texture.id);
    
    glBegin(GL_QUADS);
        // تنظیم مختصات بافت برای هر رأس
        glTexCoord2f(0.0, 0.0); glVertex3f(-0.5, -0.5, 0.0);
        glTexCoord2f(1.0, 0.0); glVertex3f(0.5, -0.5, 0.0);
        glTexCoord2f(1.0, 1.0); glVertex3f(0.5, 0.5, 0.0);
        glTexCoord2f(0.0, 1.0); glVertex3f(-0.5, 0.5, 0.0);
    glEnd();
    
    glDisable(GL_TEXTURE_2D);
}
```

### 3. بافت‌دهی چندگانه (Multitexturing)
```cpp
// تابع تنظیم بافت‌های چندگانه
void setupMultitexturing(Texture texture1, Texture texture2) {
    // فعال‌سازی بافت‌های چندگانه
    glActiveTexture(GL_TEXTURE0);
    glEnable(GL_TEXTURE_2D);
    glBindTexture(GL_TEXTURE_2D, texture1.id);
    
    glActiveTexture(GL_TEXTURE1);
    glEnable(GL_TEXTURE_2D);
    glBindTexture(GL_TEXTURE_2D, texture2.id);
    
    // تنظیم پارامترهای بافت‌ها
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
}
```

## تکنیک‌های پیشرفته بافت‌دهی

### 1. بافت‌های محیطی (Environment Mapping)
```cpp
// تابع رسم یک کره با نگاشت محیطی
void drawEnvironmentMappedSphere() {
    const int segments = 20;
    const int rings = 20;
    const float radius = 1.0;
    
    glEnable(GL_TEXTURE_2D);
    glBindTexture(GL_TEXTURE_2D, environmentTexture.id);
    
    for (int i = 0; i < rings; i++) {
        float phi1 = M_PI * float(i) / float(rings);
        float phi2 = M_PI * float(i + 1) / float(rings);
        
        glBegin(GL_QUAD_STRIP);
        for (int j = 0; j <= segments; j++) {
            float theta = 2.0f * M_PI * float(j) / float(segments);
            
            // محاسبه مختصات سه‌بعدی
            float x1 = radius * sinf(phi1) * cosf(theta);
            float y1 = radius * cosf(phi1);
            float z1 = radius * sinf(phi1) * sinf(theta);
            
            float x2 = radius * sinf(phi2) * cosf(theta);
            float y2 = radius * cosf(phi2);
            float z2 = radius * sinf(phi2) * sinf(theta);
            
            // محاسبه مختصات بافت
            float s = float(j) / float(segments);
            float t1 = float(i) / float(rings);
            float t2 = float(i + 1) / float(rings);
            
            // تنظیم نرمال برای نگاشت محیطی
            glNormal3f(x1, y1, z1);
            glTexCoord2f(s, t1);
            glVertex3f(x1, y1, z1);
            
            glNormal3f(x2, y2, z2);
            glTexCoord2f(s, t2);
            glVertex3f(x2, y2, z2);
        }
        glEnd();
    }
    
    glDisable(GL_TEXTURE_2D);
}
```

### 2. بافت‌های نرمال (Normal Mapping)
```cpp
// تابع تنظیم بافت نرمال
void setupNormalMapping(Texture diffuseTexture, Texture normalTexture) {
    // فعال‌سازی بافت‌های چندگانه
    glActiveTexture(GL_TEXTURE0);
    glEnable(GL_TEXTURE_2D);
    glBindTexture(GL_TEXTURE_2D, diffuseTexture.id);
    
    glActiveTexture(GL_TEXTURE1);
    glEnable(GL_TEXTURE_2D);
    glBindTexture(GL_TEXTURE_2D, normalTexture.id);
    
    // تنظیم پارامترهای بافت‌ها
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
}
```

### 3. بافت‌های جابجایی (Displacement Mapping)
```cpp
// تابع رسم یک سطح با بافت جابجایی
void drawDisplacementMappedSurface(Texture displacementTexture, float scale) {
    const int gridSize = 20;
    const float size = 1.0;
    
    glEnable(GL_TEXTURE_2D);
    glBindTexture(GL_TEXTURE_2D, displacementTexture.id);
    
    for (int i = 0; i < gridSize; i++) {
        glBegin(GL_TRIANGLE_STRIP);
        for (int j = 0; j <= gridSize; j++) {
            float x1 = -size + 2.0f * size * float(i) / float(gridSize);
            float z1 = -size + 2.0f * size * float(j) / float(gridSize);
            
            float x2 = -size + 2.0f * size * float(i + 1) / float(gridSize);
            float z2 = -size + 2.0f * size * float(j) / float(gridSize);
            
            // محاسبه مختصات بافت
            float s1 = float(i) / float(gridSize);
            float t1 = float(j) / float(gridSize);
            float s2 = float(i + 1) / float(gridSize);
            float t2 = float(j) / float(gridSize);
            
            // خواندن مقدار جابجایی از بافت
            unsigned char displacement1, displacement2;
            glGetTexImage(GL_TEXTURE_2D, 0, GL_RED, GL_UNSIGNED_BYTE, &displacement1);
            glGetTexImage(GL_TEXTURE_2D, 0, GL_RED, GL_UNSIGNED_BYTE, &displacement2);
            
            float y1 = scale * float(displacement1) / 255.0f;
            float y2 = scale * float(displacement2) / 255.0f;
            
            // رسم رأس‌ها با جابجایی
            glTexCoord2f(s1, t1);
            glVertex3f(x1, y1, z1);
            
            glTexCoord2f(s2, t2);
            glVertex3f(x2, y2, z2);
        }
        glEnd();
    }
    
    glDisable(GL_TEXTURE_2D);
}
```

## بافت‌دهی با شیدرها

### 1. شیدرهای بافت
```glsl
// شیدر ورتیکس برای بافت‌دهی
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec2 aTexCoord;
layout (location = 2) in vec3 aNormal;

out vec2 TexCoord;
out vec3 Normal;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

void main() {
    gl_Position = projection * view * model * vec4(aPos, 1.0);
    TexCoord = aTexCoord;
    Normal = mat3(transpose(inverse(model))) * aNormal;
}

// شیدر فرگمنت برای بافت‌دهی
#version 330 core
out vec4 FragColor;

in vec2 TexCoord;
in vec3 Normal;

uniform sampler2D texture1;
uniform vec3 lightPos;
uniform vec3 viewPos;

void main() {
    // محاسبه نورپردازی
    vec3 norm = normalize(Normal);
    vec3 lightDir = normalize(lightPos - FragPos);
    float diff = max(dot(norm, lightDir), 0.0);
    vec3 diffuse = diff * vec3(1.0);
    
    // ترکیب رنگ بافت با نورپردازی
    vec4 texColor = texture(texture1, TexCoord);
    FragColor = vec4(texColor.rgb * diffuse, texColor.a);
}
```

### 2. بافت‌دهی چندگانه با شیدرها
```glsl
// شیدر فرگمنت برای بافت‌دهی چندگانه
#version 330 core
out vec4 FragColor;

in vec2 TexCoord;
in vec3 Normal;

uniform sampler2D diffuseTexture;
uniform sampler2D normalTexture;
uniform vec3 lightPos;
uniform vec3 viewPos;

void main() {
    // استخراج نرمال از بافت نرمال
    vec3 normal = normalize(texture(normalTexture, TexCoord).rgb * 2.0 - 1.0);
    
    // محاسبه نورپردازی
    vec3 lightDir = normalize(lightPos - FragPos);
    float diff = max(dot(normal, lightDir), 0.0);
    vec3 diffuse = diff * vec3(1.0);
    
    // ترکیب رنگ بافت با نورپردازی
    vec4 texColor = texture(diffuseTexture, TexCoord);
    FragColor = vec4(texColor.rgb * diffuse, texColor.a);
}
```

## کد نمونه کامل

```cpp
#include <GL/glut.h>
#include <cmath>
#include "stb_image.h"

// متغیرهای سراسری
Texture woodTexture;
Texture metalTexture;
float rotationAngle = 0.0;

// تابع callback برای رسم
void display() {
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    
    glLoadIdentity();
    glTranslatef(0.0, 0.0, -5.0);
    glRotatef(rotationAngle, 0.0, 1.0, 0.0);
    
    // رسم مکعب با بافت چوب
    glPushMatrix();
        glTranslatef(-1.5, 0.0, 0.0);
        drawTexturedCube(woodTexture);
    glPopMatrix();
    
    // رسم مکعب با بافت فلز
    glPushMatrix();
        glTranslatef(1.5, 0.0, 0.0);
        drawTexturedCube(metalTexture);
    glPopMatrix();
    
    glutSwapBuffers();
}

// تابع رسم مکعب با بافت
void drawTexturedCube(Texture texture) {
    glEnable(GL_TEXTURE_2D);
    glBindTexture(GL_TEXTURE_2D, texture.id);
    
    glBegin(GL_QUADS);
        // وجه جلو
        glTexCoord2f(0.0, 0.0); glVertex3f(-1.0, -1.0, 1.0);
        glTexCoord2f(1.0, 0.0); glVertex3f(1.0, -1.0, 1.0);
        glTexCoord2f(1.0, 1.0); glVertex3f(1.0, 1.0, 1.0);
        glTexCoord2f(0.0, 1.0); glVertex3f(-1.0, 1.0, 1.0);
        
        // وجه پشت
        glTexCoord2f(0.0, 0.0); glVertex3f(-1.0, -1.0, -1.0);
        glTexCoord2f(1.0, 0.0); glVertex3f(-1.0, 1.0, -1.0);
        glTexCoord2f(1.0, 1.0); glVertex3f(1.0, 1.0, -1.0);
        glTexCoord2f(0.0, 1.0); glVertex3f(1.0, -1.0, -1.0);
        
        // وجه بالا
        glTexCoord2f(0.0, 0.0); glVertex3f(-1.0, 1.0, -1.0);
        glTexCoord2f(1.0, 0.0); glVertex3f(-1.0, 1.0, 1.0);
        glTexCoord2f(1.0, 1.0); glVertex3f(1.0, 1.0, 1.0);
        glTexCoord2f(0.0, 1.0); glVertex3f(1.0, 1.0, -1.0);
        
        // وجه پایین
        glTexCoord2f(0.0, 0.0); glVertex3f(-1.0, -1.0, -1.0);
        glTexCoord2f(1.0, 0.0); glVertex3f(1.0, -1.0, -1.0);
        glTexCoord2f(1.0, 1.0); glVertex3f(1.0, -1.0, 1.0);
        glTexCoord2f(0.0, 1.0); glVertex3f(-1.0, -1.0, 1.0);
        
        // وجه راست
        glTexCoord2f(0.0, 0.0); glVertex3f(1.0, -1.0, -1.0);
        glTexCoord2f(1.0, 0.0); glVertex3f(1.0, 1.0, -1.0);
        glTexCoord2f(1.0, 1.0); glVertex3f(1.0, 1.0, 1.0);
        glTexCoord2f(0.0, 1.0); glVertex3f(1.0, -1.0, 1.0);
        
        // وجه چپ
        glTexCoord2f(0.0, 0.0); glVertex3f(-1.0, -1.0, -1.0);
        glTexCoord2f(1.0, 0.0); glVertex3f(-1.0, -1.0, 1.0);
        glTexCoord2f(1.0, 1.0); glVertex3f(-1.0, 1.0, 1.0);
        glTexCoord2f(0.0, 1.0); glVertex3f(-1.0, 1.0, -1.0);
    glEnd();
    
    glDisable(GL_TEXTURE_2D);
}

// تابع callback برای تغییر اندازه پنجره
void reshape(int w, int h) {
    glViewport(0, 0, w, h);
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    gluPerspective(45.0, (float)w / (float)h, 0.1, 100.0);
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
    }
    glutPostRedisplay();
}

// تابع callback برای تایمر
void timer(int value) {
    rotationAngle += 1.0;
    glutPostRedisplay();
    glutTimerFunc(16, timer, 0); // حدود 60 FPS
}

int main(int argc, char** argv) {
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB | GLUT_DEPTH);
    glutInitWindowSize(800, 600);
    glutInitWindowPosition(100, 100);
    glutCreateWindow("بافت‌دهی در OpenGL - درس گرافیک کامپیوتری");
    
    // فعال‌سازی عمق و نورپردازی
    glEnable(GL_DEPTH_TEST);
    glEnable(GL_LIGHTING);
    glEnable(GL_LIGHT0);
    glEnable(GL_COLOR_MATERIAL);
    
    // تنظیم نور
    GLfloat light_position[] = {5.0, 5.0, 5.0, 0.0};
    GLfloat light_ambient[] = {0.2, 0.2, 0.2, 1.0};
    GLfloat light_diffuse[] = {1.0, 1.0, 1.0, 1.0};
    glLightfv(GL_LIGHT0, GL_POSITION, light_position);
    glLightfv(GL_LIGHT0, GL_AMBIENT, light_ambient);
    glLightfv(GL_LIGHT0, GL_DIFFUSE, light_diffuse);
    
    // بارگذاری بافت‌ها
    woodTexture = loadTexture("wood.jpg");
    metalTexture = loadTexture("metal.jpg");
    
    glutDisplayFunc(display);
    glutReshapeFunc(reshape);
    glutKeyboardFunc(keyboard);
    glutTimerFunc(0, timer, 0);
    
    glutMainLoop();
    return 0;
}
```

## منابع بیشتر
- [مستندات رسمی OpenGL Texturing](https://www.khronos.org/opengl/wiki/Texture)
- [OpenGL Texture Objects](https://www.khronos.org/opengl/wiki/Texture_Object)
- [مقالات آکادمیک SIGGRAPH درباره بافت‌دهی](https://www.siggraph.org/)
- [کتاب OpenGL Programming Guide (Red Book)](https://www.opengl.org/archives/resources/code/samples/glut_examples/examples/redbook/)
- [مستندات stb_image](https://github.com/nothings/stb/blob/master/stb_image.h) 
