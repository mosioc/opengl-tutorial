---
title: بهینه‌سازی
nav_order: 12
permalink: /performance/
layout: persian
---

# بهینه‌سازی عملکرد در OpenGL

## مقدمه
بهینه‌سازی عملکرد یکی از جنبه‌های مهم توسعه برنامه‌های گرافیکی است. در این بخش، با تکنیک‌های مختلف بهینه‌سازی در OpenGL آشنا می‌شویم که به شما کمک می‌کنند برنامه‌های گرافیکی کارآمدتری ایجاد کنید.

## بهینه‌سازی رندرینگ

### 1. Frustum Culling
```cpp
// ساختار برای ذخیره فراستوم
struct Frustum {
    glm::vec4 planes[6]; // 6 صفحه فراستوم
};

// محاسبه فراستوم از ماتریس برجستگی-نما
void calculateFrustum(const glm::mat4& viewProjection, Frustum& frustum) {
    // استخراج صفحات فراستوم از ماتریس برجستگی-نما
    const glm::mat4& m = viewProjection;
    
    // صفحه چپ
    frustum.planes[0] = glm::vec4(
        m[0][3] + m[0][0],
        m[1][3] + m[1][0],
        m[2][3] + m[2][0],
        m[3][3] + m[3][0]
    );
    
    // صفحه راست
    frustum.planes[1] = glm::vec4(
        m[0][3] - m[0][0],
        m[1][3] - m[1][0],
        m[2][3] - m[2][0],
        m[3][3] - m[3][0]
    );
    
    // صفحه پایین
    frustum.planes[2] = glm::vec4(
        m[0][3] + m[0][1],
        m[1][3] + m[1][1],
        m[2][3] + m[2][1],
        m[3][3] + m[3][1]
    );
    
    // صفحه بالا
    frustum.planes[3] = glm::vec4(
        m[0][3] - m[0][1],
        m[1][3] - m[1][0],
        m[2][3] - m[2][1],
        m[3][3] - m[3][1]
    );
    
    // صفحه نزدیک
    frustum.planes[4] = glm::vec4(
        m[0][3] + m[0][2],
        m[1][3] + m[1][2],
        m[2][3] + m[2][2],
        m[3][3] + m[3][2]
    );
    
    // صفحه دور
    frustum.planes[5] = glm::vec4(
        m[0][3] - m[0][2],
        m[1][3] - m[1][2],
        m[2][3] - m[2][2],
        m[3][3] - m[3][2]
    );
    
    // نرمال‌سازی صفحات
    for (int i = 0; i < 6; i++) {
        float length = glm::length(glm::vec3(frustum.planes[i]));
        frustum.planes[i] /= length;
    }
}

// بررسی تقاطع کره با فراستوم
bool sphereInFrustum(const Frustum& frustum, const glm::vec3& center, float radius) {
    for (int i = 0; i < 6; i++) {
        float distance = glm::dot(glm::vec3(frustum.planes[i]), center) + frustum.planes[i].w;
        if (distance < -radius) {
            return false; // کره کاملاً خارج از فراستوم است
        }
    }
    return true; // کره با فراستوم تقاطع دارد
}

// استفاده در حلقه رندرینگ
void renderScene() {
    // محاسبه فراستوم
    Frustum frustum;
    calculateFrustum(viewProjection, frustum);
    
    // رندرینگ فقط اشیاء قابل مشاهده
    for (const auto& object : objects) {
        if (sphereInFrustum(frustum, object.boundingSphere.center, object.boundingSphere.radius)) {
            renderObject(object);
        }
    }
}
```

### 2. Occlusion Culling
```cpp
// ایجاد بافر عمق برای Occlusion Query
GLuint occlusionQuery;
glGenQueries(1, &occlusionQuery);

// بررسی Occlusion
bool isVisible(const BoundingBox& bbox) {
    // شروع Occlusion Query
    glBeginQuery(GL_SAMPLES_PASSED, occlusionQuery);
    
    // رسم جعبه محدود کننده به صورت ساده
    glColorMask(GL_FALSE, GL_FALSE, GL_FALSE, GL_FALSE);
    glDepthMask(GL_FALSE);
    
    renderBoundingBox(bbox);
    
    glDepthMask(GL_TRUE);
    glColorMask(GL_TRUE, GL_TRUE, GL_TRUE, GL_TRUE);
    
    // پایان Occlusion Query
    glEndQuery(GL_SAMPLES_PASSED);
    
    // دریافت نتیجه
    GLuint samplesPassed;
    glGetQueryObjectuiv(occlusionQuery, GL_QUERY_RESULT, &samplesPassed);
    
    // اگر تعداد پیکسل‌های قابل مشاهده صفر باشد، شیء پنهان است
    return samplesPassed > 0;
}

// استفاده در حلقه رندرینگ
void renderScene() {
    for (const auto& object : objects) {
        if (isVisible(object.boundingBox)) {
            renderObject(object);
        }
    }
}
```

### 3. Level of Detail (LOD)
```cpp
// ساختار برای سطوح مختلف جزئیات
struct LODLevel {
    std::vector<float> vertices;
    std::vector<unsigned int> indices;
    float distance; // فاصله‌ای که این سطح جزئیات استفاده می‌شود
};

// محاسبه سطح جزئیات مناسب بر اساس فاصله
int calculateLODLevel(const glm::vec3& objectPos, const glm::vec3& cameraPos) {
    float distance = glm::length(objectPos - cameraPos);
    
    // تعیین سطح جزئیات بر اساس فاصله
    if (distance < 10.0f) return 0;      // جزئیات بالا
    else if (distance < 50.0f) return 1;  // جزئیات متوسط
    else if (distance < 100.0f) return 2; // جزئیات پایین
    else return 3;                        // کمترین جزئیات
}

// استفاده در حلقه رندرینگ
void renderObject(const Object& object) {
    int lodLevel = calculateLODLevel(object.position, cameraPosition);
    const LODLevel& lod = object.lodLevels[lodLevel];
    
    // رندرینگ با سطح جزئیات مناسب
    glBindBuffer(GL_ARRAY_BUFFER, object.vbo);
    glBufferData(GL_ARRAY_BUFFER, lod.vertices.size() * sizeof(float), lod.vertices.data(), GL_STATIC_DRAW);
    
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, object.ebo);
    glBufferData(GL_ELEMENT_ARRAY_BUFFER, lod.indices.size() * sizeof(unsigned int), lod.indices.data(), GL_STATIC_DRAW);
    
    glDrawElements(GL_TRIANGLES, lod.indices.size(), GL_UNSIGNED_INT, 0);
}
```

## بهینه‌سازی حافظه

### 1. مدیریت بافرها
```cpp
// ایجاد و مدیریت بافرها به صورت کارآمد
class BufferManager {
private:
    struct Buffer {
        GLuint id;
        size_t size;
        bool inUse;
    };
    
    std::vector<Buffer> buffers;
    
public:
    // تخصیص بافر با اندازه مشخص
    GLuint allocateBuffer(size_t size) {
        // جستجوی بافر آزاد با اندازه مناسب
        for (auto& buffer : buffers) {
            if (!buffer.inUse && buffer.size >= size) {
                buffer.inUse = true;
                return buffer.id;
            }
        }
        
        // ایجاد بافر جدید
        GLuint newBuffer;
        glGenBuffers(1, &newBuffer);
        glBindBuffer(GL_ARRAY_BUFFER, newBuffer);
        glBufferData(GL_ARRAY_BUFFER, size, nullptr, GL_DYNAMIC_DRAW);
        
        buffers.push_back({newBuffer, size, true});
        return newBuffer;
    }
    
    // آزادسازی بافر
    void freeBuffer(GLuint bufferId) {
        for (auto& buffer : buffers) {
            if (buffer.id == bufferId) {
                buffer.inUse = false;
                break;
            }
        }
    }
    
    // پاکسازی همه بافرها
    void cleanup() {
        for (const auto& buffer : buffers) {
            glDeleteBuffers(1, &buffer.id);
        }
        buffers.clear();
    }
};
```

### 2. بهینه‌سازی بافت‌ها
```cpp
// مدیریت بافت‌ها به صورت کارآمد
class TextureManager {
private:
    struct Texture {
        GLuint id;
        int width, height;
        GLenum format;
        bool inUse;
    };
    
    std::vector<Texture> textures;
    
public:
    // بارگذاری بافت با بهینه‌سازی
    GLuint loadTexture(const std::string& path) {
        // بررسی بافت‌های موجود
        for (auto& texture : textures) {
            if (!texture.inUse) {
                // استفاده مجدد از بافت موجود
                texture.inUse = true;
                return texture.id;
            }
        }
        
        // بارگذاری تصویر
        int width, height, nrChannels;
        unsigned char* data = stbi_load(path.c_str(), &width, &height, &nrChannels, 0);
        
        if (!data) {
            std::cerr << "Failed to load texture: " << path << std::endl;
            return 0;
        }
        
        // تعیین فرمت مناسب
        GLenum format;
        if (nrChannels == 1) format = GL_RED;
        else if (nrChannels == 3) format = GL_RGB;
        else if (nrChannels == 4) format = GL_RGBA;
        else {
            std::cerr << "Unsupported number of channels: " << nrChannels << std::endl;
            stbi_image_free(data);
            return 0;
        }
        
        // ایجاد بافت
        GLuint textureId;
        glGenTextures(1, &textureId);
        glBindTexture(GL_TEXTURE_2D, textureId);
        
        // تنظیم پارامترهای بافت
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
        
        // بارگذاری داده‌های بافت
        glTexImage2D(GL_TEXTURE_2D, 0, format, width, height, 0, format, GL_UNSIGNED_BYTE, data);
        glGenerateMipmap(GL_TEXTURE_2D);
        
        // آزادسازی داده‌های تصویر
        stbi_image_free(data);
        
        // ذخیره اطلاعات بافت
        textures.push_back({textureId, width, height, format, true});
        
        return textureId;
    }
    
    // آزادسازی بافت
    void freeTexture(GLuint textureId) {
        for (auto& texture : textures) {
            if (texture.id == textureId) {
                texture.inUse = false;
                break;
            }
        }
    }
    
    // پاکسازی همه بافت‌ها
    void cleanup() {
        for (const auto& texture : textures) {
            glDeleteTextures(1, &texture.id);
        }
        textures.clear();
    }
};
```

### 3. بهینه‌سازی شیدرها
```cpp
// مدیریت شیدرها به صورت کارآمد
class ShaderManager {
private:
    struct ShaderProgram {
        GLuint id;
        std::string vertexPath;
        std::string fragmentPath;
        bool inUse;
    };
    
    std::vector<ShaderProgram> shaders;
    
public:
    // کامپایل و لینک شیدرها
    GLuint compileShader(const std::string& vertexPath, const std::string& fragmentPath) {
        // بررسی شیدرهای موجود
        for (auto& shader : shaders) {
            if (shader.vertexPath == vertexPath && shader.fragmentPath == fragmentPath) {
                shader.inUse = true;
                return shader.id;
            }
        }
        
        // خواندن کد شیدرها
        std::string vertexCode = readFile(vertexPath);
        std::string fragmentCode = readFile(fragmentPath);
        
        // کامپایل شیدر رأس
        GLuint vertexShader = glCreateShader(GL_VERTEX_SHADER);
        const char* vertexSource = vertexCode.c_str();
        glShaderSource(vertexShader, 1, &vertexSource, NULL);
        glCompileShader(vertexShader);
        
        // بررسی خطاهای کامپایل شیدر رأس
        checkShaderCompileErrors(vertexShader, "VERTEX");
        
        // کامپایل شیدر فرگمنت
        GLuint fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
        const char* fragmentSource = fragmentCode.c_str();
        glShaderSource(fragmentShader, 1, &fragmentSource, NULL);
        glCompileShader(fragmentShader);
        
        // بررسی خطاهای کامپایل شیدر فرگمنت
        checkShaderCompileErrors(fragmentShader, "FRAGMENT");
        
        // لینک شیدرها
        GLuint shaderProgram = glCreateProgram();
        glAttachShader(shaderProgram, vertexShader);
        glAttachShader(shaderProgram, fragmentShader);
        glLinkProgram(shaderProgram);
        
        // بررسی خطاهای لینک
        checkShaderLinkErrors(shaderProgram);
        
        // پاکسازی شیدرهای جداگانه
        glDeleteShader(vertexShader);
        glDeleteShader(fragmentShader);
        
        // ذخیره اطلاعات شیدر
        shaders.push_back({shaderProgram, vertexPath, fragmentPath, true});
        
        return shaderProgram;
    }
    
    // استفاده از شیدر
    void useShader(GLuint shaderId) {
        glUseProgram(shaderId);
    }
    
    // آزادسازی شیدر
    void freeShader(GLuint shaderId) {
        for (auto& shader : shaders) {
            if (shader.id == shaderId) {
                shader.inUse = false;
                break;
            }
        }
    }
    
    // پاکسازی همه شیدرها
    void cleanup() {
        for (const auto& shader : shaders) {
            glDeleteProgram(shader.id);
        }
        shaders.clear();
    }
    
private:
    // خواندن فایل
    std::string readFile(const std::string& path) {
        std::ifstream file(path);
        if (!file.is_open()) {
            std::cerr << "Failed to open file: " << path << std::endl;
            return "";
        }
        
        std::stringstream buffer;
        buffer << file.rdbuf();
        return buffer.str();
    }
    
    // بررسی خطاهای کامپایل شیدر
    void checkShaderCompileErrors(GLuint shader, const std::string& type) {
        GLint success;
        GLchar infoLog[1024];
        glGetShaderiv(shader, GL_COMPILE_STATUS, &success);
        
        if (!success) {
            glGetShaderInfoLog(shader, 1024, NULL, infoLog);
            std::cerr << "ERROR::SHADER_COMPILATION_ERROR of type: " << type << "\n" << infoLog << std::endl;
        }
    }
    
    // بررسی خطاهای لینک شیدر
    void checkShaderLinkErrors(GLuint program) {
        GLint success;
        GLchar infoLog[1024];
        glGetProgramiv(program, GL_LINK_STATUS, &success);
        
        if (!success) {
            glGetProgramInfoLog(program, 1024, NULL, infoLog);
            std::cerr << "ERROR::PROGRAM_LINKING_ERROR\n" << infoLog << std::endl;
        }
    }
};
```

## بهینه‌سازی CPU

### 1. چند نخی (Multithreading)
```cpp
// مدیریت رندرینگ در چند نخ
class RenderThread {
private:
    std::thread thread;
    std::queue<RenderTask> taskQueue;
    std::mutex queueMutex;
    std::condition_variable condition;
    bool running;
    
public:
    RenderThread() : running(true) {
        thread = std::thread(&RenderThread::processTasks, this);
    }
    
    ~RenderThread() {
        {
            std::lock_guard<std::mutex> lock(queueMutex);
            running = false;
        }
        condition.notify_one();
        thread.join();
    }
    
    // اضافه کردن وظیفه به صف
    void addTask(const RenderTask& task) {
        {
            std::lock_guard<std::mutex> lock(queueMutex);
            taskQueue.push(task);
        }
        condition.notify_one();
    }
    
private:
    // پردازش وظایف در نخ جداگانه
    void processTasks() {
        while (true) {
            RenderTask task;
            
            {
                std::unique_lock<std::mutex> lock(queueMutex);
                condition.wait(lock, [this] { return !running || !taskQueue.empty(); });
                
                if (!running && taskQueue.empty()) {
                    return;
                }
                
                task = taskQueue.front();
                taskQueue.pop();
            }
            
            // پردازش وظیفه
            processRenderTask(task);
        }
    }
    
    // پردازش وظیفه رندرینگ
    void processRenderTask(const RenderTask& task) {
        // پردازش داده‌های رندرینگ در نخ جداگانه
        // ...
        
        // ارسال نتیجه به نخ اصلی برای رندرینگ نهایی
        // ...
    }
};
```

### 2. بهینه‌سازی حلقه رندرینگ
```cpp
// بهینه‌سازی حلقه رندرینگ
void optimizedRenderLoop() {
    // مقداردهی اولیه
    double lastTime = glfwGetTime();
    double deltaTime = 0.0;
    
    // حلقه رندرینگ
    while (!glfwWindowShouldClose(window)) {
        // محاسبه زمان سپری شده
        double currentTime = glfwGetTime();
        deltaTime = currentTime - lastTime;
        lastTime = currentTime;
        
        // به‌روزرسانی وضعیت بازی
        updateGameState(deltaTime);
        
        // به‌روزرسانی دوربین
        updateCamera(deltaTime);
        
        // محاسبه ماتریس‌های تبدیل
        glm::mat4 view = camera.getViewMatrix();
        glm::mat4 projection = glm::perspective(glm::radians(45.0f), (float)width / (float)height, 0.1f, 100.0f);
        glm::mat4 viewProjection = projection * view;
        
        // محاسبه فراستوم
        Frustum frustum;
        calculateFrustum(viewProjection, frustum);
        
        // پاک کردن بافرها
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
        
        // رندرینگ اشیاء قابل مشاهده
        for (const auto& object : objects) {
            // بررسی دیدپذیری با Frustum Culling
            if (!sphereInFrustum(frustum, object.boundingSphere.center, object.boundingSphere.radius)) {
                continue;
            }
            
            // بررسی دیدپذیری با Occlusion Culling
            if (!isVisible(object.boundingBox)) {
                continue;
            }
            
            // محاسبه سطح جزئیات مناسب
            int lodLevel = calculateLODLevel(object.position, camera.getPosition());
            
            // رندرینگ شیء
            renderObject(object, lodLevel);
        }
        
        // رندرینگ رابط کاربری
        renderUI();
        
        // جابجا کردن بافرها و بررسی رویدادها
        glfwSwapBuffers(window);
        glfwPollEvents();
    }
}
```

### 3. بهینه‌سازی ساختار داده‌ها
```cpp
// ساختار داده بهینه برای اشیاء
struct GameObject {
    glm::vec3 position;
    glm::vec3 scale;
    glm::quat rotation;
    BoundingSphere boundingSphere;
    BoundingBox boundingBox;
    std::vector<LODLevel> lodLevels;
    GLuint vao, vbo, ebo;
    GLuint shader;
    std::vector<GLuint> textures;
};

// ساختار داده بهینه برای مدیریت اشیاء
class GameObjectManager {
private:
    // استفاده از ساختار داده‌های کارآمد
    std::vector<GameObject> objects;
    std::unordered_map<std::string, GLuint> shaderCache;
    std::unordered_map<std::string, GLuint> textureCache;
    
    // تقسیم‌بندی فضایی برای جستجوی سریع‌تر
    struct SpatialGrid {
        std::vector<std::vector<int>> grid;
        float cellSize;
        glm::vec3 minBound, maxBound;
        
        SpatialGrid(float cellSize, const glm::vec3& minBound, const glm::vec3& maxBound)
            : cellSize(cellSize), minBound(minBound), maxBound(maxBound) {
            int xCells = static_cast<int>((maxBound.x - minBound.x) / cellSize) + 1;
            int yCells = static_cast<int>((maxBound.y - minBound.y) / cellSize) + 1;
            int zCells = static_cast<int>((maxBound.z - minBound.z) / cellSize) + 1;
            
            grid.resize(xCells * yCells * zCells);
        }
        
        // اضافه کردن شیء به گرید
        void addObject(int objectIndex, const glm::vec3& position) {
            int x = static_cast<int>((position.x - minBound.x) / cellSize);
            int y = static_cast<int>((position.y - minBound.y) / cellSize);
            int z = static_cast<int>((position.z - minBound.z) / cellSize);
            
            int index = x + y * static_cast<int>((maxBound.x - minBound.x) / cellSize) + 
                        z * static_cast<int>((maxBound.x - minBound.x) / cellSize) * 
                        static_cast<int>((maxBound.y - minBound.y) / cellSize);
            
            if (index >= 0 && index < grid.size()) {
                grid[index].push_back(objectIndex);
            }
        }
        
        // یافتن اشیاء در یک محدوده
        std::vector<int> getObjectsInRange(const glm::vec3& center, float radius) {
            std::vector<int> result;
            
            int minX = static_cast<int>((center.x - radius - minBound.x) / cellSize);
            int maxX = static_cast<int>((center.x + radius - minBound.x) / cellSize);
            int minY = static_cast<int>((center.y - radius - minBound.y) / cellSize);
            int maxY = static_cast<int>((center.y + radius - minBound.y) / cellSize);
            int minZ = static_cast<int>((center.z - radius - minBound.z) / cellSize);
            int maxZ = static_cast<int>((center.z + radius - minBound.z) / cellSize);
            
            for (int x = minX; x <= maxX; x++) {
                for (int y = minY; y <= maxY; y++) {
                    for (int z = minZ; z <= maxZ; z++) {
                        int index = x + y * static_cast<int>((maxBound.x - minBound.x) / cellSize) + 
                                    z * static_cast<int>((maxBound.x - minBound.x) / cellSize) * 
                                    static_cast<int>((maxBound.y - minBound.y) / cellSize);
                        
                        if (index >= 0 && index < grid.size()) {
                            result.insert(result.end(), grid[index].begin(), grid[index].end());
                        }
                    }
                }
            }
            
            return result;
        }
    };
    
    SpatialGrid spatialGrid;
    
public:
    GameObjectManager() : spatialGrid(10.0f, glm::vec3(-1000.0f), glm::vec3(1000.0f)) {}
    
    // اضافه کردن شیء جدید
    void addObject(const GameObject& object) {
        int index = objects.size();
        objects.push_back(object);
        spatialGrid.addObject(index, object.position);
    }
    
    // یافتن اشیاء در محدوده مشخص
    std::vector<GameObject*> getObjectsInRange(const glm::vec3& center, float radius) {
        std::vector<int> indices = spatialGrid.getObjectsInRange(center, radius);
        std::vector<GameObject*> result;
        
        for (int index : indices) {
            result.push_back(&objects[index]);
        }
        
        return result;
    }
    
    // رندرینگ اشیاء
    void renderObjects(const Frustum& frustum, const glm::vec3& cameraPos) {
        for (auto& object : objects) {
            // بررسی دیدپذیری با Frustum Culling
            if (!sphereInFrustum(frustum, object.boundingSphere.center, object.boundingSphere.radius)) {
                continue;
            }
            
            // محاسبه سطح جزئیات مناسب
            int lodLevel = calculateLODLevel(object.position, cameraPos);
            
            // رندرینگ شیء
            renderObject(object, lodLevel);
        }
    }
};
```

## بهینه‌سازی GPU

### 1. بهینه‌سازی شیدرها
```glsl
// شیدر رأس بهینه‌سازی شده
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aNormal;
layout (location = 2) in vec2 aTexCoord;

out vec3 FragPos;
out vec3 Normal;
out vec2 TexCoord;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

void main()
{
    // محاسبه موقعیت نهایی
    gl_Position = projection * view * model * vec4(aPos, 1.0);
    
    // محاسبه موقعیت در فضای جهان
    FragPos = vec3(model * vec4(aPos, 1.0));
    
    // محاسبه نرمال در فضای جهان
    Normal = mat3(transpose(inverse(model))) * aNormal;
    
    // انتقال مختصات بافت
    TexCoord = aTexCoord;
}
```

### 2. بهینه‌سازی بافرها
```cpp
// بهینه‌سازی بافرها
void optimizeBuffers() {
    // استفاده از بافرهای ثابت برای داده‌های استاتیک
    glBindBuffer(GL_ARRAY_BUFFER, staticVBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(staticVertices), staticVertices, GL_STATIC_DRAW);
    
    // استفاده از بافرهای پویا برای داده‌های متغیر
    glBindBuffer(GL_ARRAY_BUFFER, dynamicVBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(dynamicVertices), dynamicVertices, GL_DYNAMIC_DRAW);
    
    // استفاده از بافرهای استریم برای داده‌های با تغییرات مکرر
    glBindBuffer(GL_ARRAY_BUFFER, streamVBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(streamVertices), streamVertices, GL_STREAM_DRAW);
    
    // استفاده از بافرهای نگاشت شده برای دسترسی مستقیم به داده‌ها
    glBindBuffer(GL_ARRAY_BUFFER, mappedVBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(mappedVertices), NULL, GL_DYNAMIC_DRAW);
    void* data = glMapBuffer(GL_ARRAY_BUFFER, GL_WRITE_ONLY);
    memcpy(data, mappedVertices, sizeof(mappedVertices));
    glUnmapBuffer(GL_ARRAY_BUFFER);
}
```

### 3. بهینه‌سازی حالت‌های رندرینگ
```cpp
// بهینه‌سازی حالت‌های رندرینگ
void optimizeRenderStates() {
    // فعال کردن تست عمق
    glEnable(GL_DEPTH_TEST);
    
    // فعال کردن Face Culling
    glEnable(GL_CULL_FACE);
    glCullFace(GL_BACK);
    
    // فعال کردن Scissor Test برای محدود کردن ناحیه رندرینگ
    glEnable(GL_SCISSOR_TEST);
    glScissor(0, 0, width, height);
    
    // فعال کردن Stencil Test
    glEnable(GL_STENCIL_TEST);
    glStencilFunc(GL_EQUAL, 1, 0xFF);
    glStencilOp(GL_KEEP, GL_KEEP, GL_REPLACE);
    
    // فعال کردن Blending
    glEnable(GL_BLEND);
    glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
    
    // فعال کردن Polygon Offset برای جلوگیری از Z-fighting
    glEnable(GL_POLYGON_OFFSET_FILL);
    glPolygonOffset(1.0f, 1.0f);
}
```

## ابزارهای پروفایل و اشکال‌زدایی

### 1. استفاده از OpenGL Debug Output
```cpp
// فعال کردن Debug Output
void enableDebugOutput() {
    glEnable(GL_DEBUG_OUTPUT);
    glEnable(GL_DEBUG_OUTPUT_SYNCHRONOUS);
    glDebugMessageCallback(debugCallback, nullptr);
}

// تابع callback برای دریافت پیام‌های خطا
void GLAPIENTRY debugCallback(GLenum source, GLenum type, GLuint id, GLenum severity, GLsizei length, const GLchar* message, const void* userParam) {
    // نادیده گرفتن خطاهای غیرمهم
    if (severity == GL_DEBUG_SEVERITY_NOTIFICATION) return;
    
    // چاپ پیام خطا
    std::cerr << "OpenGL Debug: " << message << std::endl;
    
    // بررسی نوع خطا
    switch (type) {
        case GL_DEBUG_TYPE_ERROR:
            std::cerr << "Error: ";
            break;
        case GL_DEBUG_TYPE_DEPRECATED_BEHAVIOR:
            std::cerr << "Deprecated Behavior: ";
            break;
        case GL_DEBUG_TYPE_UNDEFINED_BEHAVIOR:
            std::cerr << "Undefined Behavior: ";
            break;
        case GL_DEBUG_TYPE_PORTABILITY:
            std::cerr << "Portability: ";
            break;
        case GL_DEBUG_TYPE_PERFORMANCE:
            std::cerr << "Performance: ";
            break;
        case GL_DEBUG_TYPE_MARKER:
            std::cerr << "Marker: ";
            break;
        case GL_DEBUG_TYPE_PUSH_GROUP:
            std::cerr << "Push Group: ";
            break;
        case GL_DEBUG_TYPE_POP_GROUP:
            std::cerr << "Pop Group: ";
            break;
        default:
            std::cerr << "Other: ";
            break;
    }
    
    // بررسی منبع خطا
    switch (source) {
        case GL_DEBUG_SOURCE_API:
            std::cerr << "API";
            break;
        case GL_DEBUG_SOURCE_WINDOW_SYSTEM:
            std::cerr << "Window System";
            break;
        case GL_DEBUG_SOURCE_SHADER_COMPILER:
            std::cerr << "Shader Compiler";
            break;
        case GL_DEBUG_SOURCE_THIRD_PARTY:
            std::cerr << "Third Party";
            break;
        case GL_DEBUG_SOURCE_APPLICATION:
            std::cerr << "Application";
            break;
        case GL_DEBUG_SOURCE_OTHER:
            std::cerr << "Other";
            break;
        default:
            std::cerr << "Unknown";
            break;
    }
    
    std::cerr << std::endl;
}
```

### 2. استفاده از Query Objects برای اندازه‌گیری عملکرد
```cpp
// اندازه‌گیری زمان رندرینگ
void measureRenderTime() {
    // ایجاد Query Objects
    GLuint timeQuery;
    glGenQueries(1, &timeQuery);
    
    // شروع اندازه‌گیری
    glBeginQuery(GL_TIME_ELAPSED, timeQuery);
    
    // رندرینگ صحنه
    renderScene();
    
    // پایان اندازه‌گیری
    glEndQuery(GL_TIME_ELAPSED);
    
    // دریافت نتیجه
    GLuint64 elapsedTime;
    glGetQueryObjectui64v(timeQuery, GL_QUERY_RESULT, &elapsedTime);
    
    // تبدیل به میلی‌ثانیه
    double renderTimeMs = elapsedTime / 1000000.0;
    
    // چاپ نتیجه
    std::cout << "Render time: " << renderTimeMs << " ms" << std::endl;
    
    // پاکسازی
    glDeleteQueries(1, &timeQuery);
}

// اندازه‌گیری تعداد پیکسل‌های رندر شده
void measurePixelsDrawn() {
    // ایجاد Query Objects
    GLuint samplesQuery;
    glGenQueries(1, &samplesQuery);
    
    // شروع اندازه‌گیری
    glBeginQuery(GL_SAMPLES_PASSED, samplesQuery);
    
    // رندرینگ صحنه
    renderScene();
    
    // پایان اندازه‌گیری
    glEndQuery(GL_SAMPLES_PASSED);
    
    // دریافت نتیجه
    GLuint samplesPassed;
    glGetQueryObjectuiv(samplesQuery, GL_QUERY_RESULT, &samplesPassed);
    
    // چاپ نتیجه
    std::cout << "Pixels drawn: " << samplesPassed << std::endl;
    
    // پاکسازی
    glDeleteQueries(1, &samplesQuery);
}
```

### 3. استفاده از Timer Queries برای پروفایل کردن بخش‌های مختلف
```cpp
// پروفایل کردن بخش‌های مختلف رندرینگ
void profileRendering() {
    // ایجاد Query Objects
    GLuint queries[4];
    glGenQueries(4, queries);
    
    // اندازه‌گیری زمان Frustum Culling
    glBeginQuery(GL_TIME_ELAPSED, queries[0]);
    performFrustumCulling();
    glEndQuery(GL_TIME_ELAPSED);
    
    // اندازه‌گیری زمان Occlusion Culling
    glBeginQuery(GL_TIME_ELAPSED, queries[1]);
    performOcclusionCulling();
    glEndQuery(GL_TIME_ELAPSED);
    
    // اندازه‌گیری زمان رندرینگ اشیاء
    glBeginQuery(GL_TIME_ELAPSED, queries[2]);
    renderObjects();
    glEndQuery(GL_TIME_ELAPSED);
    
    // اندازه‌گیری زمان پس‌پردازش
    glBeginQuery(GL_TIME_ELAPSED, queries[3]);
    performPostProcessing();
    glEndQuery(GL_TIME_ELAPSED);
    
    // دریافت نتایج
    GLuint64 times[4];
    for (int i = 0; i < 4; i++) {
        glGetQueryObjectui64v(queries[i], GL_QUERY_RESULT, &times[i]);
    }
    
    // چاپ نتایج
    std::cout << "Frustum Culling: " << times[0] / 1000000.0 << " ms" << std::endl;
    std::cout << "Occlusion Culling: " << times[1] / 1000000.0 << " ms" << std::endl;
    std::cout << "Object Rendering: " << times[2] / 1000000.0 << " ms" << std::endl;
    std::cout << "Post-Processing: " << times[3] / 1000000.0 << " ms" << std::endl;
    
    // پاکسازی
    glDeleteQueries(4, queries);
}
```
## نکات مهم
1. همیشه از Frustum Culling برای حذف اشیاء خارج از دید استفاده کنید
2. برای صحنه‌های پیچیده، از Occlusion Culling استفاده کنید
3. برای مدل‌های با جزئیات بالا، از سیستم LOD استفاده کنید
4. از ساختار داده‌های کارآمد برای مدیریت اشیاء استفاده کنید
5. از ابزارهای پروفایل برای شناسایی و رفع مشکلات عملکرد استفاده کنید

## منابع بیشتر
- [مستندات OpenGL](https://www.opengl.org/documentation/)
- [راهنمای بهینه‌سازی OpenGL](https://www.opengl.org/wiki/Performance)
- [راهنمای پروفایل OpenGL](https://www.opengl.org/wiki/Debug_Output) 
