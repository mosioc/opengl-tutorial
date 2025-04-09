---
title: مدل‌های ۳ بعدی
nav_order: 14
permalink: /threedmodels/
layout: persian
---

# مدل‌های سه بعدی در OpenGL

## مقدمه
مدل‌های سه بعدی یکی از اساسی‌ترین مفاهیم در گرافیک کامپیوتری هستند. این مدل‌ها اساس ساخت دنیاهای مجازی، انیمیشن‌ها، و شبیه‌سازی‌های سه بعدی را تشکیل می‌دهند. در این آموزش، به بررسی مفاهیم پایه‌ای تا پیشرفته مدل‌سازی سه بعدی در OpenGL می‌پردازیم.

## مفاهیم پایه‌ای مدل‌سازی سه بعدی

### 1. ساختار داده‌های هندسی
- **Vertex (رأس)**: نقطه‌ای در فضای سه بعدی با مختصات (x, y, z)
- **Edge (یال)**: خط مستقیم بین دو رأس
- **Face (وجه)**: سطح محدود شده توسط چند یال
- **Mesh (شبکه)**: مجموعه‌ای از رأس‌ها، یال‌ها و وجوه که یک مدل سه بعدی را تشکیل می‌دهند

### 2. انواع مدل‌های سه بعدی
1. **مدل‌های هندسی ساده**
   - اشکال پایه (مکعب، کره، استوانه)
   - سطوح پارامتریک
   - منحنی‌های Bezier و B-spline

2. **مدل‌های پیچیده**
   - مدل‌های پلیگونی
   - مدل‌های Subdivision
   - مدل‌های Procedural

## پیاده‌سازی مدل‌های هندسی ساده

### 1. ساختار پایه برنامه
```cpp
#include <GL/glut.h>
#include <cmath>

// ساختار برای نگهداری اطلاعات رأس
struct Vertex {
    float x, y, z;
    float r, g, b;  // رنگ
    float nx, ny, nz;  // نرمال
    
    Vertex(float x = 0, float y = 0, float z = 0,
           float r = 1, float g = 1, float b = 1,
           float nx = 0, float ny = 1, float nz = 0)
        : x(x), y(y), z(z), r(r), g(g), b(b), nx(nx), ny(ny), nz(nz) {}
};

// کلاس برای مدیریت مدل‌های هندسی
class GeometricModel {
protected:
    std::vector<Vertex> vertices;
    std::vector<unsigned int> indices;
    
public:
    virtual void generate() = 0;  // تابع مجازی برای تولید مدل
    virtual void render() {
        glBegin(GL_TRIANGLES);
        for (size_t i = 0; i < indices.size(); i += 3) {
            for (int j = 0; j < 3; j++) {
                const Vertex& v = vertices[indices[i + j]];
                glColor3f(v.r, v.g, v.b);
                glNormal3f(v.nx, v.ny, v.nz);
                glVertex3f(v.x, v.y, v.z);
            }
        }
        glEnd();
    }
};
```

### 2. پیاده‌سازی مدل‌های پایه

#### مکعب
```cpp
class Cube : public GeometricModel {
public:
    void generate() override {
        // تعریف رأس‌های مکعب
        vertices = {
            // جلو
            Vertex(-0.5f, -0.5f,  0.5f, 1,0,0),  // قرمز
            Vertex( 0.5f, -0.5f,  0.5f, 1,0,0),
            Vertex( 0.5f,  0.5f,  0.5f, 1,0,0),
            Vertex(-0.5f,  0.5f,  0.5f, 1,0,0),
            // پشت
            Vertex(-0.5f, -0.5f, -0.5f, 0,1,0),  // سبز
            Vertex( 0.5f, -0.5f, -0.5f, 0,1,0),
            Vertex( 0.5f,  0.5f, -0.5f, 0,1,0),
            Vertex(-0.5f,  0.5f, -0.5f, 0,1,0)
        };
        
        // تعریف ایندکس‌های وجوه
        indices = {
            // جلو
            0, 1, 2,    0, 2, 3,
            // پشت
            4, 5, 6,    4, 6, 7,
            // بالا
            3, 2, 6,    3, 6, 7,
            // پایین
            0, 1, 5,    0, 5, 4,
            // راست
            1, 2, 6,    1, 6, 5,
            // چپ
            0, 3, 7,    0, 7, 4
        };
    }
};
```

#### کره
```cpp
class Sphere : public GeometricModel {
private:
    int stacks, slices;
    
public:
    Sphere(int stacks = 20, int slices = 20) 
        : stacks(stacks), slices(slices) {}
        
    void generate() override {
        for (int i = 0; i <= stacks; ++i) {
            float phi = M_PI * float(i) / float(stacks);
            for (int j = 0; j <= slices; ++j) {
                float theta = 2.0f * M_PI * float(j) / float(slices);
                
                float x = sin(phi) * cos(theta);
                float y = sin(phi) * sin(theta);
                float z = cos(phi);
                
                vertices.push_back(Vertex(x, y, z, 0,0,1, x,y,z));
            }
        }
        
        // تولید ایندکس‌ها
        for (int i = 0; i < stacks; ++i) {
            for (int j = 0; j < slices; ++j) {
                int first = i * (slices + 1) + j;
                int second = first + slices + 1;
                
                indices.push_back(first);
                indices.push_back(second);
                indices.push_back(first + 1);
                
                indices.push_back(second);
                indices.push_back(second + 1);
                indices.push_back(first + 1);
            }
        }
    }
};
```

## مدل‌های پیچیده و فرمت‌های فایل

### 1. فرمت OBJ
فرمت OBJ یکی از پرکاربردترین فرمت‌ها برای مدل‌های سه بعدی است. این فرمت شامل اطلاعات زیر است:
- موقعیت رأس‌ها (v)
- نرمال‌ها (vn)
- مختصات بافت (vt)
- وجوه (f)

### 2. پیاده‌سازی بارگذاری مدل OBJ
```cpp
class OBJLoader {
private:
    struct OBJVertex {
        int position;
        int normal;
        int texcoord;
    };
    
    std::vector<glm::vec3> positions;
    std::vector<glm::vec3> normals;
    std::vector<glm::vec2> texcoords;
    std::vector<OBJVertex> vertices;
    
public:
    bool loadFromFile(const char* filename) {
        std::ifstream file(filename);
        if (!file.is_open()) return false;
        
        std::string line;
        while (std::getline(file, line)) {
            std::istringstream iss(line);
            std::string type;
            iss >> type;
            
            if (type == "v") {
                glm::vec3 pos;
                iss >> pos.x >> pos.y >> pos.z;
                positions.push_back(pos);
            }
            else if (type == "vn") {
                glm::vec3 normal;
                iss >> normal.x >> normal.y >> normal.z;
                normals.push_back(normal);
            }
            else if (type == "vt") {
                glm::vec2 texcoord;
                iss >> texcoord.x >> texcoord.y;
                texcoords.push_back(texcoord);
            }
            else if (type == "f") {
                // پردازش وجوه
                std::string v1, v2, v3;
                iss >> v1 >> v2 >> v3;
                
                auto processVertex = [](const std::string& v) -> OBJVertex {
                    OBJVertex vertex;
                    std::istringstream iss(v);
                    std::string index;
                    
                    // موقعیت
                    std::getline(iss, index, '/');
                    vertex.position = std::stoi(index) - 1;
                    
                    // مختصات بافت
                    if (std::getline(iss, index, '/')) {
                        if (!index.empty())
                            vertex.texcoord = std::stoi(index) - 1;
                    }
                    
                    // نرمال
                    if (std::getline(iss, index, '/')) {
                        if (!index.empty())
                            vertex.normal = std::stoi(index) - 1;
                    }
                    
                    return vertex;
                };
                
                vertices.push_back(processVertex(v1));
                vertices.push_back(processVertex(v2));
                vertices.push_back(processVertex(v3));
            }
        }
        
        return true;
    }
};
```

## بهینه‌سازی و تکنیک‌های پیشرفته

### 1. Vertex Buffer Objects (VBO)
```cpp
class ModernRenderer {
private:
    GLuint vao, vbo, ebo;
    std::vector<Vertex> vertices;
    std::vector<unsigned int> indices;
    
public:
    void setupBuffers() {
        // ایجاد VAO
        glGenVertexArrays(1, &vao);
        glBindVertexArray(vao);
        
        // ایجاد VBO
        glGenBuffers(1, &vbo);
        glBindBuffer(GL_ARRAY_BUFFER, vbo);
        glBufferData(GL_ARRAY_BUFFER, 
                    vertices.size() * sizeof(Vertex),
                    vertices.data(), 
                    GL_STATIC_DRAW);
        
        // ایجاد EBO
        glGenBuffers(1, &ebo);
        glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, ebo);
        glBufferData(GL_ELEMENT_ARRAY_BUFFER,
                    indices.size() * sizeof(unsigned int),
                    indices.data(),
                    GL_STATIC_DRAW);
        
        // تنظیم vertex attributes
        // موقعیت
        glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex),
                            (void*)offsetof(Vertex, x));
        glEnableVertexAttribArray(0);
        
        // رنگ
        glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex),
                            (void*)offsetof(Vertex, r));
        glEnableVertexAttribArray(1);
        
        // نرمال
        glVertexAttribPointer(2, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex),
                            (void*)offsetof(Vertex, nx));
        glEnableVertexAttribArray(2);
    }
    
    void render() {
        glBindVertexArray(vao);
        glDrawElements(GL_TRIANGLES, indices.size(), GL_UNSIGNED_INT, 0);
    }
};
```

### 2. Level of Detail (LOD)
```cpp
class LODModel {
private:
    std::vector<std::vector<Vertex>> lodLevels;
    std::vector<std::vector<unsigned int>> lodIndices;
    float currentLOD;
    
public:
    void updateLOD(float distance) {
        // محاسبه LOD بر اساس فاصله از دوربین
        currentLOD = std::min(1.0f, distance / 100.0f);
        int level = static_cast<int>(currentLOD * (lodLevels.size() - 1));
        
        // استفاده از مدل مناسب
        renderLODLevel(level);
    }
    
    void renderLODLevel(int level) {
        // رندر مدل با سطح جزئیات مشخص شده
        glBindBuffer(GL_ARRAY_BUFFER, vbo);
        glBufferData(GL_ARRAY_BUFFER,
                    lodLevels[level].size() * sizeof(Vertex),
                    lodLevels[level].data(),
                    GL_STATIC_DRAW);
                    
        glDrawElements(GL_TRIANGLES,
                      lodIndices[level].size(),
                      GL_UNSIGNED_INT,
                      0);
    }
};
```
## نکات مهم و بهترین شیوه‌ها

1. **مدیریت حافظه**
   - استفاده از Smart Pointers برای مدیریت منابع
   - آزادسازی بافرها و بافت‌ها
   - بهینه‌سازی استفاده از حافظه

2. **بهینه‌سازی عملکرد**
   - استفاده از Frustum Culling
   - پیاده‌سازی Occlusion Culling
   - بهینه‌سازی Draw Calls

3. **کیفیت بصری**
   - پیاده‌سازی سایه‌ها
   - اضافه کردن پس‌اثرها (Post-processing)
   - بهبود نورپردازی

## منابع بیشتر و پیشنهادی

1. **کتاب‌ها**
   - "OpenGL Programming Guide" (Red Book)
   - "Real-Time Rendering" by Tomas Akenine-Möller
   - "Computer Graphics: Principles and Practice" by Foley, van Dam, et al.

2. **وبسایت‌ها**
   - [LearnOpenGL](https://learnopengl.com/)
   - [OpenGL Tutorial](https://www.opengl-tutorial.org/)
   - [Khronos Group Documentation](https://www.khronos.org/opengl/)

3. **ابزارها**
   - Blender برای مدل‌سازی
   - Assimp برای بارگذاری مدل
   - GLEW برای مدیریت OpenGL extensions

## سوالات متداول

1. **تفاوت بین مدل‌های هندسی ساده و پیچیده چیست؟**
   - مدل‌های هندسی ساده از اشکال پایه تشکیل شده‌اند
   - مدل‌های پیچیده از هزاران یا میلیون‌ها پلیگون تشکیل شده‌اند

2. **چگونه می‌توانیم عملکرد رندر را بهبود دهیم؟**
   - استفاده از VBO و VAO
   - پیاده‌سازی Culling
   - بهینه‌سازی Draw Calls

3. **فرمت‌های مختلف مدل سه بعدی چه تفاوت‌هایی دارند؟**
   - OBJ: ساده و قابل حمل
   - FBX: پشتیبانی از انیمیشن
   - GLTF: استاندارد جدید برای وب
