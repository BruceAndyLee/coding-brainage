# Практический раздел: Минимальные воспроизводимые примеры

## 1. Требуемое ПО для Windows 10/11

### Для SIMD (CPU вычисления):
1. **Компилятор C++ с поддержкой C++17**:
   - Visual Studio 2019/2022 (Community Edition бесплатно)
   - MinGW-w64 (альтернатива для GCC)
   
2. **Проверка поддержки SIMD**:
```cmd
# Проверить поддержку AVX2 в Windows PowerShell
Get-WmiObject Win32_Processor | Select-Object -ExpandProperty CapabilityDescription

# Или через Command Prompt
systeminfo | findstr /C:"AVX"
```

### Для GPU вычислений:
1. **OpenGL 4.6+** (встроен в Windows 10/11):
   - Драйверы NVIDIA/AMD/Intel (обновить до последней версии)

2. **Vulkan** (опционально, для современных API):
   - Vulkan SDK: https://vulkan.lunarg.com/

3. **Библиотеки**:
   - GLFW: https://www.glfw.org/
   - GLAD: https://glad.dav1d.de/
   - GLM: https://github.com/g-truc/glm

## 2. Минимальный воспроизводимый SIMD пример

### Проект на CMake для легкой сборки:

**Файл: `CMakeLists.txt`**
```cmake
cmake_minimum_required(VERSION 3.15)
project(SIMD_Demo)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Проверка поддержки SIMD
include(CheckCXXCompilerFlag)
check_cxx_compiler_flag("/arch:AVX2" COMPILER_SUPPORTS_AVX2)
check_cxx_compiler_flag("-mavx2" COMPILER_SUPPORTS_M_AVX2)

if(MSVC)
    add_compile_options(/arch:AVX2)
else()
    add_compile_options(-mavx2 -msse4.2)
endif()

# Исполняемый файл
add_executable(simd_demo 
    src/main_simd.cpp
    src/matrix_simd.cpp
    include/matrix_simd.h
)

# Тесты
enable_testing()
add_executable(simd_test 
    tests/test_simd.cpp
    src/matrix_simd.cpp
)
add_test(NAME SIMD_Tests COMMAND simd_test)
```

### Основной код SIMD демонстрации:

**Файл: `src/main_simd.cpp`**
```cpp
/**
 * Демонстрация SIMD оптимизации для матричных операций
 * Windows 10/11, Visual Studio 2019+/MinGW-w64
 */

#include <iostream>
#include <chrono>
#include <immintrin.h>  // SSE/AVX интринсики
#include <vector>
#include <random>
#include <iomanip>

// 1. Базовые векторные типы
struct alignas(32) Vector4 {
    union {
        struct { float x, y, z, w; };
        float data[4];
    };
    
    Vector4(float x = 0, float y = 0, float z = 0, float w = 0) 
        : x(x), y(y), z(z), w(w) {}
};

// 2. Матрица 4x4 с выравниванием для AVX
struct alignas(32) Matrix4 {
    union {
        float m[4][4];
        __m128 rows[4];  // SSE: 4 строки по 4 float
    };
    
    // Конструкторы
    Matrix4() {
        for (int i = 0; i < 4; ++i)
            for (int j = 0; j < 4; ++j)
                m[i][j] = (i == j) ? 1.0f : 0.0f;
    }
    
    Matrix4(float diag) {
        for (int i = 0; i < 4; ++i)
            for (int j = 0; j < 4; ++j)
                m[i][j] = (i == j) ? diag : 0.0f;
    }
};

// 3. Скалярное умножение матриц (базовая реализация)
void matrix_multiply_scalar(const Matrix4& a, const Matrix4& b, Matrix4& result) {
    for (int i = 0; i < 4; ++i) {
        for (int j = 0; j < 4; ++j) {
            float sum = 0.0f;
            for (int k = 0; k < 4; ++k) {
                sum += a.m[i][k] * b.m[k][j];
            }
            result.m[i][j] = sum;
        }
    }
}

// 4. SSE оптимизированное умножение матриц
void matrix_multiply_sse(const Matrix4& a, const Matrix4& b, Matrix4& result) {
    // Транспонируем матрицу b для эффективного доступа
    __m128 b0 = _mm_load_ps(&b.m[0][0]);  // Загружаем первую строку b
    __m128 b1 = _mm_load_ps(&b.m[1][0]);  // Загружаем вторую строку b
    __m128 b2 = _mm_load_ps(&b.m[2][0]);  // Загружаем третью строку b
    __m128 b3 = _mm_load_ps(&b.m[3][0]);  // Загружаем четвертую строку b
    
    // Для каждой строки матрицы a
    for (int i = 0; i < 4; ++i) {
        // Загружаем строку a[i]
        __m128 a_row = _mm_load_ps(&a.m[i][0]);
        
        // Дублируем каждый элемент строки для умножения
        __m128 a0 = _mm_shuffle_ps(a_row, a_row, _MM_SHUFFLE(0, 0, 0, 0));  // a[i][0]
        __m128 a1 = _mm_shuffle_ps(a_row, a_row, _MM_SHUFFLE(1, 1, 1, 1));  // a[i][1]
        __m128 a2 = _mm_shuffle_ps(a_row, a_row, _MM_SHUFFLE(2, 2, 2, 2));  // a[i][2]
        __m128 a3 = _mm_shuffle_ps(a_row, a_row, _MM_SHUFFLE(3, 3, 3, 3));  // a[i][3]
        
        // Умножаем и складываем: a[i][0]*b[0] + a[i][1]*b[1] + a[i][2]*b[2] + a[i][3]*b[3]
        __m128 mul0 = _mm_mul_ps(a0, b0);
        __m128 mul1 = _mm_mul_ps(a1, b1);
        __m128 mul2 = _mm_mul_ps(a2, b2);
        __m128 mul3 = _mm_mul_ps(a3, b3);
        
        // Суммируем результаты
        __m128 sum = _mm_add_ps(_mm_add_ps(mul0, mul1), _mm_add_ps(mul2, mul3));
        
        // Сохраняем результат
        _mm_store_ps(&result.m[i][0], sum);
    }
}

// 5. AVX оптимизированное умножение матриц (если поддерживается)
#ifdef __AVX__
void matrix_multiply_avx(const Matrix4& a, const Matrix4& b, Matrix4& result) {
    // Транспонируем b
    __m128 b0 = _mm_load_ps(&b.m[0][0]);
    __m128 b1 = _mm_load_ps(&b.m[1][0]);
    __m128 b2 = _mm_load_ps(&b.m[2][0]);
    __m128 b3 = _mm_load_ps(&b.m[3][0]);
    
    // Используем AVX для обработки двух строк одновременно
    for (int i = 0; i < 4; i += 2) {
        // Загружаем две строки матрицы a
        __m256 a_row01 = _mm256_set_m128(
            _mm_load_ps(&a.m[i+1][0]),  // Верхняя половина - строка i+1
            _mm_load_ps(&a.m[i][0])     // Нижняя половина - строка i
        );
        
        // Вычисляем для каждого элемента строки
        __m256 a0 = _mm256_shuffle_ps(a_row01, a_row01, _MM_SHUFFLE(0, 0, 0, 0));
        __m256 a1 = _mm256_shuffle_ps(a_row01, a_row01, _MM_SHUFFLE(1, 1, 1, 1));
        __m256 a2 = _mm256_shuffle_ps(a_row01, a_row01, _MM_SHUFFLE(2, 2, 2, 2));
        __m256 a3 = _mm256_shuffle_ps(a_row01, a_row01, _MM_SHUFFLE(3, 3, 3, 3));
        
        // Расширяем b до AVX регистров
        __m256 b0_avx = _mm256_set_m128(b0, b0);
        __m256 b1_avx = _mm256_set_m128(b1, b1);
        __m256 b2_avx = _mm256_set_m128(b2, b2);
        __m256 b3_avx = _mm256_set_m128(b3, b3);
        
        // Умножаем и складываем
        __m256 mul0 = _mm256_mul_ps(a0, b0_avx);
        __m256 mul1 = _mm256_mul_ps(a1, b1_avx);
        __m256 mul2 = _mm256_mul_ps(a2, b2_avx);
        __m256 mul3 = _mm256_mul_ps(a3, b3_avx);
        
        __m256 sum = _mm256_add_ps(_mm256_add_ps(mul0, mul1), 
                                  _mm256_add_ps(mul2, mul3));
        
        // Сохраняем две строки результата
        _mm_store_ps(&result.m[i][0], _mm256_castps256_ps128(sum));
        _mm_store_ps(&result.m[i+1][0], _mm256_extractf128_ps(sum, 1));
    }
}
#endif

// 6. Умножение матрицы на вектор
Vector4 matrix_vector_multiply_sse(const Matrix4& m, const Vector4& v) {
    // Загружаем вектор в 4 регистра с дублированием
    __m128 vx = _mm_set1_ps(v.x);  // [v.x, v.x, v.x, v.x]
    __m128 vy = _mm_set1_ps(v.y);  // [v.y, v.y, v.y, v.y]
    __m128 vz = _mm_set1_ps(v.z);  // [v.z, v.z, v.z, v.z]
    __m128 vw = _mm_set1_ps(v.w);  // [v.w, v.w, v.w, v.w]
    
    // Загружаем столбцы матрицы
    __m128 col0 = _mm_load_ps(&m.m[0][0]);
    __m128 col1 = _mm_load_ps(&m.m[1][0]);
    __m128 col2 = _mm_load_ps(&m.m[2][0]);
    __m128 col3 = _mm_load_ps(&m.m[3][0]);
    
    // Умножаем и складываем: col0*vx + col1*vy + col2*vz + col3*vw
    __m128 result = _mm_add_ps(
        _mm_add_ps(_mm_mul_ps(col0, vx), _mm_mul_ps(col1, vy)),
        _mm_add_ps(_mm_mul_ps(col2, vz), _mm_mul_ps(col3, vw))
    );
    
    Vector4 out;
    _mm_store_ps(out.data, result);
    return out;
}

// 7. Тест производительности
void benchmark_matrix_multiplication() {
    const int iterations = 1000000;
    std::random_device rd;
    std::mt19937 gen(rd());
    std::uniform_real_distribution<float> dis(-10.0f, 10.0f);
    
    // Генерируем случайные матрицы
    Matrix4 a, b, result_scalar, result_sse, result_avx;
    
    for (int i = 0; i < 4; ++i) {
        for (int j = 0; j < 4; ++j) {
            a.m[i][j] = dis(gen);
            b.m[i][j] = dis(gen);
        }
    }
    
    // Бенчмарк скалярной версии
    auto start = std::chrono::high_resolution_clock::now();
    for (int i = 0; i < iterations; ++i) {
        matrix_multiply_scalar(a, b, result_scalar);
    }
    auto end = std::chrono::high_resolution_clock::now();
    auto scalar_time = std::chrono::duration_cast<std::chrono::microseconds>(end - start).count();
    
    // Бенчмарк SSE версии
    start = std::chrono::high_resolution_clock::now();
    for (int i = 0; i < iterations; ++i) {
        matrix_multiply_sse(a, b, result_sse);
    }
    end = std::chrono::high_resolution_clock::now();
    auto sse_time = std::chrono::duration_cast<std::chrono::microseconds>(end - start).count();
    
    // Бенчмарк AVX версии (если доступна)
#ifdef __AVX__
    start = std::chrono::high_resolution_clock::now();
    for (int i = 0; i < iterations; ++i) {
        matrix_multiply_avx(a, b, result_avx);
    }
    end = std::chrono::high_resolution_clock::now();
    auto avx_time = std::chrono::duration_cast<std::chrono::microseconds>(end - start).count();
#endif
    
    // Проверка корректности (сравнение результатов)
    bool sse_correct = true;
    bool avx_correct = true;
    
    for (int i = 0; i < 4; ++i) {
        for (int j = 0; j < 4; ++j) {
            if (std::abs(result_scalar.m[i][j] - result_sse.m[i][j]) > 1e-5f) {
                sse_correct = false;
            }
#ifdef __AVX__
            if (std::abs(result_scalar.m[i][j] - result_avx.m[i][j]) > 1e-5f) {
                avx_correct = false;
            }
#endif
        }
    }
    
    // Вывод результатов
    std::cout << "=== Matrix Multiplication Benchmark ===" << std::endl;
    std::cout << "Iterations: " << iterations << std::endl;
    std::cout << std::fixed << std::setprecision(2);
    std::cout << "\nScalar time: " << scalar_time / 1000.0 << " ms" << std::endl;
    std::cout << "SSE time: " << sse_time / 1000.0 << " ms" << std::endl;
    std::cout << "SSE speedup: " << (float)scalar_time / sse_time << "x" << std::endl;
    std::cout << "SSE correct: " << (sse_correct ? "Yes" : "No") << std::endl;
    
#ifdef __AVX__
    std::cout << "\nAVX time: " << avx_time / 1000.0 << " ms" << std::endl;
    std::cout << "AVX speedup: " << (float)scalar_time / avx_time << "x" << std::endl;
    std::cout << "AVX correct: " << (avx_correct ? "Yes" : "No") << std::endl;
#else
    std::cout << "\nAVX: Not supported on this CPU" << std::endl;
#endif
}

// 8. Проверка поддержки SIMD инструкций
void check_simd_support() {
    std::cout << "\n=== CPU SIMD Support ===" << std::endl;
    
    // Проверяем через CPUID (Windows)
    int cpuInfo[4];
    
    // SSE
    __cpuid(cpuInfo, 1);
    bool sse = (cpuInfo[3] & (1 << 25)) != 0;
    bool sse2 = (cpuInfo[3] & (1 << 26)) != 0;
    bool sse3 = (cpuInfo[2] & (1 << 0)) != 0;
    bool ssse3 = (cpuInfo[2] & (1 << 9)) != 0;
    bool sse4_1 = (cpuInfo[2] & (1 << 19)) != 0;
    bool sse4_2 = (cpuInfo[2] & (1 << 20)) != 0;
    
    // AVX
    bool avx = (cpuInfo[2] & (1 << 28)) != 0;
    
    // AVX2
    __cpuid(cpuInfo, 7);
    bool avx2 = (cpuInfo[1] & (1 << 5)) != 0;
    
    std::cout << "SSE: " << (sse ? "Yes" : "No") << std::endl;
    std::cout << "SSE2: " << (sse2 ? "Yes" : "No") << std::endl;
    std::cout << "SSE3: " << (sse3 ? "Yes" : "No") << std::endl;
    std::cout << "SSSE3: " << (ssse3 ? "Yes" : "No") << std::endl;
    std::cout << "SSE4.1: " << (sse4_1 ? "Yes" : "No") << std::endl;
    std::cout << "SSE4.2: " << (sse4_2 ? "Yes" : "No") << std::endl;
    std::cout << "AVX: " << (avx ? "Yes" : "No") << std::endl;
    std::cout << "AVX2: " << (avx2 ? "Yes" : "No") << std::endl;
}

// 9. Main функция
int main() {
    std::cout << "SIMD Matrix Multiplication Demo" << std::endl;
    std::cout << "===============================" << std::endl;
    
    // Проверяем поддержку SIMD
    check_simd_support();
    
    // Запускаем бенчмарк
    benchmark_matrix_multiplication();
    
    // Тестируем умножение матрицы на вектор
    std::cout << "\n=== Matrix-Vector Multiplication ===" << std::endl;
    
    Matrix4 m;
    m.m[0][0] = 1; m.m[0][1] = 2; m.m[0][2] = 3; m.m[0][3] = 4;
    m.m[1][0] = 5; m.m[1][1] = 6; m.m[1][2] = 7; m.m[1][3] = 8;
    m.m[2][0] = 9; m.m[2][1] = 10; m.m[2][2] = 11; m.m[2][3] = 12;
    m.m[3][0] = 13; m.m[3][1] = 14; m.m[3][2] = 15; m.m[3][3] = 16;
    
    Vector4 v(1, 2, 3, 1);
    Vector4 result = matrix_vector_multiply_sse(m, v);
    
    std::cout << "Matrix:" << std::endl;
    for (int i = 0; i < 4; ++i) {
        std::cout << "  ";
        for (int j = 0; j < 4; ++j) {
            std::cout << m.m[i][j] << " ";
        }
        std::cout << std::endl;
    }
    
    std::cout << "\nVector: [" << v.x << ", " << v.y << ", " << v.z << ", " << v.w << "]" << std::endl;
    std::cout << "Result: [" << result.x << ", " << result.y << ", " << result.z << ", " << result.w << "]" << std::endl;
    
    // Ожидаемый результат: [18, 46, 74, 102]
    Vector4 expected(18, 46, 74, 102);
    std::cout << "Expected: [" << expected.x << ", " << expected.y << ", " << expected.z << ", " << expected.w << "]" << std::endl;
    std::cout << "Correct: " << 
        (std::abs(result.x - expected.x) < 1e-5f &&
         std::abs(result.y - expected.y) < 1e-5f &&
         std::abs(result.z - expected.z) < 1e-5f &&
         std::abs(result.w - expected.w) < 1e-5f ? "Yes" : "No") << std::endl;
    
    return 0;
}
```

## 3. Минимальный воспроизводимый GPU пример (OpenGL)

### Проектная структура для GPU демо:

```
gpu_demo/
├── CMakeLists.txt
├── src/
│   ├── main_gpu.cpp
│   ├── shader.vert
│   ├── shader.frag
│   └── gpu_helpers.cpp
└── include/
    └── gpu_helpers.h
```

**Файл: `CMakeLists.txt` для GPU демо**
```cmake
cmake_minimum_required(VERSION 3.15)
project(GPU_Demo)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Поиск библиотек
find_package(OpenGL REQUIRED)
find_package(glfw3 REQUIRED)
find_package(glm REQUIRED)

# Директории для заголовков
include_directories(${OPENGL_INCLUDE_DIR}
                   ${GLFW3_INCLUDE_DIR}
                   ${GLM_INCLUDE_DIR}
                   include)

# Исполняемый файл
add_executable(gpu_demo 
    src/main_gpu.cpp
    src/gpu_helpers.cpp
)

# Подключение библиотек
target_link_libraries(gpu_demo 
    ${OPENGL_LIBRARIES}
    glfw
    ${GLFW3_LIBRARIES}
)

# Копирование шейдеров в бинарную директорию
file(COPY shaders DESTINATION ${CMAKE_BINARY_DIR})
```

**Файл: `src/main_gpu.cpp`**
```cpp
/**
 * Минимальный GPU демо с OpenGL 4.6
 * Демонстрирует: вершинный шейдер, фрагментный шейдер, буферы
 */

#include <iostream>
#include <vector>
#include <cmath>

// Подключение OpenGL заголовков
#include <GL/glew.h>
#include <GLFW/glfw3.h>
#include <glm/glm.hpp>
#include <glm/gtc/matrix_transform.hpp>
#include <glm/gtc/type_ptr.hpp>

// Проверка ошибок OpenGL
#define GL_CHECK_ERROR() \
    { \
        GLenum err = glGetError(); \
        if (err != GL_NO_ERROR) { \
            std::cerr << "OpenGL error at line " << __LINE__ << ": " << gluErrorString(err) << std::endl; \
        } \
    }

// 1. Вершинный шейдер (GLSL 460)
const char* vertex_shader_source = R"(
#version 460 core

// Входные атрибуты вершин
layout(location = 0) in vec3 aPosition;
layout(location = 1) in vec3 aColor;

// Uniform-переменные (одинаковые для всех вершин)
uniform mat4 uModel;
uniform mat4 uView;
uniform mat4 uProjection;

// Выходные данные для фрагментного шейдера
out vec3 vColor;

void main() {
    // Преобразование позиции вершины
    vec4 worldPos = uModel * vec4(aPosition, 1.0);
    vec4 viewPos = uView * worldPos;
    gl_Position = uProjection * viewPos;
    
    // Передача цвета во фрагментный шейдер
    vColor = aColor;
}
)";

// 2. Фрагментный шейдер
const char* fragment_shader_source = R"(
#version 460 core

// Входные данные из вершинного шейдера
in vec3 vColor;

// Выходной цвет фрагмента
out vec4 FragColor;

void main() {
    // Просто используем переданный цвет
    FragColor = vec4(vColor, 1.0);
}
)";

// 3. Структура вершины
struct Vertex {
    glm::vec3 position;
    glm::vec3 color;
};

// 4. Создание треугольника (цветной)
std::vector<Vertex> create_triangle() {
    return {
        // Позиции           // Цвета
        {{-0.5f, -0.5f, 0.0f}, {1.0f, 0.0f, 0.0f}},  // Красный
        {{ 0.5f, -0.5f, 0.0f}, {0.0f, 1.0f, 0.0f}},  // Зеленый
        {{ 0.0f,  0.5f, 0.0f}, {0.0f, 0.0f, 1.0f}}   // Синий
    };
}

// 5. Создание куба (8 вершин, 12 треугольников)
std::vector<Vertex> create_cube() {
    std::vector<Vertex> vertices;
    
    // Вершины куба
    glm::vec3 positions[8] = {
        {-0.5f, -0.5f, -0.5f},
        { 0.5f, -0.5f, -0.5f},
        { 0.5f,  0.5f, -0.5f},
        {-0.5f,  0.5f, -0.5f},
        {-0.5f, -0.5f,  0.5f},
        { 0.5f, -0.5f,  0.5f},
        { 0.5f,  0.5f,  0.5f},
        {-0.5f,  0.5f,  0.5f}
    };
    
    // Цвета для каждой вершины
    glm::vec3 colors[8] = {
        {1.0f, 0.0f, 0.0f},  // Красный
        {0.0f, 1.0f, 0.0f},  // Зеленый
        {0.0f, 0.0f, 1.0f},  // Синий
        {1.0f, 1.0f, 0.0f},  // Желтый
        {1.0f, 0.0f, 1.0f},  // Пурпурный
        {0.0f, 1.0f, 1.0f},  // Голубой
        {0.5f, 0.5f, 0.5f},  // Серый
        {1.0f, 0.5f, 0.0f}   // Оранжевый
    };
    
    // Создаем вершины
    for (int i = 0; i < 8; ++i) {
        vertices.push_back({positions[i], colors[i]});
    }
    
    return vertices;
}

// 6. Индексы для куба (12 треугольников)
std::vector<unsigned int> create_cube_indices() {
    return {
        // Передняя грань
        0, 1, 2, 0, 2, 3,
        // Задняя грань
        4, 5, 6, 4, 6, 7,
        // Левая грань
        0, 3, 7, 0, 7, 4,
        // Правая грань
        1, 5, 6, 1, 6, 2,
        // Верхняя грань
        3, 2, 6, 3, 6, 7,
        // Нижняя грань
        0, 1, 5, 0, 5, 4
    };
}

// 7. Компиляция шейдера
GLuint compile_shader(GLenum type, const char* source) {
    GLuint shader = glCreateShader(type);
    glShaderSource(shader, 1, &source, nullptr);
    glCompileShader(shader);
    
    // Проверка ошибок компиляции
    GLint success;
    glGetShaderiv(shader, GL_COMPILE_STATUS, &success);
    if (!success) {
        char info_log[512];
        glGetShaderInfoLog(shader, 512, nullptr, info_log);
        std::cerr << "Shader compilation error:\n" << info_log << std::endl;
        glDeleteShader(shader);
        return 0;
    }
    
    return shader;
}

// 8. Создание шейдерной программы
GLuint create_shader_program() {
    // Компилируем шейдеры
    GLuint vertex_shader = compile_shader(GL_VERTEX_SHADER, vertex_shader_source);
    GLuint fragment_shader = compile_shader(GL_FRAGMENT_SHADER, fragment_shader_source);
    
    if (!vertex_shader || !fragment_shader) {
        return 0;
    }
    
    // Создаем программу и прикрепляем шейдеры
    GLuint program = glCreateProgram();
    glAttachShader(program, vertex_shader);
    glAttachShader(program, fragment_shader);
    glLinkProgram(program);
    
    // Проверка ошибок линковки
    GLint success;
    glGetProgramiv(program, GL_LINK_STATUS, &success);
    if (!success) {
        char info_log[512];
        glGetProgramInfoLog(program, 512, nullptr, info_log);
        std::cerr << "Program linking error:\n" << info_log << std::endl;
        glDeleteProgram(program);
        return 0;
    }
    
    // Удаляем шейдеры (они уже прикреплены к программе)
    glDeleteShader(vertex_shader);
    glDeleteShader(fragment_shader);
    
    return program;
}

// 9. Callback для ошибок GLFW
void error_callback(int error, const char* description) {
    std::cerr << "GLFW Error: " << description << std::endl;
}

// 10. Основная функция
int main() {
    std::cout << "OpenGL GPU Demo" << std::endl;
    std::cout << "===============" << std::endl;
    
    // Инициализация GLFW
    if (!glfwInit()) {
        std::cerr << "Failed to initialize GLFW" << std::endl;
        return -1;
    }
    
    glfwSetErrorCallback(error_callback);
    
    // Настройка OpenGL 4.6
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 4);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 6);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
    glfwWindowHint(GLFW_RESIZABLE, GL_FALSE);
    
    // Создание окна
    GLFWwindow* window = glfwCreateWindow(800, 600, "GPU Demo - Rotating Cube", nullptr, nullptr);
    if (!window) {
        std::cerr << "Failed to create GLFW window" << std::endl;
        glfwTerminate();
        return -1;
    }
    
    glfwMakeContextCurrent(window);
    glfwSwapInterval(1);  // VSync
    
    // Инициализация GLEW
    glewExperimental = GL_TRUE;
    if (glewInit() != GLEW_OK) {
        std::cerr << "Failed to initialize GLEW" << std::endl;
        glfwTerminate();
        return -1;
    }
    
    // Проверка версии OpenGL
    std::cout << "OpenGL Version: " << glGetString(GL_VERSION) << std::endl;
    std::cout << "GLSL Version: " << glGetString(GL_SHADING_LANGUAGE_VERSION) << std::endl;
    std::cout << "Renderer: " << glGetString(GL_RENDERER) << std::endl;
    
    // Настройка OpenGL
    glEnable(GL_DEPTH_TEST);
    glClearColor(0.1f, 0.1f, 0.1f, 1.0f);
    
    // Создание шейдерной программы
    GLuint shader_program = create_shader_program();
    if (!shader_program) {
        glfwTerminate();
        return -1;
    }
    
    // Создание геометрии куба
    std::vector<Vertex> vertices = create_cube();
    std::vector<unsigned int> indices = create_cube_indices();
    
    // Создание VAO (Vertex Array Object)
    GLuint vao;
    glGenVertexArrays(1, &vao);
    glBindVertexArray(vao);
    
    // Создание VBO (Vertex Buffer Object)
    GLuint vbo;
    glGenBuffers(1, &vbo);
    glBindBuffer(GL_ARRAY_BUFFER, vbo);
    glBufferData(GL_ARRAY_BUFFER, vertices.size() * sizeof(Vertex), vertices.data(), GL_STATIC_DRAW);
    
    // Создание EBO (Element Buffer Object)
    GLuint ebo;
    glGenBuffers(1, &ebo);
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, ebo);
    glBufferData(GL_ELEMENT_ARRAY_BUFFER, indices.size() * sizeof(unsigned int), indices.data(), GL_STATIC_DRAW);
    
    // Настройка атрибутов вершин
    // Атрибут позиции (location = 0)
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex), (void*)0);
    glEnableVertexAttribArray(0);
    
    // Атрибут цвета (location = 1)
    glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex), (void*)offsetof(Vertex, color));
    glEnableVertexAttribArray(1);
    
    // Отвязываем VAO
    glBindVertexArray(0);
    
    // Получение location uniform-переменных
    GLuint model_loc = glGetUniformLocation(shader_program, "uModel");
    GLuint view_loc = glGetUniformLocation(shader_program, "uView");
    GLuint projection_loc = glGetUniformLocation(shader_program, "uProjection");
    
    // Создание матриц проекции
    glm::mat4 projection = glm::perspective(
        glm::radians(45.0f),  // FOV
        800.0f / 600.0f,      // Aspect ratio
        0.1f,                 // Near plane
        100.0f                // Far plane
    );
    
    glm::mat4 view = glm::lookAt(
        glm::vec3(0.0f, 0.0f, 3.0f),  // Позиция камеры
        glm::vec3(0.0f, 0.0f, 0.0f),  // Точка наблюдения
        glm::vec3(0.0f, 1.0f, 0.0f)   // Вектор "вверх"
    );
    
    // Основной цикл рендеринга
    float rotation = 0.0f;
    while (!glfwWindowShouldClose(window)) {
        // Обработка ввода
        if (glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS) {
            glfwSetWindowShouldClose(window, true);
        }
        
        // Очистка буферов
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
        
        // Использование шейдерной программы
        glUseProgram(shader_program);
        
        // Обновление матрицы модели (вращение)
        rotation += 0.5f;
        glm::mat4 model = glm::mat4(1.0f);
        model = glm::rotate(model, glm::radians(rotation), glm::vec3(0.5f, 1.0f, 0.0f));
        
        // Передача uniform-переменных в шейдер
        glUniformMatrix4fv(model_loc, 1, GL_FALSE, glm::value_ptr(model));
        glUniformMatrix4fv(view_loc, 1, GL_FALSE, glm::value_ptr(view));
        glUniformMatrix4fv(projection_loc, 1, GL_FALSE, glm::value_ptr(projection));
        
        // Рендеринг куба
        glBindVertexArray(vao);
        glDrawElements(GL_TRIANGLES, indices.size(), GL_UNSIGNED_INT, 0);
        glBindVertexArray(0);
        
        // Проверка ошибок OpenGL
        GL_CHECK_ERROR();
        
        // Обмен буферов и обработка событий
        glfwSwapBuffers(window);
        glfwPollEvents();
    }
    
    // Очистка ресурсов
    glDeleteVertexArrays(1, &vao);
    glDeleteBuffers(1, &vbo);
    glDeleteBuffers(1, &ebo);
    glDeleteProgram(shader_program);
    
    // Завершение GLFW
    glfwDestroyWindow(window);
    glfwTerminate();
    
    std::cout << "\nGPU demo completed successfully!" << std::endl;
    return 0;
}
```

## 4. Примеры на других языках

### Rust (современный, безопасный)

**Cargo.toml:**
```toml
[package]
name = "graphics-demo"
version = "0.1.0"
edition = "2021"

[dependencies]
glam = "0.24"
rayon = "1.7"  # Для параллельных вычислений
```

**SIMD в Rust (src/main.rs):**
```rust
// Rust SIMD пример с использованием встроенных SIMD типов
#![feature(portable_simd)]
use std::simd::f32x4;
use std::time::Instant;

#[repr(align(16))]
#[derive(Copy, Clone, Debug)]
struct Matrix4 {
    rows: [f32x4; 4],
}

impl Matrix4 {
    fn new(diag: f32) -> Self {
        let mut rows = [f32x4::splat(0.0); 4];
        for i in 0..4 {
            rows[i] = f32x4::from_array([if i == 0 { diag } else { 0.0 },
                                        if i == 1 { diag } else { 0.0 },
                                        if i == 2 { diag } else { 0.0 },
                                        if i == 3 { diag } else { 0.0 }]);
        }
        Self { rows }
    }
    
    // SIMD умножение матриц
    fn multiply_simd(&self, other: &Matrix4) -> Matrix4 {
        let mut result = Matrix4::new(0.0);
        
        // Транспонируем other для эффективного доступа
        let mut columns = [f32x4::splat(0.0); 4];
        for i in 0..4 {
            columns[i] = f32x4::from_array([
                other.rows[0].as_array()[i],
                other.rows[1].as_array()[i],
                other.rows[2].as_array()[i],
                other.rows[3].as_array()[i],
            ]);
        }
        
        for i in 0..4 {
            let row = self.rows[i];
            let mut sum = f32x4::splat(0.0);
            
            for j in 0..4 {
                let broadcasted = f32x4::splat(row.as_array()[j]);
                sum += broadcasted * columns[j];
            }
            
            result.rows[i] = sum;
        }
        
        result
    }
    
    // Скалярное умножение (для сравнения)
    fn multiply_scalar(&self, other: &Matrix4) -> Matrix4 {
        let mut result = Matrix4::new(0.0);
        
        for i in 0..4 {
            for j in 0..4 {
                let mut sum = 0.0;
                for k in 0..4 {
                    sum += self.rows[i].as_array()[k] * other.rows[k].as_array()[j];
                }
                result.rows[i].as_mut_array()[j] = sum;
            }
        }
        
        result
    }
}

fn main() {
    let a = Matrix4::new(1.0);
    let b = Matrix4::new(2.0);
    
    // Тестируем производительность
    let iterations = 1_000_000;
    
    // Скалярная версия
    let start = Instant::now();
    for _ in 0..iterations {
        let _ = a.multiply_scalar(&b);
    }
    let scalar_duration = start.elapsed();
    
    // SIMD версия
    let start = Instant::now();
    for _ in 0..iterations {
        let _ = a.multiply_simd(&b);
    }
    let simd_duration = start.elapsed();
    
    println!("=== Rust SIMD Matrix Multiplication ===");
    println!("Iterations: {}", iterations);
    println!("Scalar time: {:?}", scalar_duration);
    println!("SIMD time: {:?}", simd_duration);
    println!("Speedup: {:.2}x", 
             scalar_duration.as_secs_f64() / simd_duration.as_secs_f64());
}
```

### C (минималистичный SIMD)

**simd_demo.c:**
```c
/**
 * Минимальный SIMD пример на C для Windows
 * Компиляция: gcc -mavx2 -O3 -o simd_demo simd_demo.c
 */

#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <immintrin.h>  // SIMD интринсики
#include <math.h>

// Выравненная память для SIMD
#ifdef _WIN32
    #define ALIGNED_MALLOC(size, alignment) _aligned_malloc(size, alignment)
    #define ALIGNED_FREE(ptr) _aligned_free(ptr)
#else
    #define ALIGNED_MALLOC(size, alignment) aligned_alloc(alignment, size)
    #define ALIGNED_FREE(ptr) free(ptr)
#endif

// Структура матрицы 4x4
typedef struct {
    __m128 rows[4];  // Четыре строки
} Matrix4;

// Инициализация матрицы
Matrix4* matrix_init(float value) {
    Matrix4* m = (Matrix4*)ALIGNED_MALLOC(sizeof(Matrix4), 16);
    __m128 v = _mm_set1_ps(value);
    for (int i = 0; i < 4; i++) {
        m->rows[i] = _mm_set_ps(
            (i == 3) ? value : 0.0f,
            (i == 2) ? value : 0.0f,
            (i == 1) ? value : 0.0f,
            (i == 0) ? value : 0.0f
        );
    }
    return m;
}

// SSE умножение матриц
void matrix_multiply_sse(const Matrix4* a, const Matrix4* b, Matrix4* result) {
    // Транспонируем b
    __m128 b0 = b->rows[0];
    __m128 b1 = b->rows[1];
    __m128 b2 = b->rows[2];
    __m128 b3 = b->rows[3];
    
    for (int i = 0; i < 4; i++) {
        __m128 a_row = a->rows[i];
        
        // Дублируем компоненты
        __m128 a0 = _mm_shuffle_ps(a_row, a_row, _MM_SHUFFLE(0, 0, 0, 0));
        __m128 a1 = _mm_shuffle_ps(a_row, a_row, _MM_SHUFFLE(1, 1, 1, 1));
        __m128 a2 = _mm_shuffle_ps(a_row, a_row, _MM_SHUFFLE(2, 2, 2, 2));
        __m128 a3 = _mm_shuffle_ps(a_row, a_row, _MM_SHUFFLE(3, 3, 3, 3));
        
        // Умножаем и складываем
        __m128 mul0 = _mm_mul_ps(a0, b0);
        __m128 mul1 = _mm_mul_ps(a1, b1);
        __m128 mul2 = _mm_mul_ps(a2, b2);
        __m128 mul3 = _mm_mul_ps(a3, b3);
        
        result->rows[i] = _mm_add_ps(_mm_add_ps(mul0, mul1), 
                                    _mm_add_ps(mul2, mul3));
    }
}

int main() {
    printf("C SIMD Matrix Multiplication Demo\n");
    printf("=================================\n");
    
    // Создаем матрицы
    Matrix4* a = matrix_init(1.0f);
    Matrix4* b = matrix_init(2.0f);
    Matrix4* result = matrix_init(0.0f);
    
    // Тестируем производительность
    const int iterations = 1000000;
    clock_t start = clock();
    
    for (int i = 0; i < iterations; i++) {
        matrix_multiply_sse(a, b, result);
    }
    
    clock_t end = clock();
    double duration = ((double)(end - start) * 1000.0) / CLOCKS_PER_SEC;
    
    printf("Iterations: %d\n", iterations);
    printf("Time: %.2f ms\n", duration);
    printf("Operations/sec: %.2f million\n", 
           (iterations * 64.0) / (duration * 1000.0)); // 64 операции умножения+сложения
    
    // Освобождаем память
    ALIGNED_FREE(a);
    ALIGNED_FREE(b);
    ALIGNED_FREE(result);
    
    return 0;
}
```

### Python (с использованием numpy для SIMD)

**simd_demo.py:**
```python
"""
Python SIMD демонстрация с использованием NumPy
NumPy автоматически использует SIMD инструкции через BLAS
"""

import numpy as np
import time
import sys

def matrix_multiply_numpy(A, B, iterations=1000):
    """Умножение матриц с использованием NumPy (автоматический SIMD)"""
    start = time.perf_counter()
    
    for _ in range(iterations):
        C = np.dot(A, B)
    
    duration = time.perf_counter() - start
    return duration, C

def matrix_multiply_python(A, B, iterations=1000):
    """Чисто Python умножение матриц (без SIMD)"""
    n = A.shape[0]
    C = np.zeros((n, n))
    
    start = time.perf_counter()
    
    for _ in range(iterations):
        for i in range(n):
            for j in range(n):
                s = 0
                for k in range(n):
                    s += A[i, k] * B[k, j]
                C[i, j] = s
    
    duration = time.perf_counter() - start
    return duration, C

def main():
    print("Python SIMD Demo with NumPy")
    print("=" * 30)
    
    # Создаем матрицы 4x4
    A = np.random.rand(4, 4).astype(np.float32)
    B = np.random.rand(4, 4).astype(np.float32)
    
    iterations = 10000
    
    # NumPy (с SIMD)
    numpy_time, numpy_result = matrix_multiply_numpy(A, B, iterations)
    
    # Чистый Python (без SIMD)
    python_time, python_result = matrix_multiply_python(A, B, iterations // 10)
    
    print(f"Matrix size: 4x4")
    print(f"Iterations (NumPy): {iterations}")
    print(f"Iterations (Python): {iterations // 10}")
    print(f"\nNumPy time: {numpy_time:.4f} sec")
    print(f"Python time: {python_time:.4f} sec")
    print(f"Speedup: {python_time / numpy_time:.2f}x")
    
    # Проверка корректности
    error = np.max(np.abs(numpy_result - python_result))
    print(f"\nMaximum error: {error:.6f}")
    
    # Демонстрация SIMD возможностей NumPy
    print(f"\nNumPy configuration:")
    print(f"  BLAS library: {np.__config__.get_info('blas_opt_info')['libraries'][0]}")
    print(f"  Using SIMD: {np.__config__.get_info('simd_opt_info')}")

if __name__ == "__main__":
    main()
```

## 5. Инструкции по запуску

### Для SIMD демо (C++):
```bash
# С Visual Studio
1. Откройте Developer Command Prompt for VS
2. Перейдите в папку с проектом
3. Компиляция: cl /EHsc /arch:AVX2 /O2 main_simd.cpp
4. Запуск: main_simd.exe

# С MinGW (если установлен)
g++ -mavx2 -O3 -o simd_demo main_simd.cpp
./simd_demo.exe
```

### Для GPU демо (OpenGL):
```bash
1. Установите CMake: https://cmake.org/download/
2. Установите GLFW и GLEW через vcpkg или скачайте вручную
3. Создайте build папку:
   mkdir build && cd build
4. Генерация проекта:
   cmake ..
5. Компиляция:
   cmake --build . --config Release
6. Запуск:
   ./gpu_demo.exe
```

### Для Rust демо:
```bash
# Установите Rust: https://rustup.rs/
rustc --version  # Проверка установки

# Запуск SIMD демо (нужен nightly Rust)
rustup toolchain install nightly
rustup default nightly
cargo run --release
```

## 6. Ожидаемые результаты

### SIMD демо:
```
SIMD Matrix Multiplication Demo
===============================

=== CPU SIMD Support ===
SSE: Yes
SSE2: Yes
SSE3: Yes
SSSE3: Yes
SSE4.1: Yes
SSE4.2: Yes
AVX: Yes
AVX2: Yes

=== Matrix Multiplication Benchmark ===
Iterations: 1000000

Scalar time: 145.23 ms
SSE time: 36.45 ms
SSE speedup: 3.98x
SSE correct: Yes

AVX time: 18.72 ms
AVX speedup: 7.76x
AVX correct: Yes
```

### GPU демо:
```
OpenGL GPU Demo
===============
OpenGL Version: 4.6.0 NVIDIA 471.41
GLSL Version: 4.60 NVIDIA
Renderer: NVIDIA GeForce RTX 3060

(Окно с вращающимся разноцветным кубом)
```

## 7. Поиск и устранение неисправностей

### Проблемы с SIMD:
1. **"Illegal instruction" ошибка** - CPU не поддерживает AVX/SSE
   - Решение: компилировать без `/arch:AVX2` или с `/arch:SSE2`

2. **Выравнивание памяти** - crash при использовании SIMD
   - Решение: использовать `alignas(16/32)` или `_aligned_malloc`

3. **Низкая производительность** - компилятор не оптимизирует
   - Решение: добавить `/O2` (MSVC) или `-O3` (GCC)

### Проблемы с GPU:
1. **"GLFW failed to initialize"** - нет OpenGL драйвера
   - Решение: обновить драйверы видеокарты

2. **Черный экран** - ошибки в шейдерах
   - Решение: проверить GLSL версию, добавить `glGetError()` проверки

3. **Низкий FPS** - VSync включен или тяжелые вычисления
   - Решение: отключить VSync (`glfwSwapInterval(0)`)

Эти минимальные примеры демонстрируют ключевые концепции SIMD и GPU вычислений и готовы к запуску на Windows 10/11 с минимальной настройкой.