---
title: شیدرها و بافرها
nav_order: 9
permalink: /shaders/
layout: persian
---

 # شیدرها و بافرها در OpenGL

## مقدمه‌ای بر شیدرها
شیدرها برنامه‌های کوچکی هستند که روی GPU اجرا می‌شوند و نحوه پردازش رئوس و پیکسل‌ها را در OpenGL کنترل می‌کنند. در این بخش، با مفاهیم پایه شیدرها و نحوه استفاده از آنها در OpenGL آشنا می‌شویم. این برنامه‌ها به زبان GLSL (OpenGL Shading Language) نوشته می‌شوند و به شما امکان می‌دهند تا کنترل دقیقی روی پایپلاین رندرینگ داشته باشید.

## انواع اصلی شیدرها

### 1. شیدر رأس (Vertex Shader)
- **هدف**: پردازش هر رأس (vertex) به صورت مجزا
- **ورودی**: مختصات، رنگ، مختصات بافت و... برای هر رأس
- **خروجی**: موقعیت نهایی رأس (در فضای کلیپ) و سایر متغیرهای مورد نیاز برای شیدر فرگمنت

### 2. شیدر فرگمنت (Fragment Shader)
- **هدف**: تعیین رنگ نهایی پیکسل‌ها
- **ورودی**: متغیرهای خروجی از شیدر رأس که به فرگمنت‌ها درون‌یابی شده‌اند
- **خروجی**: رنگ نهایی پیکسل

### 3. شیدر هندسی یا ژئومتری (Geometry Shader) - اختیاری
- **هدف**: تغییر یا ایجاد اشکال هندسی
- **ورودی**: اطلاعات یک شکل اولیه (مثل یک مثلث)
- **خروجی**: می‌تواند اشکال جدید تولید کند یا اشکال موجود را تغییر دهد

### 4. شیدر تِسِلِیشن (Tessellation Shader) - اختیاری
- **هدف**: افزایش جزئیات مدل‌ها با تقسیم اشکال به قطعات کوچکتر
- **ورودی**: اطلاعات یک شکل اولیه
- **خروجی**: شکل با تعداد رئوس بیشتر

### 5. شیدر محاسباتی (Compute Shader) - اختیاری
- **هدف**: انجام محاسبات عمومی روی GPU بدون ارتباط مستقیم با پایپلاین گرافیکی
- **کاربردها**: فیزیک، پردازش تصویر و...

## زبان GLSL

GLSL یک زبان شبیه به C است که برای نوشتن شیدرها استفاده می‌شود. ویژگی‌های اصلی آن عبارتند از:

### انواع داده
- **انواع اسکالر**: `float`, `int`, `bool`
- **انواع برداری**: `vec2`, `vec3`, `vec4` (بردارهای شناور)، `ivec2`, `ivec3`, `ivec4` (بردارهای صحیح)
- **انواع ماتریسی**: `mat2`, `mat3`, `mat4`
- **انواع سمپلر**: `sampler2D`, `sampler3D`, `samplerCube` (برای دسترسی به بافت‌ها)

### متغیرهای ورودی و خروجی
- **`in`**: برای متغیرهای ورودی
- **`out`**: برای متغیرهای خروجی
- **`uniform`**: متغیرهای ثابت که از طرف CPU تنظیم می‌شوند و برای همه رئوس یا فرگمنت‌ها یکسان هستند

### توابع مفید داخلی
- توابع ریاضی: `sin`, `cos`, `pow`, `sqrt`, ...
- توابع درون‌یابی: `mix`, `smoothstep`, ...
- توابع هندسی: `length`, `distance`, `normalize`, `dot`, `cross`, ...
- توابع بافت: `texture`, ...


## نگاه دقیق به انواع شیدر

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

## عملکرد و بهینه‌سازی شیدرها

1. **کاهش if-else**: شرط‌های شاخه‌دار می‌توانند عملکرد را کاهش دهند.
2. **کاهش محاسبات پیچیده**: محاسبات سنگین را به شیدر رأس منتقل کنید (که کمتر اجرا می‌شود).
3. **استفاده از توابع داخلی**: توابع داخلی GLSL معمولاً در سخت‌افزار بهینه‌سازی شده‌اند.
4. **پیش‌محاسبه مقادیر ثابت**: در صورت امکان، مقادیر ثابت را از CPU ارسال کنید.

## نکات مهم شیدرها
1. همیشه خطاهای کامپایل و لینک شیدرها را بررسی کنید
2. از متغیرهای یکنواخت برای انتقال داده‌ها به شیدرها استفاده کنید
3. برای عملکرد بهتر، از حداقل تعداد متغیرهای یکنواخت استفاده کنید
4. برای اشکال‌زدایی شیدرها، از توابع `glGetShaderInfoLog` و `glGetProgramInfoLog` استفاده کنید


# بافرها و آرایه‌های رأس در OpenGL

## مقدمه‌ای بر بافرها

برای رندر کردن هر چیزی در OpenGL، باید داده‌های آن را به GPU منتقل کنیم. بافرها مکانیزم اصلی برای این انتقال هستند. در این‌جا، با سه نوع اصلی بافر آشنا می‌شویم:

1. **VBO (Vertex Buffer Object)**: برای ذخیره داده‌های رأس
2. **VAO (Vertex Array Object)**: برای ذخیره تنظیمات ویژگی‌های رأس
3. **EBO/IBO (Element Buffer Object/Index Buffer Object)**: برای ذخیره شاخص‌های رأس

## VBO (Vertex Buffer Object)

VBO یک بافر OpenGL است که داده‌های رأس مانند موقعیت، رنگ، مختصات بافت و... را ذخیره می‌کند.

### ایجاد و استفاده از VBO

```cpp
// ایجاد یک شناسه برای VBO
unsigned int VBO;
glGenBuffers(1, &VBO);

// فعال کردن بافر
glBindBuffer(GL_ARRAY_BUFFER, VBO);

// تعریف داده‌های رأس (در این مثال یک مثلث)
float vertices[] = {
    // موقعیت (x, y, z)    // رنگ (r, g, b)
    -0.5f, -0.5f, 0.0f,    1.0f, 0.0f, 0.0f,  // رأس پایین چپ
     0.5f, -0.5f, 0.0f,    0.0f, 1.0f, 0.0f,  // رأس پایین راست
     0.0f,  0.5f, 0.0f,    0.0f, 0.0f, 1.0f   // رأس بالا
};

// کپی داده‌های رأس به بافر
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
```

### پارامترهای glBufferData

- **GL_STATIC_DRAW**: داده‌ها یک بار تنظیم شده و بارها استفاده می‌شوند.
- **GL_DYNAMIC_DRAW**: داده‌ها چندین بار تغییر کرده و بارها استفاده می‌شوند.
- **GL_STREAM_DRAW**: داده‌ها هر فریم تغییر می‌کنند.

## VAO (Vertex Array Object)

VAO یک شیء در OpenGL است که تمام تنظیمات مربوط به ورودی‌های رأس را ذخیره می‌کند. استفاده از VAO فرآیند رندرینگ را ساده‌تر می‌کند، زیرا نیازی به تنظیم مجدد ویژگی‌های رأس نیست.

### ایجاد و استفاده از VAO

```cpp
// ایجاد یک شناسه برای VAO
unsigned int VAO;
glGenVertexArrays(1, &VAO);

// فعال کردن VAO
glBindVertexArray(VAO);

// اکنون VBO را بایند کنید و ویژگی‌های رأس را تنظیم کنید
glBindBuffer(GL_ARRAY_BUFFER, VBO);

// تنظیم ویژگی موقعیت (attribute 0)
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);

// تنظیم ویژگی رنگ (attribute 1)
glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)(3 * sizeof(float)));
glEnableVertexAttribArray(1);

// غیرفعال کردن VAO (اختیاری)
glBindVertexArray(0);
```

### توضیح پارامترهای glVertexAttribPointer

1. **شماره ویژگی**: باید با شماره ویژگی در شیدر مطابقت داشته باشد.
2. **اندازه**: تعداد مؤلفه‌های هر ویژگی (مثلاً 3 برای xyz).
3. **نوع**: نوع داده‌ها (GL_FLOAT، GL_INT و...).
4. **عادی‌سازی**: آیا داده‌ها باید عادی‌سازی شوند؟
5. **stride**: فاصله بین ویژگی‌های متوالی (به بایت).
6. **offset**: جابجایی شروع ویژگی در بافر (به بایت).

## EBO/IBO (Element Buffer Object/Index Buffer Object)

EBO یک بافر OpenGL است که شاخص‌های رئوس را ذخیره می‌کند. اجازه می‌دهد رئوس را یک بار تعریف کنید و با شاخص‌ها به آن‌ها مراجعه کنید، که به معنای صرفه‌جویی در حافظه است.

### ایجاد و استفاده از EBO

```cpp
// تعریف داده‌های رأس برای یک مربع
float vertices[] = {
     0.5f,  0.5f, 0.0f,  // بالا راست
     0.5f, -0.5f, 0.0f,  // پایین راست
    -0.5f, -0.5f, 0.0f,  // پایین چپ
    -0.5f,  0.5f, 0.0f   // بالا چپ
};

// تعریف شاخص‌ها برای دو مثلث که یک مربع را تشکیل می‌دهند
unsigned int indices[] = {
    0, 1, 3,  // مثلث اول
    1, 2, 3   // مثلث دوم
};

// ایجاد EBO
unsigned int EBO;
glGenBuffers(1, &EBO);

// بایند کردن EBO
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);

// ترسیم با استفاده از EBO
// توجه: این کد باید بعد از بایند کردن VAO و شیدر مناسب اجرا شود
glBindVertexArray(VAO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
```

## مثال کامل: ترسیم یک مثلث رنگی

```cpp
#include <glad/glad.h>
#include <GLFW/glfw3.h>
#include <iostream>

// کد شیدر رأس
const char* vertexShaderSource = R"(
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aColor;
out vec3 ourColor;
void main()
{
    gl_Position = vec4(aPos, 1.0);
    ourColor = aColor;
}
)";

// کد شیدر فرگمنت
const char* fragmentShaderSource = R"(
#version 330 core
in vec3 ourColor;
out vec4 FragColor;
void main()
{
    FragColor = vec4(ourColor, 1.0);
}
)";

int main()
{
    // مقداردهی اولیه GLFW و ایجاد پنجره
    glfwInit();
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

    GLFWwindow* window = glfwCreateWindow(800, 600, "Triangle with VBO & VAO", NULL, NULL);
    glfwMakeContextCurrent(window);

    // مقداردهی اولیه GLAD
    gladLoadGLLoader((GLADloadproc)glfwGetProcAddress);

    // کامپایل و لینک شیدرها
    unsigned int vertexShader = glCreateShader(GL_VERTEX_SHADER);
    glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
    glCompileShader(vertexShader);

    unsigned int fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
    glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
    glCompileShader(fragmentShader);

    unsigned int shaderProgram = glCreateProgram();
    glAttachShader(shaderProgram, vertexShader);
    glAttachShader(shaderProgram, fragmentShader);
    glLinkProgram(shaderProgram);

    glDeleteShader(vertexShader);
    glDeleteShader(fragmentShader);

    // تعریف داده‌های رأس
    float vertices[] = {
        // موقعیت            // رنگ
        -0.5f, -0.5f, 0.0f,  1.0f, 0.0f, 0.0f,  // پایین چپ
         0.5f, -0.5f, 0.0f,  0.0f, 1.0f, 0.0f,  // پایین راست
         0.0f,  0.5f, 0.0f,  0.0f, 0.0f, 1.0f   // بالا
    };

    // ایجاد و تنظیم VBO و VAO
    unsigned int VBO, VAO;
    glGenVertexArrays(1, &VAO);
    glGenBuffers(1, &VBO);

    glBindVertexArray(VAO);

    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

    // تنظیم ویژگی موقعیت
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)0);
    glEnableVertexAttribArray(0);

    // تنظیم ویژگی رنگ
    glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)(3 * sizeof(float)));
    glEnableVertexAttribArray(1);

    // حلقه اصلی رندرینگ
    while (!glfwWindowShouldClose(window))
    {
        // پاک کردن صفحه
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

        // ترسیم مثلث
        glUseProgram(shaderProgram);
        glBindVertexArray(VAO);
        glDrawArrays(GL_TRIANGLES, 0, 3);

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

## انواع دستورات ترسیم در OpenGL

### glDrawArrays
برای ترسیم مستقیم از داده‌های رأس بدون شاخص‌ها استفاده می‌شود:

```cpp
// ترسیم 3 رأس از شاخص 0 به صورت مثلث‌ها
glDrawArrays(GL_TRIANGLES, 0, 3);
```

### glDrawElements
برای ترسیم با استفاده از شاخص‌ها (EBO) استفاده می‌شود:

```cpp
// ترسیم 6 شاخص به صورت مثلث‌ها
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
```

### حالت‌های ترسیم (یادآوری)

- **GL_POINTS**: هر رأس به صورت یک نقطه ترسیم می‌شود.
- **GL_LINES**: هر دو رأس متوالی یک خط را تشکیل می‌دهند.
- **GL_LINE_STRIP**: مجموعه‌ای از خطوط متصل.
- **GL_LINE_LOOP**: مجموعه‌ای از خطوط متصل که آخرین رأس به اولین رأس متصل می‌شود.
- **GL_TRIANGLES**: هر سه رأس متوالی یک مثلث را تشکیل می‌دهند.
- **GL_TRIANGLE_STRIP**: مجموعه‌ای از مثلث‌های متصل.
- **GL_TRIANGLE_FAN**: مجموعه‌ای از مثلث‌ها که یک رأس مشترک دارند.

## نکات پیشرفته

### استفاده چندگانه از VBO و VAO

می‌توانید چندین VBO را به یک VAO متصل کنید:

```cpp
// VBO اول - موقعیت‌ها
glBindBuffer(GL_ARRAY_BUFFER, VBO1);
glBufferData(GL_ARRAY_BUFFER, sizeof(positions), positions, GL_STATIC_DRAW);
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, (void*)0);
glEnableVertexAttribArray(0);

// VBO دوم - رنگ‌ها
glBindBuffer(GL_ARRAY_BUFFER, VBO2);
glBufferData(GL_ARRAY_BUFFER, sizeof(colors), colors, GL_STATIC_DRAW);
glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 0, (void*)0);
glEnableVertexAttribArray(1);
```

### به‌روزرسانی بافرها

برای به‌روزرسانی بخشی از داده‌ها در یک بافر موجود:

```cpp
// به‌روزرسانی بخشی از بافر
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferSubData(GL_ARRAY_BUFFER, offset, size, data);
```

## نکات بهینه‌سازی بافر

1. **استفاده از VAO**: همیشه از VAO استفاده کنید، حتی برای یک شیء ساده.
2. **تنظیم مناسب حالت ذخیره‌سازی**: از GL_STATIC_DRAW برای داده‌های ثابت و GL_DYNAMIC_DRAW برای داده‌های متغیر استفاده کنید.
3. **ترکیب داده‌ها**: اگر ممکن است، داده‌های مرتبط را در یک VBO ترکیب کنید.
4. **استفاده از شاخص‌ها**: برای صرفه‌جویی در حافظه، از EBO استفاده کنید.

## منابع بیشتر
- [مستندات OpenGL](https://www.opengl.org/documentation/)
- [راهنمای شیدرها](https://www.opengl.org/wiki/Shader)
- [کتابخانه GLSL](https://www.khronos.org/opengl/wiki/OpenGL_Shading_Language)
