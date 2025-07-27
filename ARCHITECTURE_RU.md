# 🏗️ Архитектура ESP32-S3 FPV Drone Camera System

## 📋 Общий обзор

Проект построен на модульной архитектуре с четким разделением ответственности между компонентами. Каждый модуль инкапсулирует свою функциональность и взаимодействует с другими через определенные интерфейсы.

## 🎯 Ключевые принципы архитектуры

- ✅ **Модульность**: Каждый компонент - отдельный класс с ясной ответственностью
- ✅ **Расширяемость**: Легко добавлять новые функции и модули
- ✅ **Отказоустойчивость**: Система продолжает работать при сбоях отдельных компонентов
- ✅ **Производительность**: Оптимизация для двухъядерного ESP32-S3
- ✅ **Диагностика**: Подробное логирование и команды для отладки

## 🧩 Структура модулей

```
┌─────────────────────────────────────────────────────────────┐
│                    SystemManager                            │
│              (Центральный координатор)                       │
└─────────────┬─────────────┬─────────────┬─────────────┬─────┘
              │             │             │             │
    ┌─────────▼──────┐ ┌────▼─────┐ ┌─────▼──────┐ ┌───▼────────┐
    │  WiFiModule    │ │OV2640Camera│ │WebSocketSvr│ │TaskManager │
    │ (Точка доступа)│ │ (Драйвер)  │ │ (Стриминг) │ │(FreeRTOS)  │
    └────────────────┘ └───────────┘ └────────────┘ └────────────┘
              │             │             │             │
    ┌─────────▼──────┐ ┌────▼─────┐ ┌─────▼──────┐ ┌───▼────────┐
    │   ESP32 WiFi   │ │ OV2640   │ │Native WS   │ │  Core 0    │
    │   Hardware     │ │ Hardware │ │ Protocol   │ │  Core 1    │
    └────────────────┘ └──────────┘ └────────────┘ └────────────┘
```

## 📂 Детальная структура проекта

### `/src/main.cpp` - Точка входа

```cpp
// Минималистичная точка входа
void setup() {
    SystemManager::getInstance().init();  // Инициализация всех модулей
}

void loop() {
    SystemManager::getInstance().run();   // Основной цикл
}
```

### `/src/system/system_manager.cpp` - Центральный менеджер

**Ответственность:**

- Инициализация всех модулей в правильном порядке
- Координация работы между модулями
- Мониторинг состояния системы
- Обработка ошибок и восстановление

**Ключевые методы:**

```cpp
bool initialize()     // Инициализация всех компонентов
void update()         // Основной цикл обновления
void shutdown()       // Корректное завершение работы
void printSystemStatus() // Статус всех модулей
```

### `/src/wifi/wifi_module.cpp` - WiFi точка доступа

**Ответственность:**

- Создание WiFi точки доступа (AP режим)
- Мониторинг подключений клиентов
- Обработка событий WiFi
- Диагностика сетевых проблем

**Настройки по умолчанию:**

```cpp
SSID: "ESP32-S3_Drone_30fps"
Password: "drone2024"
IP: 192.168.4.1
Channel: Auto
Max Clients: 4
```

### `/src/camera/ov2640.cpp` - Драйвер камеры

**Ответственность:**

- Инициализация камеры OV2640/OV5640
- Захват кадров в JPEG формате
- Настройка качества и разрешения
- Статистика производительности

**Оптимизации:**

```cpp
Target FPS: 30
JPEG Quality: 8 (оптимальный баланс)
Frame Size: QVGA (320x240)
Buffer Count: 2 (двойная буферизация)
XCLK Frequency: 24MHz
```

### `/src/ws/websocket_server.cpp` - WebSocket сервер

**Ответственность:**

- Нативная реализация WebSocket протокола (RFC 6455)
- Обслуживание до 3 клиентов одновременно
- Бинарная передача JPEG кадров
- Встроенный HTML клиент для браузера

**Особенности:**

- Отсутствие библиотечной зависимости от WebSocket
- Оптимизированная передача без буферизации
- Автоматическое переподключение клиентов
- Ping/Pong для контроля соединения

### `/src/tasks/task_manager.cpp` - Менеджер задач FreeRTOS

**Ответственность:**

- Создание задач на разных ядрах процессора
- Распределение нагрузки между Core0 и Core1
- Мониторинг производительности задач
- Обработка приоритетов

**Архитектура задач:**

```
Core 0 (Protocol CPU):           Core 1 (Application CPU):
├── WebSocket Server Task        ├── Video Stream Task (HIGH)
├── WiFi Event Handling          ├── Camera Capture Loop
├── Serial Commands              ├── Frame Processing
└── System Monitoring            └── Memory Management
```

### `/src/commands/command_handler.cpp` - Serial команды

**Ответственность:**

- Обработка команд через Serial порт
- Диагностика и отладка системы
- Интерактивное управление
- Мониторинг состояния

## 🔄 Жизненный цикл системы

### 1. Инициализация (setup)

```
1. Serial.begin(115200)
2. SystemManager::getInstance().init()
   ├── Camera initialization
   ├── WiFi AP start
   ├── WebSocket server start
   └── Task creation (Core0 + Core1)
3. CommandHandler setup
4. "System ready" message
```

### 2. Основной цикл (loop)

```
1. SystemManager::update()
   ├── Task monitoring
   ├── Error checking
   └── Statistics update
2. CommandHandler::processCommands()
   ├── Serial input processing
   └── Command execution
3. Minimal delay (100μs)
```

### 3. Параллельные задачи (FreeRTOS)

```
VideoStreamTask (Core1, Priority 2):
├── Camera frame capture
├── JPEG compression
├── Frame statistics
└── Memory management

WebSocketTask (Core0, Priority 1):
├── Client connection handling
├── Frame transmission
├── Connection monitoring
└── Protocol management
```

## 📊 Потоки данных

### Видео поток:

```
OV2640 Camera → Frame Buffer → JPEG Encoder → WebSocket → Client Browser
     ↓             ↓              ↓              ↓              ↓
  Hardware     PSRAM Memory   Compression   Network TCP    Display
  20-30ms        <1ms          5-10ms       10-50ms       16ms
```

### Управляющие данные:

```
Serial Commands → CommandHandler → SystemManager → Module Actions
Browser ← HTML Page ← WebSocket Server ← HTTP Request
```

## ⚡ Оптимизации производительности

### Память:

- **PSRAM** для буферов кадров (8MB доступно)
- **Heap** для небольших объектов и стека
- **Двойная буферизация** для плавного захвата
- **Автоматическая очистка** неиспользуемых буферов

### Сеть:

- **Нативный WebSocket** без библиотек
- **Бинарная передача** JPEG без кодирования
- **Отсутствие дополнительной буферизации**
- **TCP_NODELAY** для низкой задержки

### Процессор:

- **Dual-core** использование (Core0 + Core1)
- **Приоритеты задач** для критичных операций
- **Минимальные задержки** в основном цикле
- **240MHz** тактовая частота

## 🔧 Настройка и конфигурация

### Основные параметры в `/include/ov2640.h`:

```cpp
constexpr uint8_t TARGET_FPS = 30;        // Целевой FPS
constexpr uint8_t JPEG_QUALITY = 8;       // Качество (0-63)
constexpr framesize_t FRAME_SIZE = FRAMESIZE_QVGA; // Разрешение
```

### WiFi настройки в `/src/wifi/wifi_module.cpp`:

```cpp
wifi.init("ESP32-S3_Drone_30fps", "drone2024");
```

### WebSocket настройки в `/include/websocket_server.h`:

```cpp
static const int MAX_CLIENTS = 3;         // Максимум клиентов
uint16_t port = 8080;                     // Порт сервера
```

## 🚨 Обработка ошибок

### Стратегии восстановления:

1. **Camera failure**: Автоперезапуск драйвера камеры
2. **WiFi disconnection**: Автовосстановление AP
3. **WebSocket errors**: Переподключение клиентов
4. **Memory exhaustion**: Автоочистка буферов
5. **Task watchdog**: Мягкий перезапуск задач

### Логирование:

- **Serial output** для отладки (115200 baud)
- **Уровни логов**: ERROR, WARN, INFO, DEBUG
- **Статистика производительности** в реальном времени
- **Memory monitoring** для предотвращения переполнения

## 📈 Мониторинг и диагностика

### Serial команды:

```bash
status     # Полный статус всех модулей
memory     # Использование памяти (Heap/PSRAM)
fps        # Текущая частота кадров
clients    # Подключенные WebSocket клиенты
wifi       # Статус WiFi соединения
restart    # Перезагрузка системы
help       # Справка по всем командам
```

### Автоматический мониторинг:

- FPS tracking каждую секунду
- Memory usage каждые 5 секунд
- Connection health каждые 30 секунд
- System uptime и статистика перезагрузок

## 🔮 Планы развития

### Возможные улучшения:

- **Поддержка H.264** для лучшего сжатия
- **Управление моторами** для дрона
- **Телеметрия** (GPS, IMU, Battery)
- **Запись на SD карту**
- **OTA обновления** через WiFi
- **Мобильное приложение**

### Добавление новых модулей:

1. Создать класс в `/src/new_module/`
2. Добавить заголовок в `/include/`
3. Интегрировать в `SystemManager`
4. Добавить команды в `CommandHandler`
