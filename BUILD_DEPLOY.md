# 🚀 Инструкция по сборке и развертыванию ESP32-S3 FPV Drone Camera

## 🎯 Пошаговая инструкция для начинающих

### ⚡ Быстрый старт (5 минут)

1. **Скачайте и установите PlatformIO**

   ```bash
   # Установка через pip
   pip install platformio

   # Или установите VS Code + расширение PlatformIO IDE
   ```

2. **Клонируйте проект**

   ```bash
   git clone <repository-url>
   cd drone
   ```

3. **Подключите ESP32-S3 CAM к компьютеру через USB-C**

4. **Соберите и загрузите**

   ```bash
   # Сборка проекта
   pio run

   # Загрузка в ESP32-S3
   pio run --target upload

   # Мониторинг работы
   pio run --target monitor
   ```

5. **Подключитесь к дрону**
   - WiFi: `ESP32-S3_Drone_30fps`
   - Пароль: `drone2024`
   - Браузер: `http://192.168.4.1:8080`

## 🔧 Детальная инструкция по сборке

### Предварительные требования

**Аппаратные требования:**

- ESP32-S3 N16R8 CAM (16MB Flash, 8MB PSRAM)
- Камера OV2640 или OV5640
- USB-C кабель для программирования
- Стабильное питание 5V/2A

**Программные требования:**

- PlatformIO Core 6.0+
- Python 3.7+
- Git

### Установка среды разработки

#### Вариант 1: PlatformIO + VS Code (рекомендуется)

```bash
# 1. Установите VS Code
# 2. Установите расширение "PlatformIO IDE"
# 3. Перезапустите VS Code
```

#### Вариант 2: PlatformIO CLI

```bash
# Установка через pip
pip install platformio

# Проверка установки
pio --version
```

### Настройка проекта

1. **Клонирование репозитория**

   ```bash
   git clone <repository-url>
   cd drone
   ```

2. **Проверка конфигурации**

   ```bash
   # Просмотр доступных плат
   pio boards esp32

   # Проверка конфигурации проекта
   cat platformio.ini
   ```

3. **Установка зависимостей**
   ```bash
   # Автоматическая установка библиотек
   pio lib install
   ```

### Сборка проекта

#### Стандартная сборка

```bash
# Сборка для текущей среды
pio run

# Сборка с очисткой
pio run --target clean
pio run
```

#### Сборка с отладочной информацией

```bash
# Включение подробных логов
pio run --environment esp32s3_fpv_drone --target upload --target monitor
```

#### Проверка размера прошивки

```bash
# Информация о размере
pio run --target size

# Детальный анализ памяти
pio run --target size --target verbose
```

### Загрузка в ESP32-S3

#### Автоматическое определение порта

```bash
# PlatformIO автоматически найдет порт
pio run --target upload
```

#### Ручное указание порта

```bash
# Linux/macOS
pio run --target upload --upload-port /dev/ttyUSB0

# Windows
pio run --target upload --upload-port COM3
```

#### Режим загрузки

```bash
# Если ESP32-S3 не входит в режим загрузки автоматически:
# 1. Удерживайте кнопку BOOT
# 2. Нажмите кнопку RESET
# 3. Отпустите RESET, затем BOOT
# 4. Запустите команду загрузки
```

### Мониторинг и отладка

#### Serial монитор

```bash
# Базовый мониторинг
pio run --target monitor

# С автоматическим перезапуском при загрузке
pio run --target upload --target monitor

# С фильтрацией исключений
pio run --target monitor --monitor-filters esp32_exception_decoder
```

#### Расширенная отладка

```bash
# Увеличение уровня отладки
pio run --environment esp32s3_fpv_drone_debug

# Логирование в файл
pio run --target monitor > debug.log 2>&1
```

### Конфигурация камеры

#### Проверка пинов камеры

```cpp
// В include/ov2640.h проверьте соответствие пинов вашей плате:
namespace CameraPins {
    constexpr int XCLK = 15;    // Тактовый сигнал
    constexpr int SIOD = 4;     // I2C Data
    constexpr int SIOC = 5;     // I2C Clock
    // ... остальные пины
}
```

#### Настройка производительности

```cpp
// В include/ov2640.h для оптимизации:
constexpr uint8_t TARGET_FPS = 30;           // Целевой FPS
constexpr uint8_t JPEG_QUALITY = 8;          // Качество (0-63, меньше = лучше)
constexpr framesize_t FRAME_SIZE = FRAMESIZE_QVGA; // Разрешение
```

## 🔍 Диагностика проблем

### Проблемы со сборкой

#### Ошибка: "Platform not found"

```bash
# Обновление платформ
pio platform update

# Установка ESP32 платформы
pio platform install espressif32
```

#### Ошибка: "Library not found"

```bash
# Обновление библиотек
pio lib update

# Принудительная переустановка
pio lib uninstall --all
pio lib install
```

#### Проблемы с памятью при сборке

```bash
# Увеличение размера стека
export PLATFORMIO_BUILD_FLAGS="-Wl,-Tesp32s3.rom.ld"

# Очистка кэша
pio system prune
```

### Проблемы с загрузкой

#### ESP32-S3 не определяется

```bash
# Проверка подключенных устройств
pio device list

# Принудительная установка драйвера (Windows)
# Скачайте CP2102 драйвер с сайта Silicon Labs
```

#### Ошибка загрузки

```bash
# Сброс в режим загрузки
# 1. Удерживайте BOOT
# 2. Кратко нажмите RESET
# 3. Отпустите BOOT через 2 секунды

# Попробуйте другую скорость загрузки
pio run --target upload --upload-speed 115200
```

### Проблемы с камерой

#### Камера не инициализируется

```cpp
// Проверьте последовательность пинов в CameraPins
// Убедитесь что питание стабильное (5V/2A минимум)
// Проверьте подключение шлейфа камеры
```

#### Низкий FPS или артефакты

```cpp
// Уменьшите разрешение
config_.frame_size = FRAMESIZE_QQVGA;

// Увеличьте JPEG качество (ухудшит сжатие)
constexpr uint8_t JPEG_QUALITY = 12;

// Проверьте питание и помехи
```

## 📊 Тестирование системы

### Автоматические тесты

```bash
# Запуск базовых тестов
pio test

# Тесты на устройстве
pio test --environment esp32s3_fpv_drone
```

### Ручное тестирование

#### Проверка основных функций

```bash
# 1. Подключитесь к Serial монитору (115200 baud)
# 2. Введите команды:
status    # Проверка всех модулей
memory    # Проверка памяти
fps       # Проверка камеры
wifi      # Проверка WiFi
clients   # Проверка подключений
```

#### Тест производительности

```bash
# Подключите несколько клиентов одновременно
# Проверьте стабильность FPS
# Мониторьте использование памяти
```

## 🚀 Развертывание

### Подготовка к производству

1. **Финальная сборка**

   ```bash
   # Сборка с оптимизацией
   pio run --environment esp32s3_fpv_drone_release
   ```

2. **Настройка для конечного пользователя**

   ```cpp
   // Отключите отладочные сообщения
   #define CORE_DEBUG_LEVEL 0

   // Установите финальные настройки WiFi
   wifi.init("YOUR_DRONE_NAME", "your_password");
   ```

3. **Создание образа прошивки**
   ```bash
   # Экспорт скомпилированной прошивки
   pio run --target buildfs
   pio run --target uploadfs
   ```

### OTA обновления (будущая функция)

```cpp
// Заготовка для OTA обновлений
// В будущих версиях будет реализована возможность
// обновления прошивки через WiFi
```

## 📱 Клиентские приложения

### Веб-клиент

- Встроенный HTML клиент доступен по адресу `http://192.168.4.1:8080`
- Автоматическое переподключение при потере связи
- Отображение статистики и FPS

### Мобильные приложения

```javascript
// Пример WebSocket подключения для мобильного приложения
const ws = new WebSocket("ws://192.168.4.1:8080");
ws.binaryType = "arraybuffer";

ws.onmessage = function (event) {
  if (event.data instanceof ArrayBuffer) {
    // Обработка JPEG кадра
    const blob = new Blob([event.data], { type: "image/jpeg" });
    const url = URL.createObjectURL(blob);
    imageElement.src = url;
  }
};
```

## 🔧 Кастомизация

### Изменение сетевых настроек

```cpp
// В src/wifi/wifi_module.cpp
void WiFiModule::init(const char* ssid, const char* password) {
    // Измените на свои значения
    WiFi.softAP("YOUR_DRONE_SSID", "YOUR_PASSWORD");
}
```

### Добавление новых функций

```cpp
// 1. Создайте новый модуль в src/your_module/
// 2. Добавьте заголовок в include/
// 3. Интегрируйте в SystemManager
// 4. Добавьте Serial команды в CommandHandler
```

### Настройка качества видео

```cpp
// Различные профили качества
// Профиль "Высокое качество" (низкий FPS)
constexpr framesize_t FRAME_SIZE = FRAMESIZE_VGA;
constexpr uint8_t JPEG_QUALITY = 4;

// Профиль "Высокий FPS" (среднее качество)
constexpr framesize_t FRAME_SIZE = FRAMESIZE_QVGA;
constexpr uint8_t JPEG_QUALITY = 8;

// Профиль "Максимальный FPS" (низкое качество)
constexpr framesize_t FRAME_SIZE = FRAMESIZE_QQVGA;
constexpr uint8_t JPEG_QUALITY = 12;
```

## 📚 Полезные ресурсы

### Документация

- [ESP32-S3 Technical Reference](https://www.espressif.com/sites/default/files/documentation/esp32-s3_technical_reference_manual_en.pdf)
- [OV2640 Datasheet](https://www.uctronics.com/download/cam_module/OV2640DS.pdf)
- [PlatformIO Documentation](https://docs.platformio.org/)

### Сообщества

- [ESP32 Arduino Forum](https://www.arduino.cc/en/Forum/ESP32)
- [PlatformIO Community](https://community.platformio.org/)
- [ESP32.com Forum](https://esp32.com/)

### Примеры кода

- [ESP32 Camera Examples](https://github.com/espressif/esp32-camera)
- [WebSocket Server Examples](https://github.com/Links2004/arduinoWebSockets)

---

**🎯 Результат:** После выполнения всех шагов у вас будет работающая система FPV дрона с камерой, готовая к использованию и дальнейшему развитию!
