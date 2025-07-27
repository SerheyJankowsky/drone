# 🛠️ Руководство по настройке ESP32-S3 FPV Drone Camera

## 📋 Подготовка к работе

### Требования

- **Плата**: ESP32-S3 N16R8 CAM (16MB Flash, 8MB PSRAM)
- **Камера**: OV2640 или OV5640
- **ПО**: PlatformIO IDE или Arduino IDE с поддержкой ESP32-S3
- **Кабель**: USB-C для прошивки и питания

### Установка PlatformIO (рекомендуется)

```bash
# Через pip
pip install platformio

# Или через VS Code
# Установите расширение PlatformIO IDE
```

## ⚡ Быстрый запуск

### 1. Клонирование и сборка

```bash
git clone <repository-url>
cd drone
pio run
```

### 2. Прошивка платы

```bash
# Подключите ESP32-S3 через USB
pio run --target upload
```

### 3. Мониторинг работы

```bash
# Просмотр логов (115200 baud)
pio run --target monitor
```

### 4. Подключение к дрону

1. Найдите WiFi сеть: **ESP32-S3_Drone_30fps**
2. Пароль: **drone2024**
3. Откройте: **http://192.168.4.1:8080**

## 🔧 Конфигурация пинов камеры

### ESP32-S3 CAM стандартные пины:

```cpp
// В include/ov2640.h проверьте соответствие:
namespace CameraPins {
    constexpr int PWDN = -1;      // Не используется
    constexpr int RESET = -1;     // Не используется
    constexpr int XCLK = 15;      // Тактовый сигнал
    constexpr int SIOD = 4;       // I2C SDA
    constexpr int SIOC = 5;       // I2C SCL
    constexpr int Y9 = 16;        // Данные D7
    constexpr int Y8 = 17;        // Данные D6
    constexpr int Y7 = 18;        // Данные D5
    constexpr int Y6 = 12;        // Данные D4
    constexpr int Y5 = 10;        // Данные D3
    constexpr int Y4 = 8;         // Данные D2
    constexpr int Y3 = 9;         // Данные D1
    constexpr int Y2 = 11;        // Данные D0
    constexpr int VSYNC = 6;      // Вертикальная синхронизация
    constexpr int HREF = 7;       // Горизонтальная синхронизация
    constexpr int PCLK = 13;      // Пиксельный такт
}
```

### Если пины не совпадают с вашей платой:

1. Найдите схему вашей платы ESP32-S3 CAM
2. Отредактируйте пины в `include/ov2640.h`
3. Пересоберите проект: `pio run`

## 📊 Оптимизация производительности

### Настройка FPS и качества

```cpp
// В include/ov2640.h
namespace CameraConfig {
    constexpr uint8_t TARGET_FPS = 30;     // Целевой FPS
    constexpr uint8_t JPEG_QUALITY = 8;    // Качество JPEG (0-63, меньше = лучше)
    constexpr framesize_t FRAME_SIZE = FRAMESIZE_QVGA;  // Разрешение
}
```

### Доступные разрешения:

- `FRAMESIZE_QQVGA` - 160x120 (максимальный FPS)
- `FRAMESIZE_QVGA` - 320x240 (рекомендуется)
- `FRAMESIZE_VGA` - 640x480 (хорошее качество)
- `FRAMESIZE_SVGA` - 800x600 (высокое качество)
- `FRAMESIZE_UXGA` - 1600x1200 (максимальное качество)

### Настройка WiFi

```cpp
// В src/wifi/wifi_module.cpp, функция init()
wifi.init("ВАШ_SSID", "ВАШ_ПАРОЛЬ");
```

## 🔍 Serial команды для диагностики

После подключения к Serial монитору (115200 baud) доступны команды:

```
help          - Показать все команды
status        - Полный статус системы
camera        - Информация о камере
wifi          - Статус WiFi соединения
clients       - Подключенные WebSocket клиенты
memory        - Использование памяти
fps           - Текущий FPS камеры
restart       - Перезагрузка системы
```

### Пример вывода команды status:

```
==================================================
ESP32-S3 Drone System Status
==================================================
System: ✅ ONLINE - Uptime: 00:15:32
Camera: ✅ Active - 28.7 FPS (QVGA, Quality: 8)
WiFi:   ✅ AP Mode - SSID: ESP32-S3_Drone_30fps
        📶 Signal: Strong, 2 clients connected
Memory: 📊 Heap: 180KB free / 512KB total
        📊 PSRAM: 7.2MB free / 8MB total
WebSocket: 🌐 Port 8080, 2 active connections
Network: 📈 1.2MB/s transmitted, 847 frames sent
Uptime: ⏱️ 15m 32s
==================================================
```

## 🌐 Клиенты для подключения

### 1. Встроенный веб-клиент (рекомендуется)

- URL: `http://192.168.4.1:8080`
- Функции: стрим видео, статистика, скриншоты

### 2. Пользовательский HTML клиент

- Используйте файл `drone_client.html`
- Откройте в браузере
- Автоподключение к WebSocket

### 3. WebSocket API для разработчиков

```javascript
const ws = new WebSocket("ws://192.168.4.1:8080");

ws.onmessage = function (event) {
  if (event.data instanceof Blob) {
    // Получен JPEG кадр
    const url = URL.createObjectURL(event.data);
    imageElement.src = url;
  }
};
```

## 🚨 Решение проблем

### Проблема: Камера не инициализируется

```
Проверьте:
1. Правильность подключения камеры
2. Соответствие пинов в CameraPins
3. Качество питания (5V/2A минимум)
4. Serial лог на ошибки инициализации
```

### Проблема: Низкий FPS или зависания

```
Решения:
1. Увеличьте JPEG_QUALITY (ухудшит качество, повысит FPS)
2. Уменьшите разрешение (FRAMESIZE_QQVGA)
3. Проверьте количество подключенных клиентов (максимум 3)
4. Убедитесь в стабильности питания
5. Используйте Serial команду 'memory' для проверки RAM
```

### Проблема: WiFi не создается

```
Проверьте:
1. Регион WiFi (может требоваться настройка)
2. Перезагрузите ESP32
3. Используйте команду 'wifi' в Serial
4. Проверьте антенну (если внешняя)
```

### Проблема: WebSocket не подключается

```
Решения:
1. Убедитесь что подключены к правильной WiFi сети
2. IP должен быть 192.168.4.1
3. Порт 8080 должен быть открыт
4. Попробуйте другой браузер/устройство
5. Проверьте Serial лог на ошибки WebSocket
```

## 📱 Мобильные приложения

Для создания мобильного приложения используйте WebSocket подключение:

- **Android**: WebView с JavaScript
- **iOS**: WKWebView с JavaScript
- **Flutter**: web_socket_channel пакет
- **React Native**: WebSocket API

## 🔧 Расширение функциональности

### Добавление новой Serial команды

```cpp
// В src/commands/command_handler.cpp
void CommandHandler::handleCustomCommand() {
    Serial.println("[CMD] Моя команда выполнена");
    // Ваш код здесь
}

// Добавьте в список команд в процессе Commands
```

### Добавление нового модуля

```cpp
// 1. Создайте файлы: src/your_module/your_module.cpp и include/your_module.h
// 2. В include/system_manager.h добавьте:
class YourModule;
YourModule* yourModule;

// 3. В src/system/system_manager.cpp в методе initialize():
yourModule = new YourModule();
yourModule->init();
```

## ⚙️ Файл platformio.ini

Убедитесь что настройки соответствуют вашему проекту:

```ini
[env:esp32-s3-devkitc-1]
platform = espressif32
board = esp32-s3-devkitc-1
framework = arduino
monitor_speed = 115200
board_build.flash_mode = qio
board_build.psram_type = opi
```

## 🎯 Советы по оптимизации

1. **Память**: Используйте PSRAM для больших буферов
2. **FPS**: Баланс между качеством и производительностью
3. **Сеть**: Ограничивайте количество клиентов для стабильности
4. **Питание**: Обеспечьте стабильное питание 5V/2A+
5. **Отладка**: Используйте Serial команды для мониторинга

## 📞 Поддержка

При возникновении проблем:

1. Проверьте Serial лог (115200 baud)
2. Используйте команду `status` для диагностики
3. Убедитесь в правильности подключения камеры
4. Проверьте соответствие пинов вашей плате
