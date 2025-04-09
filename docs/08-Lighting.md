---
title: نورپردازی
nav_order: 8
permalink: /lightning/
layout: persian
---

# نورپردازی در OpenGL

## مقدمه
در این بخش، با مفاهیم نورپردازی در OpenGL آشنا می‌شویم. نورپردازی به اشیاء سه‌بعدی عمق و واقع‌گرایی می‌بخشد و باعث می‌شود صحنه‌های گرافیکی طبیعی‌تر به نظر برسند.

## انواع نور

### 1. نور محیطی (Ambient Light)
نور محیطی، نور یکنواختی است که از همه جهات به اشیاء می‌تابد و سایه‌ای ایجاد نمی‌کند.

```cpp
// تنظیم نور محیطی
GLfloat ambient_light[] = {0.2f, 0.2f, 0.2f, 1.0f};
glLightModelfv(GL_LIGHT_MODEL_AMBIENT, ambient_light);
```

### 2. نور پراکنده (Diffuse Light)
نور پراکنده، نور جهتمندی است که از یک منبع نور به سطوح اشیاء می‌تابد و بسته به زاویه برخورد، شدت متفاوتی دارد.

```cpp
// تنظیم نور پراکنده
GLfloat diffuse_light[] = {1.0f, 1.0f, 1.0f, 1.0f};
glLightfv(GL_LIGHT0, GL_DIFFUSE, diffuse_light);
```

### 3. نور بازتابی (Specular Light)
نور بازتابی، درخششی است که از سطوح صیقلی بازتاب می‌شود و به چشم بیننده می‌رسد.

```cpp
// تنظیم نور بازتابی
GLfloat specular_light[] = {1.0f, 1.0f, 1.0f, 1.0f};
glLightfv(GL_LIGHT0, GL_SPECULAR, specular_light);
```

## تنظیمات نورپردازی

### فعال‌سازی نورپردازی
```cpp
// فعال‌سازی نورپردازی
glEnable(GL_LIGHTING);
glEnable(GL_LIGHT0);  // فعال‌سازی نور اول
glEnable(GL_COLOR_MATERIAL);  // اجازه تغییر رنگ مواد
```

### تنظیم موقعیت نور
```cpp
// تنظیم موقعیت نور
GLfloat light_position[] = {5.0f, 5.0f, 5.0f, 1.0f};  // نور جهت‌دار
// یا
GLfloat light_position[] = {5.0f, 5.0f, 5.0f, 0.0f};  // نور نقطه‌ای
glLightfv(GL_LIGHT0, GL_POSITION, light_position);
```

### تنظیم خواص مواد
```cpp
// تنظیم خواص مواد
GLfloat mat_ambient[] = {0.2f, 0.2f, 0.2f, 1.0f};
GLfloat mat_diffuse[] = {0.8f, 0.8f, 0.8f, 1.0f};
GLfloat mat_specular[] = {1.0f, 1.0f, 1.0f, 1.0f};
GLfloat mat_shininess[] = {50.0f};

glMaterialfv(GL_FRONT, GL_AMBIENT, mat_ambient);
glMaterialfv(GL_FRONT, GL_DIFFUSE, mat_diffuse);
glMaterialfv(GL_FRONT, GL_SPECULAR, mat_specular);
glMaterialfv(GL_FRONT, GL_SHININESS, mat_shininess);
```

## محاسبه نور

### نرمال‌ها
برای محاسبه صحیح نور، باید نرمال‌های سطوح را تعریف کنیم:

```cpp
// تعریف نرمال برای یک سطح
GLfloat normal[] = {0.0f, 1.0f, 0.0f};  // نرمال به سمت بالا
glNormal3fv(normal);
```

### محاسبه نور در شیدرها (در نسخه‌های مدرن OpenGL)
```glsl
// شیدر رأس
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aNormal;

out vec3 FragPos;
out vec3 Normal;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

void main()
{
    FragPos = vec3(model * vec4(aPos, 1.0));
    Normal = mat3(transpose(inverse(model))) * aNormal;
    gl_Position = projection * view * model * vec4(aPos, 1.0);
}
```

```glsl
// شیدر فرگمنت
#version 330 core
out vec4 FragColor;

in vec3 FragPos;
in vec3 Normal;

uniform vec3 lightPos;
uniform vec3 viewPos;
uniform vec3 lightColor;
uniform vec3 objectColor;

void main()
{
    // نرمال‌سازی
    vec3 norm = normalize(Normal);
    
    // محاسبه نور پراکنده
    vec3 lightDir = normalize(lightPos - FragPos);
    float diff = max(dot(norm, lightDir), 0.0);
    vec3 diffuse = diff * lightColor;
    
    // محاسبه نور بازتابی
    float specularStrength = 0.5;
    vec3 viewDir = normalize(viewPos - FragPos);
    vec3 reflectDir = reflect(-lightDir, norm);
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), 32);
    vec3 specular = specularStrength * spec * lightColor;
    
    // نور محیطی
    vec3 ambient = 0.1 * lightColor;
    
    // ترکیب نورها
    vec3 result = (ambient + diffuse + specular) * objectColor;
    FragColor = vec4(result, 1.0);
}
```

## مثال کامل: مکعب با نورپردازی

```cpp
#include <GL/glut.h>
#include <cmath>

// متغیرهای سراسری
float angle = 0.0f;
float light_angle = 0.0f;

void setupLighting() {
    // فعال‌سازی نورپردازی
    glEnable(GL_LIGHTING);
    glEnable(GL_LIGHT0);
    glEnable(GL_COLOR_MATERIAL);
    glEnable(GL_NORMALIZE);
    
    // تنظیم نور محیطی
    GLfloat ambient_light[] = {0.2f, 0.2f, 0.2f, 1.0f};
    glLightModelfv(GL_LIGHT_MODEL_AMBIENT, ambient_light);
    
    // تنظیم نور پراکنده
    GLfloat diffuse_light[] = {1.0f, 1.0f, 1.0f, 1.0f};
    glLightfv(GL_LIGHT0, GL_DIFFUSE, diffuse_light);
    
    // تنظیم نور بازتابی
    GLfloat specular_light[] = {1.0f, 1.0f, 1.0f, 1.0f};
    glLightfv(GL_LIGHT0, GL_SPECULAR, specular_light);
    
    // تنظیم خواص مواد
    GLfloat mat_ambient[] = {0.2f, 0.2f, 0.2f, 1.0f};
    GLfloat mat_diffuse[] = {0.8f, 0.8f, 0.8f, 1.0f};
    GLfloat mat_specular[] = {1.0f, 1.0f, 1.0f, 1.0f};
    GLfloat mat_shininess[] = {50.0f};
    
    glMaterialfv(GL_FRONT, GL_AMBIENT, mat_ambient);
    glMaterialfv(GL_FRONT, GL_DIFFUSE, mat_diffuse);
    glMaterialfv(GL_FRONT, GL_SPECULAR, mat_specular);
    glMaterialfv(GL_FRONT, GL_SHININESS, mat_shininess);
}

void display() {
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    
    // تنظیم ماتریس مدل
    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();
    glTranslatef(0.0f, 0.0f, -5.0f);
    glRotatef(angle, 0.0f, 1.0f, 0.0f);
    
    // تنظیم موقعیت نور
    GLfloat light_position[] = {
        5.0f * cos(light_angle),
        5.0f * sin(light_angle),
        5.0f,
        1.0f
    };
    glLightfv(GL_LIGHT0, GL_POSITION, light_position);
    
    // رسم مکعب
    glBegin(GL_QUADS);
        // وجه جلو
        glNormal3f(0.0f, 0.0f, 1.0f);
        glColor3f(1.0f, 0.0f, 0.0f);
        glVertex3f(-1.0f, -1.0f, 1.0f);
        glVertex3f(1.0f, -1.0f, 1.0f);
        glVertex3f(1.0f, 1.0f, 1.0f);
        glVertex3f(-1.0f, 1.0f, 1.0f);
        
        // وجه پشت
        glNormal3f(0.0f, 0.0f, -1.0f);
        glColor3f(0.0f, 1.0f, 0.0f);
        glVertex3f(-1.0f, -1.0f, -1.0f);
        glVertex3f(-1.0f, 1.0f, -1.0f);
        glVertex3f(1.0f, 1.0f, -1.0f);
        glVertex3f(1.0f, -1.0f, -1.0f);
        
        // وجه بالا
        glNormal3f(0.0f, 1.0f, 0.0f);
        glColor3f(0.0f, 0.0f, 1.0f);
        glVertex3f(-1.0f, 1.0f, -1.0f);
        glVertex3f(-1.0f, 1.0f, 1.0f);
        glVertex3f(1.0f, 1.0f, 1.0f);
        glVertex3f(1.0f, 1.0f, -1.0f);
        
        // وجه پایین
        glNormal3f(0.0f, -1.0f, 0.0f);
        glColor3f(1.0f, 1.0f, 0.0f);
        glVertex3f(-1.0f, -1.0f, -1.0f);
        glVertex3f(1.0f, -1.0f, -1.0f);
        glVertex3f(1.0f, -1.0f, 1.0f);
        glVertex3f(-1.0f, -1.0f, 1.0f);
        
        // وجه راست
        glNormal3f(1.0f, 0.0f, 0.0f);
        glColor3f(1.0f, 0.0f, 1.0f);
        glVertex3f(1.0f, -1.0f, -1.0f);
        glVertex3f(1.0f, 1.0f, -1.0f);
        glVertex3f(1.0f, 1.0f, 1.0f);
        glVertex3f(1.0f, -1.0f, 1.0f);
        
        // وجه چپ
        glNormal3f(-1.0f, 0.0f, 0.0f);
        glColor3f(0.0f, 1.0f, 1.0f);
        glVertex3f(-1.0f, -1.0f, -1.0f);
        glVertex3f(-1.0f, -1.0f, 1.0f);
        glVertex3f(-1.0f, 1.0f, 1.0f);
        glVertex3f(-1.0f, 1.0f, -1.0f);
    glEnd();
    
    glutSwapBuffers();
}

void reshape(int w, int h) {
    glViewport(0, 0, w, h);
    
    // تنظیم ماتریس برجستگی
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    gluPerspective(45.0f, (float)w/h, 0.1f, 100.0f);
    
    glMatrixMode(GL_MODELVIEW);
}

void keyboard(unsigned char key, int x, int y) {
    switch(key) {
        case 27:  // ESC
            exit(0);
            break;
    }
}

void update(int value) {
    // چرخش مکعب
    angle += 2.0f;
    if(angle > 360.0f) {
        angle -= 360.0f;
    }
    
    // چرخش نور
    light_angle += 0.05f;
    if(light_angle > 2 * M_PI) {
        light_angle -= 2 * M_PI;
    }
    
    glutPostRedisplay();
    glutTimerFunc(16, update, 0);  // حدود 60 FPS
}

int main(int argc, char** argv) {
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB | GLUT_DEPTH);
    glutInitWindowSize(800, 600);
    glutCreateWindow("نورپردازی در OpenGL");
    
    glEnable(GL_DEPTH_TEST);
    glClearColor(0.0f, 0.0f, 0.0f, 1.0f);
    
    setupLighting();
    
    glutDisplayFunc(display);
    glutReshapeFunc(reshape);
    glutKeyboardFunc(keyboard);
    glutTimerFunc(0, update, 0);
    
    glutMainLoop();
    return 0;
}
```

## انواع نورپردازی پیشرفته

### 1. نورپردازی نقطه‌ای (Point Light)
نوری که از یک نقطه در فضا به همه جهات منتشر می‌شود.

```cpp
// تنظیم نور نقطه‌ای
GLfloat light_position[] = {x, y, z, 1.0f};  // پارامتر آخر 1.0 برای نور نقطه‌ای
glLightfv(GL_LIGHT0, GL_POSITION, light_position);

// تنظیم تضعیف نور با فاصله
glLightf(GL_LIGHT0, GL_CONSTANT_ATTENUATION, 1.0f);
glLightf(GL_LIGHT0, GL_LINEAR_ATTENUATION, 0.0f);
glLightf(GL_LIGHT0, GL_QUADRATIC_ATTENUATION, 0.0f);
```

### 2. نورپردازی جهت‌دار (Directional Light)
نوری که از یک جهت خاص با شدت یکسان به همه نقاط می‌تابد.

```cpp
// تنظیم نور جهت‌دار
GLfloat light_position[] = {x, y, z, 0.0f};  // پارامتر آخر 0.0 برای نور جهت‌دار
glLightfv(GL_LIGHT0, GL_POSITION, light_position);
```

### 3. نورپردازی لکه‌ای (Spot Light)
نوری که از یک نقطه در یک جهت خاص با زاویه محدود منتشر می‌شود.

```cpp
// تنظیم نور لکه‌ای
GLfloat light_position[] = {x, y, z, 1.0f};
GLfloat light_direction[] = {dx, dy, dz};
GLfloat spot_cutoff = 45.0f;  // زاویه باز شدن نور (درجه)
GLfloat spot_exponent = 2.0f;  // نرمی لبه نور

glLightfv(GL_LIGHT0, GL_POSITION, light_position);
glLightfv(GL_LIGHT0, GL_SPOT_DIRECTION, light_direction);
glLightf(GL_LIGHT0, GL_SPOT_CUTOFF, spot_cutoff);
glLightf(GL_LIGHT0, GL_SPOT_EXPONENT, spot_exponent);
```

## سایه‌ها (Shadows)

برای ایجاد سایه در OpenGL، می‌توان از تکنیک‌های مختلفی استفاده کرد:

### 1. سایه‌های صفحه‌ای (Planar Shadows)
```cpp
void drawShadow() {
    glPushMatrix();
    
    // ماتریس مسطح کردن برای سایه
    GLfloat matrix[16] = {
        1.0f, 0.0f, 0.0f, 0.0f,
        0.0f, 1.0f, 0.0f, 0.0f,
        light_x, light_y, light_z, 1.0f,
        0.0f, 0.0f, 0.0f, 1.0f
    };
    
    glMultMatrixf(matrix);
    
    // رسم شیء با رنگ تیره
    glColor4f(0.0f, 0.0f, 0.0f, 0.5f);
    drawObject();
    
    glPopMatrix();
}
```

### 2. سایه‌های حجمی (Shadow Volumes)
تکنیک پیشرفته‌تری که نیاز به محاسبات بیشتری دارد و در اینجا توضیح داده نمی‌شود.

## نکات مهم
1. همیشه نرمال‌ها را برای سطوح تعریف کنید
2. برای نورپردازی واقع‌گرایانه، از ترکیب نورهای محیطی، پراکنده و بازتابی استفاده کنید
3. از `glEnable(GL_NORMALIZE)` برای نرمال‌سازی خودکار نرمال‌ها استفاده کنید
4. برای سایه‌های واقع‌گرایانه، از تکنیک‌های پیشرفته‌تر استفاده کنید

## منابع بیشتر
- [مستندات OpenGL](https://www.opengl.org/documentation/)
- [راهنمای نورپردازی](https://www.opengl.org/wiki/Lighting) 
