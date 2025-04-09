---
title: رندرینگ
nav_order: 13
permalink: /rendering/
layout: persian
---

# تکنیک‌های پیشرفته رندرینگ در OpenGL

## مقدمه
در این بخش، با تکنیک‌های پیشرفته رندرینگ در OpenGL آشنا می‌شویم. این تکنیک‌ها به شما امکان می‌دهند صحنه‌های گرافیکی پیچیده‌تر و واقع‌گرایانه‌تری ایجاد کنید.

## رندرینگ چند پاسه (Multi-Pass Rendering)

### 1. رندرینگ به بافر (Render to Texture)
```cpp
// ایجاد فریم‌بافر
GLuint framebuffer;
glGenFramebuffers(1, &framebuffer);
glBindFramebuffer(GL_FRAMEBUFFER, framebuffer);

// ایجاد بافت برای ذخیره نتیجه
GLuint textureColorbuffer;
glGenTextures(1, &textureColorbuffer);
glBindTexture(GL_TEXTURE_2D, textureColorbuffer);
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, NULL);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);

// اتصال بافت به فریم‌بافر
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, textureColorbuffer, 0);

// ایجاد بافر عمق و استنسیل
GLuint rbo;
glGenRenderbuffers(1, &rbo);
glBindRenderbuffer(GL_RENDERBUFFER, rbo);
glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH24_STENCIL8, width, height);
glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_RENDERBUFFER, rbo);

// بررسی وضعیت فریم‌بافر
if (glCheckFramebufferStatus(GL_FRAMEBUFFER) != GL_FRAMEBUFFER_COMPLETE) {
    std::cout << "Framebuffer is not complete!" << std::endl;
}

// بازگشت به فریم‌بافر پیش‌فرض
glBindFramebuffer(GL_FRAMEBUFFER, 0);
```

### 2. پاس اول: رندرینگ صحنه
```cpp
// بایند کردن فریم‌بافر
glBindFramebuffer(GL_FRAMEBUFFER, framebuffer);
glClearColor(0.1f, 0.1f, 0.1f, 1.0f);
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

// رندرینگ صحنه
renderScene();

// بازگشت به فریم‌بافر پیش‌فرض
glBindFramebuffer(GL_FRAMEBUFFER, 0);
```

### 3. پاس دوم: اعمال افکت‌ها
```cpp
// استفاده از شیدر پس‌پردازش
glUseProgram(postProcessShader);

// تنظیم بافت رندر شده
glActiveTexture(GL_TEXTURE0);
glBindTexture(GL_TEXTURE_2D, textureColorbuffer);

// رسم یک صفحه تمام صفحه
renderScreenQuad();
```

## سایه‌های پویا (Dynamic Shadows)

### 1. سایه‌های نقشه عمق (Shadow Mapping)
```cpp
// ایجاد فریم‌بافر برای نقشه سایه
GLuint shadowMapFBO;
glGenFramebuffers(1, &shadowMapFBO);
glBindFramebuffer(GL_FRAMEBUFFER, shadowMapFBO);

// ایجاد بافت نقشه عمق
GLuint shadowMap;
glGenTextures(1, &shadowMap);
glBindTexture(GL_TEXTURE_2D, shadowMap);
glTexImage2D(GL_TEXTURE_2D, 0, GL_DEPTH_COMPONENT, shadowWidth, shadowHeight, 0, GL_DEPTH_COMPONENT, GL_FLOAT, NULL);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_BORDER);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_BORDER);
float borderColor[] = { 1.0f, 1.0f, 1.0f, 1.0f };
glTexParameterfv(GL_TEXTURE_2D, GL_TEXTURE_BORDER_COLOR, borderColor);

// اتصال نقشه عمق به فریم‌بافر
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT, GL_TEXTURE_2D, shadowMap, 0);
glDrawBuffer(GL_NONE);
glReadBuffer(GL_NONE);
```

### 2. پاس اول: رندرینگ از دید نور
```cpp
// تنظیم ماتریس برجستگی برای نور
glm::mat4 lightProjection = glm::ortho(-10.0f, 10.0f, -10.0f, 10.0f, 0.1f, 100.0f);
glm::mat4 lightView = glm::lookAt(lightPos, glm::vec3(0.0f), glm::vec3(0.0f, 1.0f, 0.0f));
glm::mat4 lightSpaceMatrix = lightProjection * lightView;

// استفاده از شیدر سایه
glUseProgram(shadowShader);
glUniformMatrix4fv(glGetUniformLocation(shadowShader, "lightSpaceMatrix"), 1, GL_FALSE, glm::value_ptr(lightSpaceMatrix));

// رندرینگ صحنه
glBindFramebuffer(GL_FRAMEBUFFER, shadowMapFBO);
glClear(GL_DEPTH_BUFFER_BIT);
renderScene();
```

### 3. پاس دوم: رندرینگ نهایی با سایه‌ها
```cpp
// استفاده از شیدر اصلی
glUseProgram(shader);
glUniformMatrix4fv(glGetUniformLocation(shader, "lightSpaceMatrix"), 1, GL_FALSE, glm::value_ptr(lightSpaceMatrix));

// تنظیم نقشه سایه
glActiveTexture(GL_TEXTURE1);
glBindTexture(GL_TEXTURE_2D, shadowMap);
glUniform1i(glGetUniformLocation(shader, "shadowMap"), 1);

// رندرینگ صحنه
glBindFramebuffer(GL_FRAMEBUFFER, 0);
renderScene();
```

## پس‌پردازش (Post-Processing)

### 1. افکت بلور (Blur Effect)
```glsl
// شیدر فرگمنت برای بلور
#version 330 core
out vec4 FragColor;

in vec2 TexCoords;

uniform sampler2D screenTexture;
uniform float offset;

void main()
{
    vec2 offsets[9] = vec2[](
        vec2(-offset, offset),  // بالا-چپ
        vec2(0.0f, offset),    // بالا-وسط
        vec2(offset, offset),  // بالا-راست
        vec2(-offset, 0.0f),   // وسط-چپ
        vec2(0.0f, 0.0f),      // وسط-وسط
        vec2(offset, 0.0f),    // وسط-راست
        vec2(-offset, -offset),// پایین-چپ
        vec2(0.0f, -offset),   // پایین-وسط
        vec2(offset, -offset)  // پایین-راست
    );

    float kernel[9] = float[](
        1.0/16, 2.0/16, 1.0/16,
        2.0/16, 4.0/16, 2.0/16,
        1.0/16, 2.0/16, 1.0/16
    );

    vec3 result = vec3(0.0);
    for(int i = 0; i < 9; i++)
        result += texture(screenTexture, TexCoords.st + offsets[i]).rgb * kernel[i];
    
    FragColor = vec4(result, 1.0);
}
```

### 2. افکت لبه‌یابی (Edge Detection)
```glsl
// شیدر فرگمنت برای لبه‌یابی
#version 330 core
out vec4 FragColor;

in vec2 TexCoords;

uniform sampler2D screenTexture;
uniform float offset;

void main()
{
    vec2 offsets[9] = vec2[](
        vec2(-offset, offset),
        vec2(0.0f, offset),
        vec2(offset, offset),
        vec2(-offset, 0.0f),
        vec2(0.0f, 0.0f),
        vec2(offset, 0.0f),
        vec2(-offset, -offset),
        vec2(0.0f, -offset),
        vec2(offset, -offset)
    );

    float kernel[9] = float[](
        1.0,  1.0, 1.0,
        1.0, -8.0, 1.0,
        1.0,  1.0, 1.0
    );

    vec3 result = vec3(0.0);
    for(int i = 0; i < 9; i++)
        result += texture(screenTexture, TexCoords.st + offsets[i]).rgb * kernel[i];
    
    FragColor = vec4(result, 1.0);
}
```

### 3. افکت HDR (High Dynamic Range)
```cpp
// تنظیمات HDR
glBindFramebuffer(GL_FRAMEBUFFER, hdrFBO);
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

// رندرینگ صحنه با HDR
glUseProgram(hdrShader);
glUniform1i(glGetUniformLocation(hdrShader, "hdr"), hdr);
glUniform1f(glGetUniformLocation(hdrShader, "exposure"), exposure);

// رسم صفحه تمام صفحه با بافت HDR
renderScreenQuad();
```

## رندرینگ پیشرفته هندسه

### 1. رندرینگ اینستنس (Instanced Rendering)
```cpp
// ایجاد بافر برای ماتریس‌های تبدیل
GLuint instanceVBO;
glGenBuffers(1, &instanceVBO);
glBindBuffer(GL_ARRAY_BUFFER, instanceVBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(glm::mat4) * amount, &modelMatrices[0], GL_STATIC_DRAW);

// تنظیم ویژگی‌های رأس برای ماتریس‌ها
glEnableVertexAttribArray(3);
glVertexAttribPointer(3, 4, GL_FLOAT, GL_FALSE, sizeof(glm::mat4), (void*)0);
glEnableVertexAttribArray(4);
glVertexAttribPointer(4, 4, GL_FLOAT, GL_FALSE, sizeof(glm::mat4), (void*)(sizeof(glm::vec4)));
glEnableVertexAttribArray(5);
glVertexAttribPointer(5, 4, GL_FLOAT, GL_FALSE, sizeof(glm::mat4), (void*)(2 * sizeof(glm::vec4)));
glEnableVertexAttribArray(6);
glVertexAttribPointer(6, 4, GL_FLOAT, GL_FALSE, sizeof(glm::mat4), (void*)(3 * sizeof(glm::vec4)));

glVertexAttribDivisor(3, 1);
glVertexAttribDivisor(4, 1);
glVertexAttribDivisor(5, 1);
glVertexAttribDivisor(6, 1);

// رندرینگ اینستنس
glDrawArraysInstanced(GL_TRIANGLES, 0, vertices.size(), amount);
```

### 2. رندرینگ هندسه پویا (Dynamic Geometry)
```cpp
// ایجاد بافر برای هندسه پویا
GLuint dynamicVBO;
glGenBuffers(1, &dynamicVBO);
glBindBuffer(GL_ARRAY_BUFFER, dynamicVBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(float) * maxVertices, NULL, GL_DYNAMIC_DRAW);

// به‌روزرسانی هندسه
glBindBuffer(GL_ARRAY_BUFFER, dynamicVBO);
glBufferSubData(GL_ARRAY_BUFFER, 0, sizeof(float) * vertices.size(), vertices.data());

// رندرینگ هندسه پویا
glDrawArrays(GL_TRIANGLES, 0, vertices.size());
```

## رندرینگ پیشرفته نور

### 1. نورپردازی مبتنی بر تصویر (Image-Based Lighting)
```cpp
// بارگذاری محیط اطراف
unsigned int hdrTexture = loadHDR("environment.hdr");
unsigned int envCubemap = convertHDRtoCubemap(hdrTexture);
unsigned int irradianceMap = createIrradianceMap(envCubemap);
unsigned int prefilterMap = createPrefilterMap(envCubemap);
unsigned int brdfLUT = createBRDFLUT();

// استفاده از نقشه‌های محیطی در شیدر
glActiveTexture(GL_TEXTURE0);
glBindTexture(GL_TEXTURE_CUBE_MAP, irradianceMap);
glActiveTexture(GL_TEXTURE1);
glBindTexture(GL_TEXTURE_CUBE_MAP, prefilterMap);
glActiveTexture(GL_TEXTURE2);
glBindTexture(GL_TEXTURE_2D, brdfLUT);
```

### 2. نورپردازی مبتنی بر فیزیک (Physically Based Rendering)
```glsl
// شیدر فرگمنت برای PBR
#version 330 core
out vec4 FragColor;

in vec3 WorldPos;
in vec3 Normal;
in vec2 TexCoords;

uniform sampler2D albedoMap;
uniform sampler2D normalMap;
uniform sampler2D metallicMap;
uniform sampler2D roughnessMap;
uniform sampler2D aoMap;

uniform samplerCube irradianceMap;
uniform samplerCube prefilterMap;
uniform sampler2D brdfLUT;

const float PI = 3.14159265359;

// تابع Fresnel-Schlick
vec3 fresnelSchlick(float cosTheta, vec3 F0)
{
    return F0 + (max(vec3(1.0 - roughness), F0) - F0) * pow(clamp(1.0 - cosTheta, 0.0, 1.0), 5.0);
}

// تابع توزیع نرمال
float DistributionGGX(vec3 N, vec3 H, float roughness)
{
    float a = roughness*roughness;
    float a2 = a*a;
    float NdotH = max(dot(N, H), 0.0);
    float NdotH2 = NdotH*NdotH;

    float nom   = a2;
    float denom = (NdotH2 * (a2 - 1.0) + 1.0);
    denom = PI * denom * denom;

    return nom / denom;
}

// تابع هندسه
float GeometrySchlickGGX(float NdotV, float roughness)
{
    float r = (roughness + 1.0);
    float k = (r*r) / 8.0;

    float nom   = NdotV;
    float denom = NdotV * (1.0 - k) + k;

    return nom / denom;
}

void main()
{
    // نمونه‌برداری از بافت‌ها
    vec3 albedo = texture(albedoMap, TexCoords).rgb;
    float metallic = texture(metallicMap, TexCoords).r;
    float roughness = texture(roughnessMap, TexCoords).r;
    float ao = texture(aoMap, TexCoords).r;

    // محاسبه نورپردازی PBR
    vec3 N = normalize(Normal);
    vec3 V = normalize(cameraPos - WorldPos);
    vec3 R = reflect(-V, N);

    vec3 F0 = vec3(0.04);
    F0 = mix(F0, albedo, metallic);

    vec3 Lo = vec3(0.0);
    for(int i = 0; i < lightCount; i++)
    {
        vec3 L = normalize(lightPositions[i] - WorldPos);
        vec3 H = normalize(V + L);
        float distance = length(lightPositions[i] - WorldPos);
        float attenuation = 1.0 / (distance * distance);
        vec3 radiance = lightColors[i] * attenuation;

        float NDF = DistributionGGX(N, H, roughness);
        float G = GeometrySchlickGGX(max(dot(N, V), 0.0), roughness) * GeometrySchlickGGX(max(dot(N, L), 0.0), roughness);
        vec3 F = fresnelSchlick(max(dot(H, V), 0.0), F0);

        vec3 numerator = NDF * G * F;
        float denominator = 4.0 * max(dot(N, V), 0.0) * max(dot(N, L), 0.0) + 0.0001;
        vec3 specular = numerator / denominator;

        vec3 kS = F;
        vec3 kD = vec3(1.0) - kS;
        kD *= 1.0 - metallic;

        float NdotL = max(dot(N, L), 0.0);
        Lo += (kD * albedo / PI + specular) * radiance * NdotL;
    }

    // نورپردازی محیطی
    vec3 F = fresnelSchlick(max(dot(N, V), 0.0), F0);
    vec3 kS = F;
    vec3 kD = 1.0 - kS;
    kD *= 1.0 - metallic;

    vec3 irradiance = texture(irradianceMap, N).rgb;
    vec3 diffuse = irradiance * albedo;

    const float MAX_REFLECTION_LOD = 4.0;
    vec3 prefilteredColor = textureLod(prefilterMap, R, roughness * MAX_REFLECTION_LOD).rgb;
    vec2 brdf = texture(brdfLUT, vec2(max(dot(N, V), 0.0), roughness)).rg;
    vec3 specular = prefilteredColor * (F * brdf.x + brdf.y);

    vec3 ambient = (kD * diffuse + specular) * ao;
    vec3 color = ambient + Lo;

    // تصحیح گاما
    color = color / (color + vec3(1.0));
    color = pow(color, vec3(1.0/2.2));

    FragColor = vec4(color, 1.0);
}
```

## تمرینات
1. یک سیستم سایه‌پردازی پویا با استفاده از نقشه‌های عمق پیاده‌سازی کنید
2. یک سیستم پس‌پردازش با افکت‌های مختلف (بلور، لبه‌یابی، HDR) ایجاد کنید
3. یک سیستم رندرینگ اینستنس برای نمایش تعداد زیادی از اشیاء مشابه پیاده‌سازی کنید
4. یک سیستم نورپردازی مبتنی بر فیزیک با استفاده از PBR پیاده‌سازی کنید

## نکات مهم
1. برای عملکرد بهتر، از فریم‌بافرهای متعدد استفاده کنید
2. برای سایه‌های واقع‌گرایانه، از تکنیک‌های پیشرفته‌تر مانند PCF یا PCSS استفاده کنید
3. برای نورپردازی واقع‌گرایانه، از PBR استفاده کنید
4. برای بهینه‌سازی عملکرد، از رندرینگ اینستنس استفاده کنید

## منابع بیشتر
- [مستندات OpenGL](https://www.opengl.org/documentation/)
- [راهنمای رندرینگ پیشرفته](https://www.opengl.org/wiki/Advanced_OpenGL)
- [راهنمای PBR](https://learnopengl.com/PBR/Theory) 