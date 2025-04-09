---
title: شیدرها
nav_order: 9
permalink: /shaders/
layout: persian
---

 # شیدرها در OpenGL

## مقدمه
شیدرها برنامه‌های کوچکی هستند که روی GPU اجرا می‌شوند و نحوه پردازش رئوس و پیکسل‌ها را در OpenGL کنترل می‌کنند. در این بخش، با مفاهیم پایه شیدرها و نحوه استفاده از آنها در OpenGL آشنا می‌شویم.

## انواع شیدر

### 1. شیدر رأس (Vertex Shader)
شیدر رأس برای هر رأس در مدل سه‌بعدی اجرا می‌شود و وظیفه تبدیل مختصات رئوس را بر عهده دارد.

```glsl
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aColor;

out vec3 vertexColor;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

void main()
{
    gl_Position = projection * view * model * vec4(aPos, 1.0);
    vertexColor = aColor;
}
```

### 2. شیدر فرگمنت (Fragment Shader)
شیدر فرگمنت برای هر پیکسل قابل مشاهده اجرا می‌شود و رنگ نهایی پیکسل را تعیین می‌کند.

```glsl
#version 330 core
in vec3 vertexColor;

out vec4 FragColor;

void main()
{
    FragColor = vec4(vertexColor, 1.0);
}
```

### 3. شیدر هندسی (Geometry Shader)
شیدر هندسی بین شیدر رأس و شیدر فرگمنت اجرا می‌شود و می‌تواند هندسه را تغییر دهد.

```glsl
#version 330 core
layout (triangles) in;
layout (triangle_strip, max_vertices = 3) out;

in vec3 vertexColor[];
out vec3 fragColor;

void main()
{
    for(int i = 0; i < 3; i++)
    {
        gl_Position = gl_in[i].gl_Position;
        fragColor = vertexColor[i];
        EmitVertex();
    }
    EndPrimitive();
}
```

## کامپایل و لینک شیدرها

### 1. خواندن کد شیدر از فایل
```cpp
std::string readShaderFile(const char* filePath) {
    std::string content;
    std::ifstream fileStream(filePath, std::ios::in);
    
    if(!fileStream.is_open()) {
        std::cerr << "Could not read file " << filePath << ". File does not exist." << std::endl;
        return "";
    }
    
    std::string line = "";
    while(!fileStream.eof()) {
        std::getline(fileStream, line);
        content.append(line + "\n");
    }
    
    fileStream.close();
    return content;
}
```

### 2. کامپایل شیدر
```cpp
GLuint compileShader(const char* source, GLenum type) {
    GLuint shader = glCreateShader(type);
    glShaderSource(shader, 1, &source, NULL);
    glCompileShader(shader);
    
    // بررسی خطاهای کامپایل
    GLint success;
    GLchar infoLog[512];
    glGetShaderiv(shader, GL_COMPILE_STATUS, &success);
    if(!success) {
        glGetShaderInfoLog(shader, 512, NULL, infoLog);
        std::cerr << "ERROR::SHADER::COMPILATION_FAILED\n" << infoLog << std::endl;
        return 0;
    }
    
    return shader;
}
```

### 3. لینک شیدرها
```cpp
GLuint createShaderProgram(const char* vertexPath, const char* fragmentPath) {
    // خواندن کد شیدرها
    std::string vertexCode = readShaderFile(vertexPath);
    std::string fragmentCode = readShaderFile(fragmentPath);
    
    // کامپایل شیدرها
    GLuint vertexShader = compileShader(vertexCode.c_str(), GL_VERTEX_SHADER);
    GLuint fragmentShader = compileShader(fragmentCode.c_str(), GL_FRAGMENT_SHADER);
    
    // ایجاد برنامه شیدر
    GLuint shaderProgram = glCreateProgram();
    glAttachShader(shaderProgram, vertexShader);
    glAttachShader(shaderProgram, fragmentShader);
    glLinkProgram(shaderProgram);
    
    // بررسی خطاهای لینک
    GLint success;
    GLchar infoLog[512];
    glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
    if(!success) {
        glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog);
        std::cerr << "ERROR::SHADER::PROGRAM::LINKING_FAILED\n" << infoLog << std::endl;
        return 0;
    }
    
    // پاکسازی شیدرها
    glDeleteShader(vertexShader);
    glDeleteShader(fragmentShader);
    
    return shaderProgram;
}
```

## متغیرها در شیدرها

### 1. متغیرهای ورودی (Input Variables)
```glsl
// شیدر رأس
layout (location = 0) in vec3 aPos;  // موقعیت رأس
layout (location = 1) in vec3 aColor;  // رنگ رأس
layout (location = 2) in vec2 aTexCoord;  // مختصات بافت
```

### 2. متغیرهای خروجی (Output Variables)
```glsl
// شیدر رأس
out vec3 vertexColor;
out vec2 TexCoord;

// شیدر فرگمنت
out vec4 FragColor;
```

### 3. متغیرهای یکنواخت (Uniform Variables)
```glsl
// تعریف در شیدر
uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;
uniform vec3 lightPos;
uniform float time;

// تنظیم در کد C++
glUseProgram(shaderProgram);
glUniformMatrix4fv(glGetUniformLocation(shaderProgram, "model"), 1, GL_FALSE, glm::value_ptr(model));
glUniformMatrix4fv(glGetUniformLocation(shaderProgram, "view"), 1, GL_FALSE, glm::value_ptr(view));
glUniformMatrix4fv(glGetUniformLocation(shaderProgram, "projection"), 1, GL_FALSE, glm::value_ptr(projection));
glUniform3fv(glGetUniformLocation(shaderProgram, "lightPos"), 1, &lightPos[0]);
glUniform1f(glGetUniformLocation(shaderProgram, "time"), (float)glfwGetTime());
```

## مثال کامل: استفاده از شیدرها

### 1. فایل شیدر رأس (vertex_shader.glsl)
```glsl
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aColor;

out vec3 vertexColor;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;
uniform float time;

void main()
{
    // تبدیل موقعیت رأس
    vec3 pos = aPos;
    pos.y += sin(time + pos.x) * 0.1;  // اضافه کردن حرکت موجی
    
    gl_Position = projection * view * model * vec4(pos, 1.0);
    vertexColor = aColor;
}
```

### 2. فایل شیدر فرگمنت (fragment_shader.glsl)
```glsl
#version 330 core
in vec3 vertexColor;

out vec4 FragColor;

uniform float time;

void main()
{
    // تغییر رنگ با زمان
    vec3 color = vertexColor;
    color.r = (sin(time) + 1.0) * 0.5;
    
    FragColor = vec4(color, 1.0);
}
```

### 3. برنامه اصلی
```cpp
#include <GL/glew.h>
#include <GLFW/glfw3.h>
#include <glm/glm.hpp>
#include <glm/gtc/matrix_transform.hpp>
#include <glm/gtc/type_ptr.hpp>
#include <iostream>
#include <fstream>
#include <string>

// تعریف توابع کمکی
std::string readShaderFile(const char* filePath);
GLuint compileShader(const char* source, GLenum type);
GLuint createShaderProgram(const char* vertexPath, const char* fragmentPath);

// متغیرهای سراسری
GLuint VBO, VAO;
GLuint shaderProgram;

void setupBuffers() {
    // تعریف رئوس یک مثلث
    float vertices[] = {
        // موقعیت            // رنگ
        -0.5f, -0.5f, 0.0f,  1.0f, 0.0f, 0.0f,  // پایین چپ
         0.5f, -0.5f, 0.0f,  0.0f, 1.0f, 0.0f,  // پایین راست
         0.0f,  0.5f, 0.0f,  0.0f, 0.0f, 1.0f   // بالا
    };
    
    // ایجاد و بایند کردن VAO و VBO
    glGenVertexArrays(1, &VAO);
    glGenBuffers(1, &VBO);
    
    glBindVertexArray(VAO);
    
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    
    // تنظیم ویژگی‌های رأس
    // موقعیت
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)0);
    glEnableVertexAttribArray(0);
    
    // رنگ
    glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)(3 * sizeof(float)));
    glEnableVertexAttribArray(1);
    
    glBindBuffer(GL_ARRAY_BUFFER, 0);
    glBindVertexArray(0);
}

void render() {
    // پاک کردن صفحه
    glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
    glClear(GL_COLOR_BUFFER_BIT);
    
    // استفاده از برنامه شیدر
    glUseProgram(shaderProgram);
    
    // تنظیم متغیرهای یکنواخت
    float time = glfwGetTime();
    glUniform1f(glGetUniformLocation(shaderProgram, "time"), time);
    
    // تنظیم ماتریس‌های تبدیل
    glm::mat4 model = glm::mat4(1.0f);
    glm::mat4 view = glm::mat4(1.0f);
    glm::mat4 projection = glm::mat4(1.0f);
    
    glUniformMatrix4fv(glGetUniformLocation(shaderProgram, "model"), 1, GL_FALSE, glm::value_ptr(model));
    glUniformMatrix4fv(glGetUniformLocation(shaderProgram, "view"), 1, GL_FALSE, glm::value_ptr(view));
    glUniformMatrix4fv(glGetUniformLocation(shaderProgram, "projection"), 1, GL_FALSE, glm::value_ptr(projection));
    
    // رسم مثلث
    glBindVertexArray(VAO);
    glDrawArrays(GL_TRIANGLES, 0, 3);
    glBindVertexArray(0);
}

int main() {
    // مقداردهی اولیه GLFW
    if (!glfwInit()) {
        std::cerr << "Failed to initialize GLFW" << std::endl;
        return -1;
    }
    
    // تنظیمات GLFW
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
    
    // ایجاد پنجره
    GLFWwindow* window = glfwCreateWindow(800, 600, "Shaders", NULL, NULL);
    if (window == NULL) {
        std::cerr << "Failed to create GLFW window" << std::endl;
        glfwTerminate();
        return -1;
    }
    glfwMakeContextCurrent(window);
    
    // مقداردهی اولیه GLEW
    if (glewInit() != GLEW_OK) {
        std::cerr << "Failed to initialize GLEW" << std::endl;
        return -1;
    }
    
    // ایجاد برنامه شیدر
    shaderProgram = createShaderProgram("vertex_shader.glsl", "fragment_shader.glsl");
    if (shaderProgram == 0) {
        std::cerr << "Failed to create shader program" << std::endl;
        return -1;
    }
    
    // تنظیم بافرها
    setupBuffers();
    
    // حلقه اصلی رندرینگ
    while (!glfwWindowShouldClose(window)) {
        render();
        
        // جابجا کردن بافرها و بررسی رویدادها
        glfwSwapBuffers(window);
        glfwPollEvents();
    }
    
    // پاکسازی منابع
    glDeleteVertexArrays(1, &VAO);
    glDeleteBuffers(1, &VBO);
    glDeleteProgram(shaderProgram);
    
    glfwTerminate();
    return 0;
}
```

## افکت‌های پیشرفته با شیدرها

### 1. تغییر رنگ با زمان
```glsl
// شیدر فرگمنت
uniform float time;

void main()
{
    vec3 color = vec3(
        sin(time) * 0.5 + 0.5,
        sin(time + 2.094) * 0.5 + 0.5,
        sin(time + 4.189) * 0.5 + 0.5
    );
    FragColor = vec4(color, 1.0);
}
```

### 2. تغییر شکل با زمان
```glsl
// شیدر رأس
uniform float time;

void main()
{
    vec3 pos = aPos;
    pos.x += sin(time + pos.y) * 0.1;
    pos.y += cos(time + pos.x) * 0.1;
    gl_Position = projection * view * model * vec4(pos, 1.0);
}
```

### 3. افکت موجی
```glsl
// شیدر فرگمنت
uniform float time;
uniform sampler2D texture1;

void main()
{
    vec2 texCoord = TexCoord;
    texCoord.x += sin(texCoord.y * 10.0 + time) * 0.01;
    texCoord.y += cos(texCoord.x * 10.0 + time) * 0.01;
    FragColor = texture(texture1, texCoord);
}
```

## شیدرهای پیشرفته

### 1. شیدر تسل (Tessellation Shader)
برای تقسیم‌بندی سطوح و ایجاد جزئیات بیشتر.

```glsl
#version 410 core
layout(vertices = 3) out;

void main()
{
    if (gl_InvocationID == 0) {
        gl_TessLevelInner[0] = 5.0;
        gl_TessLevelOuter[0] = 5.0;
        gl_TessLevelOuter[1] = 5.0;
        gl_TessLevelOuter[2] = 5.0;
    }
    
    gl_out[gl_InvocationID].gl_Position = gl_in[gl_InvocationID].gl_Position;
}
```

### 2. شیدر محاسباتی (Compute Shader)
برای انجام محاسبات عمومی روی GPU.

```glsl
#version 430

layout(local_size_x = 256) in;

layout(std430, binding = 0) buffer InputBuffer {
    float data[];
} inputBuffer;

layout(std430, binding = 1) buffer OutputBuffer {
    float data[];
} outputBuffer;

void main() {
    uint index = gl_GlobalInvocationID.x;
    outputBuffer.data[index] = inputBuffer.data[index] * 2.0;
}
```
## نکات مهم
1. همیشه خطاهای کامپایل و لینک شیدرها را بررسی کنید
2. از متغیرهای یکنواخت برای انتقال داده‌ها به شیدرها استفاده کنید
3. برای عملکرد بهتر، از حداقل تعداد متغیرهای یکنواخت استفاده کنید
4. برای اشکال‌زدایی شیدرها، از توابع `glGetShaderInfoLog` و `glGetProgramInfoLog` استفاده کنید

## منابع بیشتر
- [مستندات OpenGL](https://www.opengl.org/documentation/)
- [راهنمای شیدرها](https://www.opengl.org/wiki/Shader)
- [کتابخانه GLSL](https://www.khronos.org/opengl/wiki/OpenGL_Shading_Language)
