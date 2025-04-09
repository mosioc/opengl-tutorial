---
title: انیمیشن
nav_order: 7
permalink: /animation/
layout: persian
---

# انیمیشن در OpenGL 
## مقدمه
در این بخش، با مفاهیم پیشرفته انیمیشن در OpenGL آشنا می‌شویم. انیمیشن یکی از مهم‌ترین جنبه‌های گرافیک کامپیوتری است که به ما امکان می‌دهد صحنه‌های پویا و تعاملی ایجاد کنیم. در این درس، علاوه بر روش‌های پایه‌ای انیمیشن، تکنیک‌های پیشرفته‌تر مانند انیمیشن کاراکتر، سیستم‌های ذره‌ای، و انیمیشن‌های فیزیکی را نیز بررسی می‌کنیم.

## مفاهیم پایه‌ای انیمیشن

### 1. اصول انیمیشن
- **فریم‌ریت (Frame Rate)**: تعداد فریم‌های نمایش داده شده در هر ثانیه
- **فریم‌کی (Keyframe)**: نقاط کلیدی در انیمیشن که حالت‌های اصلی حرکت را تعریف می‌کنند
- **میان‌فریم‌سازی (In-betweening)**: محاسبه حالت‌های میانی بین فریم‌های کلیدی
- **زمان‌بندی (Timing)**: کنترل سرعت و ریتم انیمیشن

### 2. روش‌های ایجاد انیمیشن

#### استفاده از glutTimerFunc
```cpp
void update(int value) {
    // به‌روزرسانی متغیرهای انیمیشن
    angle += 2.0f;
    if(angle > 360.0f) {
        angle -= 360.0f;
    }
    
    glutPostRedisplay();
    glutTimerFunc(16, update, 0);  // فراخوانی مجدد بعد از 16 میلی‌ثانیه (حدود 60 FPS)
}

int main(int argc, char** argv) {
    // ... کد قبلی ...
    glutTimerFunc(0, update, 0);
    glutMainLoop();
    return 0;
}
```

#### استفاده از glutIdleFunc
```cpp
void update() {
    // به‌روزرسانی متغیرهای انیمیشن
    angle += 2.0f;
    if(angle > 360.0f) {
        angle -= 360.0f;
    }
    
    glutPostRedisplay();
}

int main(int argc, char** argv) {
    // ... کد قبلی ...
    glutIdleFunc(update);
    glutMainLoop();
    return 0;
}
```

## کنترل فریم‌ریت

### 1. محدود کردن فریم‌ریت
```cpp
#include <ctime>

const int FPS = 60;
const int FRAME_DELAY = 1000 / FPS;

void update(int value) {
    static clock_t lastTime = clock();
    clock_t currentTime = clock();
    
    if(currentTime - lastTime >= FRAME_DELAY) {
        // به‌روزرسانی انیمیشن
        angle += 2.0f;
        if(angle > 360.0f) {
            angle -= 360.0f;
        }
        
        lastTime = currentTime;
        glutPostRedisplay();
    }
    
    glutTimerFunc(1, update, 0);
}
```

### 2. محاسبه فریم‌ریت
```cpp
void display() {
    static clock_t lastTime = clock();
    static int frameCount = 0;
    static float fps = 0.0f;
    
    clock_t currentTime = clock();
    frameCount++;
    
    if(currentTime - lastTime >= 1000) {  // هر ثانیه
        fps = frameCount * 1000.0f / (currentTime - lastTime);
        frameCount = 0;
        lastTime = currentTime;
        printf("FPS: %.2f\n", fps);
    }
    
    // ... کد رسم ...
}
```

### 3. انیمیشن مستقل از زمان
برای اطمینان از ثبات انیمیشن در سیستم‌های مختلف، بهتر است از زمان واقعی استفاده کنیم:

```cpp
void update(int value) {
    static clock_t lastTime = clock();
    clock_t currentTime = clock();
    float deltaTime = (currentTime - lastTime) / 1000.0f;  // زمان به ثانیه
    
    // به‌روزرسانی انیمیشن بر اساس زمان واقعی
    angle += 90.0f * deltaTime;  // 90 درجه در ثانیه
    if(angle > 360.0f) {
        angle -= 360.0f;
    }
    
    lastTime = currentTime;
    glutPostRedisplay();
    glutTimerFunc(1, update, 0);
}
```

## تکنیک‌های پیشرفته انیمیشن

### 1. انیمیشن فریم‌کلیدی (Keyframe Animation)
```cpp
struct Keyframe {
    float time;
    glm::vec3 position;
    glm::vec3 rotation;
    glm::vec3 scale;
};

std::vector<Keyframe> keyframes = {
    {0.0f, glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(1.0f, 1.0f, 1.0f)},
    {1.0f, glm::vec3(1.0f, 1.0f, 0.0f), glm::vec3(0.0f, 90.0f, 0.0f), glm::vec3(1.5f, 1.5f, 1.5f)},
    {2.0f, glm::vec3(0.0f, 2.0f, 0.0f), glm::vec3(0.0f, 180.0f, 0.0f), glm::vec3(1.0f, 1.0f, 1.0f)},
    {3.0f, glm::vec3(-1.0f, 1.0f, 0.0f), glm::vec3(0.0f, 270.0f, 0.0f), glm::vec3(0.5f, 0.5f, 0.5f)},
    {4.0f, glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(0.0f, 360.0f, 0.0f), glm::vec3(1.0f, 1.0f, 1.0f)}
};

float animationTime = 0.0f;
const float animationDuration = 4.0f;

void updateAnimation(float deltaTime) {
    animationTime += deltaTime;
    if(animationTime > animationDuration) {
        animationTime -= animationDuration;
    }
    
    // یافتن فریم‌های کلیدی قبل و بعد از زمان فعلی
    int nextKeyframeIndex = 0;
    for(int i = 0; i < keyframes.size(); i++) {
        if(keyframes[i].time > animationTime) {
            nextKeyframeIndex = i;
            break;
        }
    }
    
    int prevKeyframeIndex = (nextKeyframeIndex - 1 + keyframes.size()) % keyframes.size();
    
    // محاسبه ضریب درون‌یابی
    float t = (animationTime - keyframes[prevKeyframeIndex].time) / 
              (keyframes[nextKeyframeIndex].time - keyframes[prevKeyframeIndex].time);
    
    // درون‌یابی خطی بین فریم‌های کلیدی
    glm::vec3 position = glm::mix(keyframes[prevKeyframeIndex].position, 
                                 keyframes[nextKeyframeIndex].position, t);
    glm::vec3 rotation = glm::mix(keyframes[prevKeyframeIndex].rotation, 
                                 keyframes[nextKeyframeIndex].rotation, t);
    glm::vec3 scale = glm::mix(keyframes[prevKeyframeIndex].scale, 
                              keyframes[nextKeyframeIndex].scale, t);
    
    // اعمال تبدیلات
    glPushMatrix();
    glTranslatef(position.x, position.y, position.z);
    glRotatef(rotation.x, 1.0f, 0.0f, 0.0f);
    glRotatef(rotation.y, 0.0f, 1.0f, 0.0f);
    glRotatef(rotation.z, 0.0f, 0.0f, 1.0f);
    glScalef(scale.x, scale.y, scale.z);
    
    // رسم شیء
    drawObject();
    
    glPopMatrix();
}
```

### 2. انیمیشن اسپلاین (Spline Animation)
```cpp
struct SplinePoint {
    glm::vec3 position;
    glm::vec3 tangent;
};

std::vector<SplinePoint> splinePoints = {
    {glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(1.0f, 0.0f, 0.0f)},
    {glm::vec3(1.0f, 1.0f, 0.0f), glm::vec3(0.0f, 1.0f, 0.0f)},
    {glm::vec3(0.0f, 2.0f, 0.0f), glm::vec3(-1.0f, 0.0f, 0.0f)},
    {glm::vec3(-1.0f, 1.0f, 0.0f), glm::vec3(0.0f, -1.0f, 0.0f)}
};

float splineTime = 0.0f;
const float splineDuration = 4.0f;

glm::vec3 calculateSplinePoint(float t) {
    int segmentCount = splinePoints.size() - 1;
    float segmentTime = t * segmentCount;
    int segmentIndex = (int)segmentTime;
    float segmentT = segmentTime - segmentIndex;
    
    if(segmentIndex >= segmentCount) {
        segmentIndex = segmentCount - 1;
        segmentT = 1.0f;
    }
    
    // محاسبه نقطه روی اسپلاین با استفاده از فرمول هرمیت
    glm::vec3 p0 = splinePoints[segmentIndex].position;
    glm::vec3 m0 = splinePoints[segmentIndex].tangent;
    glm::vec3 p1 = splinePoints[segmentIndex + 1].position;
    glm::vec3 m1 = splinePoints[segmentIndex + 1].tangent;
    
    float t2 = t * t;
    float t3 = t2 * t;
    
    glm::vec3 position = (2.0f * t3 - 3.0f * t2 + 1.0f) * p0 +
                         (t3 - 2.0f * t2 + t) * m0 +
                         (-2.0f * t3 + 3.0f * t2) * p1 +
                         (t3 - t2) * m1;
    
    return position;
}

void updateSplineAnimation(float deltaTime) {
    splineTime += deltaTime;
    if(splineTime > splineDuration) {
        splineTime -= splineDuration;
    }
    
    float t = splineTime / splineDuration;
    glm::vec3 position = calculateSplinePoint(t);
    
    // اعمال موقعیت به شیء
    glPushMatrix();
    glTranslatef(position.x, position.y, position.z);
    
    // محاسبه جهت حرکت برای چرخش شیء
    float nextT = (t + 0.01f) > 1.0f ? 1.0f : (t + 0.01f);
    glm::vec3 nextPosition = calculateSplinePoint(nextT);
    glm::vec3 direction = glm::normalize(nextPosition - position);
    
    // محاسبه زاویه چرخش
    float angle = glm::degrees(glm::acos(glm::dot(direction, glm::vec3(1.0f, 0.0f, 0.0f))));
    glm::vec3 rotationAxis = glm::cross(glm::vec3(1.0f, 0.0f, 0.0f), direction);
    
    if(glm::length(rotationAxis) > 0.001f) {
        glRotatef(angle, rotationAxis.x, rotationAxis.y, rotationAxis.z);
    }
    
    // رسم شیء
    drawObject();
    
    glPopMatrix();
}
```

### 3. انیمیشن کاراکتر (Character Animation)
```cpp
struct Joint {
    glm::vec3 position;
    glm::vec3 rotation;
    std::vector<int> childIndices;
};

struct Skeleton {
    std::vector<Joint> joints;
    std::vector<glm::mat4> jointMatrices;
};

Skeleton skeleton;
std::vector<glm::mat4> boneOffsets;

void initSkeleton() {
    // تعریف مفاصل اسکلت
    skeleton.joints.resize(5);
    
    // مفصل ریشه (لگن)
    skeleton.joints[0].position = glm::vec3(0.0f, 0.0f, 0.0f);
    skeleton.joints[0].rotation = glm::vec3(0.0f, 0.0f, 0.0f);
    skeleton.joints[0].childIndices = {1, 2};
    
    // مفصل ستون فقرات
    skeleton.joints[1].position = glm::vec3(0.0f, 1.0f, 0.0f);
    skeleton.joints[1].rotation = glm::vec3(0.0f, 0.0f, 0.0f);
    skeleton.joints[1].childIndices = {3};
    
    // مفصل سر
    skeleton.joints[2].position = glm::vec3(0.0f, 2.0f, 0.0f);
    skeleton.joints[2].rotation = glm::vec3(0.0f, 0.0f, 0.0f);
    skeleton.joints[2].childIndices = {};
    
    // مفصل بازوی چپ
    skeleton.joints[3].position = glm::vec3(-0.5f, 1.0f, 0.0f);
    skeleton.joints[3].rotation = glm::vec3(0.0f, 0.0f, 0.0f);
    skeleton.joints[3].childIndices = {4};
    
    // مفصل ساعد چپ
    skeleton.joints[4].position = glm::vec3(-1.0f, 1.0f, 0.0f);
    skeleton.joints[4].rotation = glm::vec3(0.0f, 0.0f, 0.0f);
    skeleton.joints[4].childIndices = {};
    
    // محاسبه ماتریس‌های استخوان
    boneOffsets.resize(skeleton.joints.size());
    for(size_t i = 0; i < skeleton.joints.size(); i++) {
        boneOffsets[i] = glm::translate(glm::mat4(1.0f), skeleton.joints[i].position);
    }
    
    // مقداردهی اولیه ماتریس‌های مفصل
    skeleton.jointMatrices.resize(skeleton.joints.size());
    for(size_t i = 0; i < skeleton.joints.size(); i++) {
        skeleton.jointMatrices[i] = glm::mat4(1.0f);
    }
}

void updateSkeleton(int jointIndex, const glm::mat4& parentMatrix) {
    // محاسبه ماتریس تبدیل فعلی
    glm::mat4 localMatrix = glm::translate(glm::mat4(1.0f), skeleton.joints[jointIndex].position);
    localMatrix = localMatrix * glm::rotate(glm::mat4(1.0f), glm::radians(skeleton.joints[jointIndex].rotation.x), glm::vec3(1.0f, 0.0f, 0.0f));
    localMatrix = localMatrix * glm::rotate(glm::mat4(1.0f), glm::radians(skeleton.joints[jointIndex].rotation.y), glm::vec3(0.0f, 1.0f, 0.0f));
    localMatrix = localMatrix * glm::rotate(glm::mat4(1.0f), glm::radians(skeleton.joints[jointIndex].rotation.z), glm::vec3(0.0f, 0.0f, 1.0f));
    
    // محاسبه ماتریس جهانی
    skeleton.jointMatrices[jointIndex] = parentMatrix * localMatrix;
    
    // به‌روزرسانی مفاصل فرزند
    for(int childIndex : skeleton.joints[jointIndex].childIndices) {
        updateSkeleton(childIndex, skeleton.jointMatrices[jointIndex]);
    }
}

void animateCharacter(float time) {
    // به‌روزرسانی چرخش مفاصل بر اساس زمان
    skeleton.joints[0].rotation.y = 30.0f * sin(time);  // چرخش لگن
    skeleton.joints[3].rotation.x = 45.0f * sin(time * 2.0f);  // چرخش بازو
    
    // به‌روزرسانی اسکلت
    updateSkeleton(0, glm::mat4(1.0f));
    
    // رسم اسکلت
    drawSkeleton();
}

void drawSkeleton() {
    for(size_t i = 0; i < skeleton.joints.size(); i++) {
        // رسم مفصل
        glPushMatrix();
        glMultMatrixf(glm::value_ptr(skeleton.jointMatrices[i]));
        drawJoint();
        glPopMatrix();
        
        // رسم استخوان‌ها
        for(int childIndex : skeleton.joints[i].childIndices) {
            glPushMatrix();
            glLoadIdentity();
            
            // رسم خط بین مفصل فعلی و مفصل فرزند
            glBegin(GL_LINES);
            glVertex3f(skeleton.jointMatrices[i][3].x, skeleton.jointMatrices[i][3].y, skeleton.jointMatrices[i][3].z);
            glVertex3f(skeleton.jointMatrices[childIndex][3].x, skeleton.jointMatrices[childIndex][3].y, skeleton.jointMatrices[childIndex][3].z);
            glEnd();
            
            glPopMatrix();
        }
    }
}
```

### 4. سیستم ذره‌ای (Particle System)
```cpp
struct Particle {
    glm::vec3 position;
    glm::vec3 velocity;
    glm::vec3 acceleration;
    glm::vec3 color;
    float life;
    float maxLife;
    float size;
};

class ParticleSystem {
private:
    std::vector<Particle> particles;
    glm::vec3 emitterPosition;
    glm::vec3 emitterVelocity;
    float emissionRate;
    float timeSinceLastEmission;
    
public:
    ParticleSystem(const glm::vec3& position, float rate) 
        : emitterPosition(position), emissionRate(rate), timeSinceLastEmission(0.0f) {}
    
    void update(float deltaTime) {
        // به‌روزرسانی ذرات موجود
        for(auto it = particles.begin(); it != particles.end();) {
            // به‌روزرسانی موقعیت و سرعت
            it->velocity += it->acceleration * deltaTime;
            it->position += it->velocity * deltaTime;
            
            // به‌روزرسانی عمر
            it->life -= deltaTime;
            
            // حذف ذرات مرده
            if(it->life <= 0.0f) {
                it = particles.erase(it);
            } else {
                ++it;
            }
        }
        
        // تولید ذرات جدید
        timeSinceLastEmission += deltaTime;
        float emissionInterval = 1.0f / emissionRate;
        
        while(timeSinceLastEmission >= emissionInterval) {
            emitParticle();
            timeSinceLastEmission -= emissionInterval;
        }
    }
    
    void emitParticle() {
        Particle particle;
        
        // تنظیم موقعیت اولیه
        particle.position = emitterPosition;
        
        // تنظیم سرعت اولیه با کمی تغییرات تصادفی
        float angle = (rand() % 360) * 3.14159f / 180.0f;
        float speed = 1.0f + (rand() % 100) / 100.0f;
        particle.velocity = emitterVelocity + glm::vec3(cos(angle) * speed, (rand() % 100) / 50.0f, sin(angle) * speed);
        
        // تنظیم شتاب (مثلاً گرانش)
        particle.acceleration = glm::vec3(0.0f, -9.8f, 0.0f);
        
        // تنظیم رنگ
        particle.color = glm::vec3(1.0f, 0.5f, 0.0f);  // نارنجی
        
        // تنظیم عمر
        particle.maxLife = 2.0f + (rand() % 100) / 100.0f;
        particle.life = particle.maxLife;
        
        // تنظیم اندازه
        particle.size = 0.1f + (rand() % 50) / 1000.0f;
        
        particles.push_back(particle);
    }
    
    void render() {
        glEnable(GL_BLEND);
        glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
        
        for(const auto& particle : particles) {
            // محاسبه ضریب شفافیت بر اساس عمر
            float alpha = particle.life / particle.maxLife;
            
            glPushMatrix();
            glTranslatef(particle.position.x, particle.position.y, particle.position.z);
            
            // تنظیم رنگ با شفافیت
            glColor4f(particle.color.x, particle.color.y, particle.color.z, alpha);
            
            // رسم ذره به صورت نقطه یا مربع
            glPointSize(particle.size * 10.0f);
            glBegin(GL_POINTS);
            glVertex3f(0.0f, 0.0f, 0.0f);
            glEnd();
            
            glPopMatrix();
        }
        
        glDisable(GL_BLEND);
    }
};

// استفاده از سیستم ذره‌ای
ParticleSystem fireSystem(glm::vec3(0.0f, 0.0f, 0.0f), 50.0f);  // 50 ذره در ثانیه

void updateParticles(float deltaTime) {
    fireSystem.update(deltaTime);
}

void renderParticles() {
    fireSystem.render();
}
```

### 5. انیمیشن فیزیکی (Physics Animation)
```cpp
struct RigidBody {
    glm::vec3 position;
    glm::vec3 velocity;
    glm::vec3 acceleration;
    glm::vec3 angularVelocity;
    glm::vec3 angularAcceleration;
    glm::quat rotation;
    float mass;
    float restitution;  // ضریب بازگشت
};

void updatePhysics(RigidBody& body, float deltaTime) {
    // به‌روزرسانی سرعت خطی
    body.velocity += body.acceleration * deltaTime;
    
    // به‌روزرسانی موقعیت
    body.position += body.velocity * deltaTime;
    
    // به‌روزرسانی سرعت زاویه‌ای
    body.angularVelocity += body.angularAcceleration * deltaTime;
    
    // به‌روزرسانی چرخش
    glm::quat spin(0.0f, body.angularVelocity * deltaTime * 0.5f);
    body.rotation = body.rotation + spin * body.rotation;
    body.rotation = glm::normalize(body.rotation);
    
    // برخورد با زمین
    if(body.position.y < 0.0f) {
        body.position.y = 0.0f;
        body.velocity.y = -body.velocity.y * body.restitution;
        
        // کاهش سرعت افقی به دلیل اصطکاک
        body.velocity.x *= 0.95f;
        body.velocity.z *= 0.95f;
    }
}

void renderRigidBody(const RigidBody& body) {
    glPushMatrix();
    
    // اعمال تبدیلات
    glTranslatef(body.position.x, body.position.y, body.position.z);
    
    // تبدیل کواترنیون به ماتریس چرخش
    glm::mat4 rotationMatrix = glm::mat4_cast(body.rotation);
    glMultMatrixf(glm::value_ptr(rotationMatrix));
    
    // رسم جسم
    drawObject();
    
    glPopMatrix();
}
```

## مثال کامل: انیمیشن مکعب چرخان

```cpp
#include <GL/glut.h>
#include <cmath>
#include <ctime>
#include <vector>
#include <glm/glm.hpp>
#include <glm/gtc/matrix_transform.hpp>
#include <glm/gtc/type_ptr.hpp>

// متغیرهای سراسری
float angle = 0.0f;
float scale = 1.0f;
bool growing = true;
const int FPS = 60;
const int FRAME_DELAY = 1000 / FPS;

// سیستم ذره‌ای
struct Particle {
    glm::vec3 position;
    glm::vec3 velocity;
    glm::vec3 color;
    float life;
    float maxLife;
};

std::vector<Particle> particles;
glm::vec3 emitterPosition(0.0f, 2.0f, 0.0f);

void emitParticle() {
    Particle particle;
    
    // تنظیم موقعیت اولیه
    particle.position = emitterPosition;
    
    // تنظیم سرعت اولیه با کمی تغییرات تصادفی
    float angle = (rand() % 360) * 3.14159f / 180.0f;
    float speed = 0.5f + (rand() % 100) / 200.0f;
    particle.velocity = glm::vec3(cos(angle) * speed, (rand() % 100) / 50.0f, sin(angle) * speed);
    
    // تنظیم رنگ
    particle.color = glm::vec3(1.0f, 0.5f, 0.0f);  // نارنجی
    
    // تنظیم عمر
    particle.maxLife = 1.0f + (rand() % 100) / 100.0f;
    particle.life = particle.maxLife;
    
    particles.push_back(particle);
}

void updateParticles(float deltaTime) {
    // به‌روزرسانی ذرات موجود
    for(auto it = particles.begin(); it != particles.end();) {
        // به‌روزرسانی موقعیت
        it->position += it->velocity * deltaTime;
        
        // اعمال گرانش
        it->velocity.y -= 9.8f * deltaTime;
        
        // به‌روزرسانی عمر
        it->life -= deltaTime;
        
        // حذف ذرات مرده
        if(it->life <= 0.0f) {
            it = particles.erase(it);
        } else {
            ++it;
        }
    }
    
    // تولید ذرات جدید
    static float timeSinceLastEmission = 0.0f;
    timeSinceLastEmission += deltaTime;
    
    if(timeSinceLastEmission >= 0.05f) {  // 20 ذره در ثانیه
        emitParticle();
        timeSinceLastEmission = 0.0f;
    }
}

void renderParticles() {
    glEnable(GL_BLEND);
    glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
    
    for(const auto& particle : particles) {
        // محاسبه ضریب شفافیت بر اساس عمر
        float alpha = particle.life / particle.maxLife;
        
        glPushMatrix();
        glTranslatef(particle.position.x, particle.position.y, particle.position.z);
        
        // تنظیم رنگ با شفافیت
        glColor4f(particle.color.x, particle.color.y, particle.color.z, alpha);
        
        // رسم ذره به صورت نقطه
        glPointSize(5.0f);
        glBegin(GL_POINTS);
        glVertex3f(0.0f, 0.0f, 0.0f);
        glEnd();
        
        glPopMatrix();
    }
    
    glDisable(GL_BLEND);
}

void display() {
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    
    // تنظیم ماتریس مدل
    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();
    glTranslatef(0.0f, 0.0f, -5.0f);
    
    // رسم مکعب
    glPushMatrix();
    glRotatef(angle, 0.0f, 1.0f, 0.0f);
    glScalef(scale, scale, scale);
    
    glBegin(GL_QUADS);
        // وجه جلو
        glColor3f(1.0f, 0.0f, 0.0f);
        glVertex3f(-1.0f, -1.0f, 1.0f);
        glVertex3f(1.0f, -1.0f, 1.0f);
        glVertex3f(1.0f, 1.0f, 1.0f);
        glVertex3f(-1.0f, 1.0f, 1.0f);
        
        // وجه پشت
        glColor3f(0.0f, 1.0f, 0.0f);
        glVertex3f(-1.0f, -1.0f, -1.0f);
        glVertex3f(-1.0f, 1.0f, -1.0f);
        glVertex3f(1.0f, 1.0f, -1.0f);
        glVertex3f(1.0f, -1.0f, -1.0f);
        
        // وجه بالا
        glColor3f(0.0f, 0.0f, 1.0f);
        glVertex3f(-1.0f, 1.0f, -1.0f);
        glVertex3f(-1.0f, 1.0f, 1.0f);
        glVertex3f(1.0f, 1.0f, 1.0f);
        glVertex3f(1.0f, 1.0f, -1.0f);
        
        // وجه پایین
        glColor3f(1.0f, 1.0f, 0.0f);
        glVertex3f(-1.0f, -1.0f, -1.0f);
        glVertex3f(1.0f, -1.0f, -1.0f);
        glVertex3f(1.0f, -1.0f, 1.0f);
        glVertex3f(-1.0f, -1.0f, 1.0f);
        
        // وجه راست
        glColor3f(1.0f, 0.0f, 1.0f);
        glVertex3f(1.0f, -1.0f, -1.0f);
        glVertex3f(1.0f, 1.0f, -1.0f);
        glVertex3f(1.0f, 1.0f, 1.0f);
        glVertex3f(1.0f, -1.0f, 1.0f);
        
        // وجه چپ
        glColor3f(0.0f, 1.0f, 1.0f);
        glVertex3f(-1.0f, -1.0f, -1.0f);
        glVertex3f(-1.0f, -1.0f, 1.0f);
        glVertex3f(-1.0f, 1.0f, 1.0f);
        glVertex3f(-1.0f, 1.0f, -1.0f);
    glEnd();
    glPopMatrix();
    
    // رسم ذرات
    renderParticles();
    
    glutSwapBuffers();
}

void reshape(int w, int h) {
    glViewport(0, 0, w, h);
    
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    gluPerspective(45.0f, (float)w/h, 0.1f, 100.0f);
    
    glMatrixMode(GL_MODELVIEW);
}

void update(int value) {
    static clock_t lastTime = clock();
    clock_t currentTime = clock();
    float deltaTime = (currentTime - lastTime) / 1000.0f;
    
    if(currentTime - lastTime >= FRAME_DELAY) {
        // به‌روزرسانی چرخش
        angle += 2.0f;
        if(angle > 360.0f) {
            angle -= 360.0f;
        }
        
        // به‌روزرسانی مقیاس
        if(growing) {
            scale += 0.01f;
            if(scale >= 1.5f) {
                growing = false;
            }
        } else {
            scale -= 0.01f;
            if(scale <= 0.5f) {
                growing = true;
            }
        }
        
        // به‌روزرسانی ذرات
        updateParticles(deltaTime);
        
        lastTime = currentTime;
        glutPostRedisplay();
    }
    
    glutTimerFunc(1, update, 0);
}

void keyboard(unsigned char key, int x, int y) {
    switch(key) {
        case 27:  // ESC
            exit(0);
            break;
    }
}

int main(int argc, char** argv) {
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB | GLUT_DEPTH);
    glutInitWindowSize(800, 600);
    glutCreateWindow("انیمیشن پیشرفته - درس گرافیک کامپیوتری");
    
    glEnable(GL_DEPTH_TEST);
    glClearColor(0.0f, 0.0f, 0.0f, 1.0f);
    
    glutDisplayFunc(display);
    glutReshapeFunc(reshape);
    glutKeyboardFunc(keyboard);
    glutTimerFunc(0, update, 0);
    
    glutMainLoop();
    return 0;
}
```

### 2. بهینه‌سازی انیمیشن
- **LOD (Level of Detail)**: کاهش جزئیات برای اشیاء دور
- **انیمیشن اسکلتی بهینه (Optimized Skeletal Animation)**: استفاده از ماتریس‌های پیش‌محاسبه شده
- **انیمیشن ذره‌ای بهینه (Optimized Particle Systems)**: استفاده از GPU برای محاسبات ذرات
- **انیمیشن فیزیکی بهینه (Optimized Physics)**: استفاده از ساده‌سازی‌های فیزیکی

### 3. انیمیشن در OpenGL مدرن
- **انیمیشن در شیدرها (Animation in Shaders)**: محاسبه انیمیشن در GPU
- **انیمیشن با استفاده از Transform Feedback**: ذخیره نتایج محاسبات در بافر
- **انیمیشن با استفاده از Compute Shaders**: محاسبات موازی برای انیمیشن‌های پیچیده
- **انیمیشن با استفاده از Instancing**: رسم همزمان چندین نمونه از یک شیء با انیمیشن‌های مختلف

## نکات مهم
1. از `glutTimerFunc` برای کنترل فریم‌ریت استفاده کنید
2. همیشه قبل از رسم، ماتریس مدل را ریست کنید
3. برای انیمیشن‌های پیچیده، از متغیرهای سراسری استفاده کنید
4. از `glutPostRedisplay()` برای به‌روزرسانی صحنه استفاده کنید
5. برای انیمیشن‌های روان، از زمان واقعی برای به‌روزرسانی استفاده کنید
6. برای سیستم‌های ذره‌ای، از ترکیب رنگ‌ها (blending) استفاده کنید
7. برای انیمیشن‌های کاراکتر، از ساختار سلسله مراتبی استفاده کنید
8. برای انیمیشن‌های فیزیکی، از روش‌های عددی مناسب برای یکپارچه‌سازی استفاده کنید

## منابع بیشتر
- [مستندات OpenGL](https://www.opengl.org/documentation/)
- [راهنمای انیمیشن در OpenGL](https://www.opengl.org/wiki/Animation)
- [کتاب The Nature of Code](https://natureofcode.com/)
- [مقالات GDC درباره انیمیشن](https://www.gdc.com/)
- [مستندات GLM](https://github.com/g-truc/glm)
- [کتاب Game AI Pro](https://www.gameaipro.com/)
